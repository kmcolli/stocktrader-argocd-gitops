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

## Cannot add cluster via UI, only avaialbe via cli
https://github.com/argoproj/argo-cd/issues/3551

Install argocd cli. 
Loging to argocd using sso  (add : `--grpc-web` if using tls edge termination not passthrough)
```
argocd login --sso argocd-server-argocd.mycluster
WARNING: server certificate had error: x509: certificate signed by unknown authority. Proceed insecurely (y/n)? y
Opening browser for authentication
```



Login to openshift cluster using `oc`

to see already configured clusters type:

```
argocd cluster list
SERVER                                                  NAME         VERSION  STATUS   MESSAGE
https://kubernetes.default.svc                          in-cluster            Unknown  Cluster has no application and not being monitored.
```

to see available clusters type :
```
argocd cluster add
```
to see available clusters
```
argocd cluster add stocktrader/clustername:port/YOURUSER  --name clusterNameForArgo
time="2020-12-10T16:21:15+01:00" level=info msg="ServiceAccount \"argocd-manager\" created in namespace \"kube-system\""
time="2020-12-10T16:21:15+01:00" level=info msg="ClusterRole \"argocd-manager-role\" created"
time="2020-12-10T16:21:15+01:00" level=info msg="ClusterRoleBinding \"argocd-manager-role-binding\" created"
Cluster 'https://clusterName:port' added

```


Issue:

```
services is forbidden: User "system:serviceaccount:argocd:argocd-application-controller" cannot create resource "services" in API group "" in the namespace "stocktrader-argocd-test"
```

ArgoCD user cannot edit namespaces.

Create cluster role:
```
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: argocd-application-controller
  labels:
    app: argocd
    app.kubernetes.io/instance: argocd-rbac
rules:
  - verbs:
      - '*'
    apiGroups:
      - '*'
    resources:
      - '*'
      
```      
And cluster role binding:

```
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: argocd-application-controller
subjects:
  - kind: ServiceAccount
    name: argocd-application-controller
    namespace: argocd
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: argocd-application-controller
```


