# Issues found during configuration of ArgoCD

## Cannot access github repo
Error while configuring repository:
```
Unable to connect HTTPS repository: permission denied: repositories, create, https://github.com/stocktrader-ops/stocktrader-argocd-gitops.git, 
sub: CiRmZTk5NDBiYS02NmQwLTRkOTMtODRhNC1lNDM4Yzk5ODY5YjcSCW9wZW5zaGlmdA, iat: 2020-12-10T09:08:00Z
```
Solved by changing modifying `argocd-rbac-cm` config map with following values:

```
data:
  policy.csv: |
    g, argocd-admins, role:admin
  policy.default: 'role:admin'
  scopes: '[groups]'
 ``` 
