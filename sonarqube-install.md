# This document describes installation and setup of the SonarQube in OCP cluster

## Install using Helm chart

Get the latest Helm chart from here:

```
git clone https://github.com/SonarSource/helm-chart-sonarqube.git
cd helm-chart-sonarqube/charts/sonarqube
helm dependency update
```
You can get already customized version with route support from here: https://github.com/stocktrader-ops/helm-chart-sonarqube


Customize deployment for OpenShift. Change the following settings in the `helm-chart-sonarqube\charts\sonarqube\values.yaml` file:

```
OpenShift:
  enabled: true
  
  securityContext:
    # For standard Kubernetes deployment, set enabled=true
    # If using OpenShift, enabled=false for restricted SCC and enabled=true for anyuid/nonroot SCC
    enabled: false  
  
  volumePermissions:
    # For standard Kubernetes deployment, set enabled=false
    # For OpenShift, set enabled=true and ensure to set volumepermissions.securitycontext.runAsUser below.
    enabled: true
    # if using restricted SCC set runAsUser: "auto" and if running under anyuid/nonroot SCC - runAsUser needs to match runAsUser above
    securityContext:
      runAsUser: "auto"
      
# For OpenShift set create=true to ensure service account is created.
serviceAccount:
  create: true  
```

Install chart with the following commands (login to your OCP cluster first):

```
oc new-project sonarqube
cd helm-chart-sonarqube/charts/sonarqube
helm upgrade --install -f values.yaml -n sonarqube sonarqube ./
```

If you see the follwing message chart is installed successfully
```
I0706 12:37:25.645523    3900 request.go:621] Throttling request took 1.1159359s, request: GET:https://yourcluster/apis/route.openshift.io/v1?timeout=32s
Release "sonarqube" has been upgraded. Happy Helming!
NAME: sonarqube
LAST DEPLOYED: Tue Jul  6 12:37:26 2021
NAMESPACE: sonarqube
STATUS: deployed
REVISION: 4
NOTES:
1. Get the application URL by running these commands:
  export ROUTE_HOST=$(kubectl get route sonarqube --namespace sonarqube -o jsonpath="{.spec.host}")
  export PROTOCOL="http"
     export PROTOCOL="https"
  echo $PROTOCOL://$ROUTE_HOST
```

### Create Route
If you are not using OpenShift customized version of the chart, you need to create route to access SonarQube.  
Either create route via web UI with the following settings:
- name: sonarqube
- service: sonarqube-sonarqube
- Target Port: 9000 -> http (TCP)
- Secure route: checked
- TLS termination: edge

Or apply following yaml:

```
kind: Route
apiVersion: route.openshift.io/v1
metadata:
  annotations:
    meta.helm.sh/release-name: sonarqube
    meta.helm.sh/release-namespace: sonarqube
    openshift.io/host.generated: 'true'
  name: sonarqube
  namespace: sonarqube
  labels:
    app: sonarqube
    app.kubernetes.io/managed-by: Helm
    chart: sonarqube-1.0.15
    heritage: Helm
    release: sonarqube
spec:
  to:
    kind: Service
    name: sonarqube-sonarqube
  port:
    targetPort: http
  tls:
    termination: edge
```

## Access and configure SonarQube




