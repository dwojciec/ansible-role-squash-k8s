Ansible Role: Squash (solo.io) Kubernetes
======================


Manages a Squash Debugger application in Kubernetes|OpenShift. This project also
includes the necessary bits for deploying this role with Ansible Operator in a
Kubernetes|OpenShift cluster.

This project is based on this blog post [Reaching for the Stars with Ansible Operator] {https://blog.openshift.com/reaching-for-the-stars-with-ansible-operator/} and my previous project [Squash Operator]{https://github.com/dwojciec/squash-operator}

Step (A)

```
$ git clone https://github.com/dwojciec/ansible-role-squash-k8s.git
$ cd ansible-role-squash-k8s/build
$ docker build -t squash-ansible-operator -f Dockerfile .
```

Then push the image build to your repository 
```
# docker login quay.io
Username: dwojciec
Password:
Login Succeeded
# docker tag squash-ansible-operator:latest quay.io/dwojciec/squash-ansible-operator:latest
# docker push quay.io/dwojciec/squash-ansible-operator
```

Then, we create the important objects our operator needs to run:

```
# oc new-project squash-server
# kubectl create -f deploy/sa.yaml \
                  -f deploy/role.yaml \
                  -f deploy/role_binding.yaml \
                  -f deploy/crds/crd.yaml
serviceaccount/squash-operator created
role.rbac.authorization.k8s.io/squash-operator created
rolebinding.rbac.authorization.k8s.io/squash-operator created
customresourcedefinition.apiextensions.k8s.io/squashes.app.dwojciec.com created

We need scc privileged to be able to hostPID for squash-client daemonset 
# oc adm policy add-scc-to-user privileged -n squash-server -z default

# oc adm policy add-role-to-user cluster-admin system:serviceaccount:squash-server:squash-operator
```

We need additionnal privileges to list route on the cluster. We are creating a clusterrole and we are assigning it to the Service Account
```
# oc create clusterrole listroute --verb=list --resource=route
# oc adm policy add-cluster-role-to-user listroute system:serviceaccount:squash-server:squash-operator
```

Then, we start the operator:


```
# Use the image name from the operator-sdk build step (A)  above
# sed 's|REPLACE_IMAGE|quay.io/dwojciec/squash-ansible-operator:latest|g' deploy/operator.yaml | kubectl create -f -
deployment.apps/squash-operator created
```

Finally, create a Squash resource:

```
# kubectl create -f deploy/crds/cr.yaml
squash.app.dwojciec.com/squashtest created
```


### Cleanup the squash-operator:
```
$ kubectl delete -f deploy/crds/cr.yaml \
                 -f deploy/operator.yaml \
                 -f deploy/role.yaml \ 
                 -f deploy/role_binding.yaml \
                 -f deploy/sa.yaml \
                 -f deploy/crds/crd.yaml
```
