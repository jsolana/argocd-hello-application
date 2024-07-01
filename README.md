# argocd-applications

This repository contains examples with ArgoCD.
Requirements:

- Install Arco CD in a k8s cluster locally using `kind`

```bash
    brew install kind
    kind create cluster
    kubectl create namespace argocd
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.12.0-rc2/manifests/install.yaml

    kubectl port-forward svc/argocd-server -n argocd 8080:443
    kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath='{.data.password}' | base64 -d

```

## Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: busybox-echo
  namespace: argocd
spec:
  destination:
    namespace: busybox-echo
    server: https://kubernetes.default.svc
  project: default
  source:
    path: busybox/echo
    repoURL: https://github.com/jsolana/argocd-applications
    targetRevision: HEAD
  syncPolicy:
    automated:
      allowEmpty: true
      prune: true
    retry:
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 1m
      limit: 2
    syncOptions:
    - CreateNamespace=true
```

## ApplicationSet

A git generator's applicationset:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: echo-appset
  namespace: argocd
spec:
  generators:
  - git:
      repoURL: https://github.com/jsolana/argocd-applications.git
      revision: HEAD
      directories:
      - path: busybox/*
  template:
    metadata:
      name: '{{path.basename}}'
    spec:
      project: "default"
      source:
        repoURL: https://github.com/jsolana/argocd-applications.git
        targetRevision: HEAD
        path: '{{path}}'
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{path.basename}}'
      syncPolicy:
        automated:
          prune: true
          allowEmpty: true
        syncOptions:
        - ApplyOutOfSyncOnly=true
        - Validate=true
        - CreateNamespace=true
        - PrunePropagationPolicy=foreground
        - ServerSideApply=true
        - FailOnSharedResource=true
        retry:
          limit: 1
          backoff:
            duration: 2m
            factor: 2
            maxDuration: 3m
```
