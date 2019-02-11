FROM quay.io/operator-framework/ansible-operator

RUN ansible-galaxy install dwojciec.squash_k8s

RUN echo $'--- \n\
- version: v1alpha1\n\
  group: examples.dwojciec.com\n\
  kind: Squash\n\
  role: /opt/ansible/roles/dwojciec.squash_k8s' > ${HOME}/watches.yaml
