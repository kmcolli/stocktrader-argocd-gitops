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
 
 ## Cannot create project nor application
 
 While trying to create project I got the following error:
 
 ```
 Unable to create project: json: cannot unmarshal object into Go struct field ApplicationDestination.project.spec.destinations.server of type string
 ```
 
  And also while creating application the following one:
  
  ```
  Unable to load data: configmap "argocd-gpg-keys-cm" not found
  ```
Solved by manually creating "argocd-gpg-keys-cm" config map -  bug explained here - https://github.com/argoproj-labs/argocd-operator/issues/191

Following cm solved the issue:
```
kind: ConfigMap
apiVersion: v1
metadata:
  name: argocd-gpg-keys-cm
  namespace: argocd
  labels:
    app.kubernetes.io/name: argocd-gpg-keys-cm
    app.kubernetes.io/part-of: argocd

```
