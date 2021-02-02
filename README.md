# stocktrader-argocd-gitops

Devops git for Stocktrader app.

Each branch in the repo matches one of the environments:
- test - test environment in the local argo cluster
- qa - qa environment in the local cluster
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
    insecure: false
    route:
      enabled: true
      tls:
        termination: passthrough
      wildcardPolicy: None    
```

In some cases, when self sign certs are not acceptable, you may need to switch security configuration to edge termination and use OCP cert instead of default ArgoCD one. Change to: `insecure: true`  and `termination: edge`

- solve configuration issues - see argo-issues.md

- login to argocd via cmd  (add : --grpc-web if using tls edge termination not passthrough):

```
argocd login --sso argocd-server-argocd.mycluster
```

- Login to target remote ocp cluster using oc, add remote cluster to argo

```
argocd cluster add stocktrader/clustername:port/YOURUSER  --name clusterNameForArgo
```

- in repositories add gitops repo

```
argocd repo add https://github.com/stocktrader-ops/stocktrader-argocd-gitops

repository 'https://github.com/stocktrader-ops/stocktrader-argocd-gitops' added
```

- configure project

```
argocd proj create stocktrader --description "All stocktrader apps" -d https://kubernetes.default.svc,stocktrader-test -d https://kubernetes.default.svc,stocktrader-qa -s https://github.com/stocktrader-ops/stocktrader-argocd-gitops
```

- configure applications; repeat same command for diff envs  (test, qa, prod) changing `revision` and `-dest-xxx` params

```
argocd app create test-trader --repo https://github.com/stocktrader-ops/stocktrader-argocd-gitops --revision test --path trader --dest-server https://kubernetes.default.svc --dest-namespace stocktrader-test --directory-recurse  --project stocktrader --sync-policy auto

application 'test-trader' created
```

## configure pull secret to local quay registry
Go to the robot account: 

https://quay-quay-quay.mycluster-us-sout-363772-a01ee4194ed985a1e32b1d96fd4ae346-0000.us-south.containers.appdomain.cloud/organization/cicd/teams/cicd

Click account and `Kubernetes secret`.

Copy secret to all target namespaces in all clusters that should have access to that registry

# configure pull role for local cluster

```
oc policy add-role-to-user system:image-puller system:serviceaccount:stocktrader-test:default --namespace=stock-quote-quarkus
oc policy add-role-to-user system:image-puller system:serviceaccount:stocktrader-qa:default --namespace=stock-quote-quarkus
oc policy add-role-to-user system:image-puller system:serviceaccount:stocktrader-prod:default --namespace=stock-quote-quarkus
```

