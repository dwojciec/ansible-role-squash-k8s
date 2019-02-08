Ansible Role: Squash (solo.io) Kubernetes
======================


Manages a Squash Debugger application in Kubernetes|OpenShift. This project also
includes the necessary bits for deploying this role with Ansible Operator in a
Kubernetes|OpenShift cluster.

```
$ operator-sdk build squash-operator
```

Then, we create the important objects our operator needs to run:

```
$ kubectl create -f deploy/sa.yaml \
                 -f deploy/rbac.yaml \
                 -f deploy/crds/crd.yaml \
```

Then, we start the operator:

```
# Use the image name from the operator-sdk build step above
# and set the imagePullPolicy to Never
$ sed 's|REPLACE_IMAGE|squasg-operator|g; s|Always|Never|' deploy/operator.yaml | kubectl create -f -
```

Finally, create a Squash resource:

```
$ kubectl create -f deploy/crds/cr.yaml
```
