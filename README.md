# stocktrader-argocd-gitops

Devops git for Stocktrader app.

Each branch in the repo matches one of the environments:
- test - test environment in the local argo cluster
- uat - uat environment in the remote cluster
- production - prod environmet in the remote cluster

Steps:

- install ArgoCD via operator, use the following yaml:

```
apiVersion: argoproj.io/v1alpha1
kind: ArgoCD
metadata:
  name: argocd
  namespace: argocd
spec: 
  dex:
    image: quay.io/ablock/dex
    openShiftOAuth: true
    version: openshift-connector
  rbac:
    defaultPolicy: 'role:admin'
    policy: |
      g, argocd-admins, role:admin
    scopes: '[groups]'    
  server:
    insecure: true
    route:
      enabled: true
      tls:
        termination: edge
      wildcardPolicy: None    
```

- solve configuration issues - see argo-issues.md

- add remote cluster

- in repositories add gitops repo

- configure project

- configure applications
