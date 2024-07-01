# argocd-applications

Argo CD dummy applications


## ApplicationSet's

This example demonstrates the git directory generator:

```yaml
kind: ApplicationSet
metadata:
  name: cluster-addons
spec:
  generators:
  - git:
      repoURL: https://github.com/jsolana/argocd-applications.git
      directories:
      - path: busibox/*
  template:
    metadata:
      name: '{{path.basename}}'
    spec:
      source:
        repoURL: https://github.com/jsolana/argocd-applications.git
        targetRevision: HEAD
        path: '{{path}}'
      destination:
        server: http://kubernetes.default.svc
        namespace: '{{path.basename}}'

```
