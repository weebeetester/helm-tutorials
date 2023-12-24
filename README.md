# Overview
Helm(https://helm.sh/) is the package manager for Kubernetes. Helm is a tool that helps you manage Kubernetes applications with charts, which are easy to create, version, share, and publish.

Purpose of this document is to help learn helm and it's associated concepts, with hands-on approach. Information on this page comes from various sources including official helm documentation. 


# Helm Installation
## Docs: https://helm.sh/docs/intro/install/
```bash
>curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
>chmod 700 get_helm.sh
>./get_helm.sh
```


# Helm  Usage

### As a prerequisite, make sure that k8s cluster is running.
>minikube start --vm=true -p cluster-1



## Create sample chart
>helm create helloworld

## Install chart on K8S cluster
### demohelloworld will be the name of app to be deployed on K8S
```bash
>helm install demohelloworld helloworld
NAME: demohelloworld
LAST DEPLOYED: Sat Dec 23 21:01:39 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services demohelloworld)
  export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
```


## List all the deployed helm charts
```bash
>helm list -a
NAME          	NAMESPACE	REVISION	UPDATED                             	STATUS  	CHART           	APP VERSION
demohelloworld	default  	1       	2023-12-23 21:01:39.093674 +0100 CET	deployed	helloworld-0.1.0	1.16.0
```

## Test the deployed app
### Get the port from K8S nodeport service
```bash
>kubectl get service
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
demohelloworld   NodePort    10.107.196.85   <none>        80:30682/TCP   3m38s
kubernetes       ClusterIP   10.96.0.1       <none>        443/TCP        129m
```
### Access application using MINIKUBE-IP:PORT
```bash
>minikube ip -p cluster-1
192.168.73.2
>curl 192.168.73.2:30682
<!DOCTYPE html>
<html>
..
```

#### **SideNote: To view default K8s dashboard
>minikube dashboard -p cluster-1


## Upgrade the helm chart
### If we make any change in app or in chart, we should run the upgrade. This will increase the revision-number (shown in helm list)
```bash
>helm upgrade demohelloworld helloworld
Release "demohelloworld" has been upgraded. Happy Helming!
NAME: demohelloworld
LAST DEPLOYED: Sat Dec 23 21:12:40 2023
NAMESPACE: default
STATUS: deployed
REVISION: 2
NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services demohelloworld)
  export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
```

### Verify that revision has been increased.
```bash
>helm list -a
NAME          	NAMESPACE	REVISION	UPDATED                             	STATUS  	CHART           	APP VERSION
demohelloworld	default  	2       	2023-12-23 21:12:40.304711 +0100 CET	deployed	helloworld-0.1.0	1.16.0
```
## Rollback the helm chart
### If we want to go back to a specific revision, we can do that. In this example, we are at version 2 and will rollabck to 1.
```bash
>helm rollback demohelloworld 1
Rollback was a success! Happy Helming!
```

### Verify that revision has been increased to 3.
#### Do note that above change is a change afterall. create, change, rollback : total 3 revisions
```bash
>helm list -a
NAME          	NAMESPACE	REVISION	UPDATED                             	STATUS  	CHART           	APP VERSION
demohelloworld	default  	3       	2023-12-23 21:18:36.355869 +0100 CET	deployed	helloworld-0.1.0	1.16.0
```


## Pre-deployment Validation
### Debug & dry-run
#### Validate your helm chart before install. This will actually validate against K8S api-server.
```bash
>helm install demohelloworld --debug --dry-run helloworld
install.go:214: [debug] Original chart version: ""
install.go:231: [debug] CHART PATH: /Users/weebeetester/workspace/repositories/helm/helloworld

NAME: demohelloworld
LAST DEPLOYED: Sat Dec 23 21:28:34 2023
NAMESPACE: default
STATUS: pending-install
REVISION: 1
USER-SUPPLIED VALUES:
{}

COMPUTED VALUES:
affinity: {}
autoscaling:
  enabled: false
  maxReplicas: 100
  minReplicas: 1
  targetCPUUtilizationPercentage: 80
fullnameOverride: ""
image:
  pullPolicy: IfNotPresent
  repository: nginx
  tag: ""
imagePullSecrets: []
ingress:
  annotations: {}
  className: ""
  enabled: false
  hosts:
  - host: chart-example.local
    paths:
    - path: /
      pathType: ImplementationSpecific
  tls: []
nameOverride: ""
nodeSelector: {}
podAnnotations: {}
podLabels: {}
podSecurityContext: {}
replicaCount: 2
resources: {}
securityContext: {}
service:
  port: 80
  type: NodePort
serviceAccount:
  annotations: {}
  automount: true
  create: true
  name: ""
tolerations: []
volumeMounts: []
volumes: []

HOOKS:
---
# Source: helloworld/templates/tests/test-connection.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "demohelloworld-test-connection"
  labels:
    helm.sh/chart: helloworld-0.1.0
    app.kubernetes.io/name: helloworld
    app.kubernetes.io/instance: demohelloworld
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
  annotations:
    "helm.sh/hook": test
spec:
  containers:
    - name: wget
      image: busybox
      command: ['wget']
      args: ['demohelloworld:80']
  restartPolicy: Never
MANIFEST:
---
# Source: helloworld/templates/serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: demohelloworld
  labels:
    helm.sh/chart: helloworld-0.1.0
    app.kubernetes.io/name: helloworld
    app.kubernetes.io/instance: demohelloworld
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
automountServiceAccountToken: true
---
# Source: helloworld/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: demohelloworld
  labels:
    helm.sh/chart: helloworld-0.1.0
    app.kubernetes.io/name: helloworld
    app.kubernetes.io/instance: demohelloworld
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app.kubernetes.io/name: helloworld
    app.kubernetes.io/instance: demohelloworld
---
# Source: helloworld/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demohelloworld
  labels:
    helm.sh/chart: helloworld-0.1.0
    app.kubernetes.io/name: helloworld
    app.kubernetes.io/instance: demohelloworld
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
spec:
  replicas: 2
  selector:
    matchLabels:
      app.kubernetes.io/name: helloworld
      app.kubernetes.io/instance: demohelloworld
  template:
    metadata:
      labels:
        helm.sh/chart: helloworld-0.1.0
        app.kubernetes.io/name: helloworld
        app.kubernetes.io/instance: demohelloworld
        app.kubernetes.io/version: "1.16.0"
        app.kubernetes.io/managed-by: Helm
    spec:
      serviceAccountName: demohelloworld
      securityContext:
        {}
      containers:
        - name: helloworld
          securityContext:
            {}
          image: "nginx:1.16.0"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
          resources:
            {}

NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services demohelloworld)
  export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
 

```


### Helm template
#### Validates and renders the chart template locally, without connecting with K8s api-server.
```bash
 > helm template helloworld
---
# Source: helloworld/templates/serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: release-name-helloworld
  labels:
    helm.sh/chart: helloworld-0.1.0
    app.kubernetes.io/name: helloworld
    app.kubernetes.io/instance: release-name
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
automountServiceAccountToken: true
---
# Source: helloworld/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: release-name-helloworld
  labels:
    helm.sh/chart: helloworld-0.1.0
    app.kubernetes.io/name: helloworld
    app.kubernetes.io/instance: release-name
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app.kubernetes.io/name: helloworld
    app.kubernetes.io/instance: release-name
---
# Source: helloworld/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: release-name-helloworld
  labels:
    helm.sh/chart: helloworld-0.1.0
    app.kubernetes.io/name: helloworld
    app.kubernetes.io/instance: release-name
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
spec:
  replicas: 2
  selector:
    matchLabels:
      app.kubernetes.io/name: helloworld
      app.kubernetes.io/instance: release-name
  template:
    metadata:
      labels:
        helm.sh/chart: helloworld-0.1.0
        app.kubernetes.io/name: helloworld
        app.kubernetes.io/instance: release-name
        app.kubernetes.io/version: "1.16.0"
        app.kubernetes.io/managed-by: Helm
    spec:
      serviceAccountName: release-name-helloworld
      securityContext:
        {}
      containers:
        - name: helloworld
          securityContext:
            {}
          image: "nginx:1.16.0"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
          resources:
            {}
---
# Source: helloworld/templates/tests/test-connection.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "release-name-helloworld-test-connection"
  labels:
    helm.sh/chart: helloworld-0.1.0
    app.kubernetes.io/name: helloworld
    app.kubernetes.io/instance: release-name
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
  annotations:
    "helm.sh/hook": test
spec:
  containers:
    - name: wget
      image: busybox
      command: ['wget']
      args: ['release-name-helloworld:80']
  restartPolicy: Never
 
 ```

### Helm Lint
#### Verify if there are any errors or misconfigurations associated with your helm chart.

```bash
>helm lint helloworld
==> Linting helloworld
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```




## Uninstall a helm chart
### It will uninstall the chart, as a result the app will be removed from cluster
```bash
>helm uninstall demohelloworld
release "demohelloworld" uninstalled
 >helm list 
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
```




<hr>

# Other realted tooling

## helmfile
Helmfile is another wrapper on top of helm chart.
https://github.com/helmfile/helmfile

>helmfile sync


Using helmfile. we can 
- manage multiple helm charts via a single helmfile template(yaml)
- isolate deployment among different environments (dev, test, prod)
- ..

## helm-git
It's a handy utility which can be used to refer to remote helm repos.
https://github.com/aslafy-z/helm-git




<hr>

# Helm Repository Management

### Prebuild charts made available in helm repo(by different providers) which we can search and deploy onto our clusters.

## List all the repos named 'wordpress' available on helm hub
```bash
>helm search hub wordpress
 
URL                                                     CHART VERSION   APP VERSION             DESCRIPTION                                       
https://artifacthub.io/packages/helm/wordpress-...      1.0.2           1.0.0                   A Helm chart for deploying Wordpress+Mariadb st...
https://artifacthub.io/packages/helm/kube-wordp...      0.1.0           1.1                     this is my wordpress package                      
https://artifacthub.io/packages/helm/truecharts...      4.0.5           6.4.2                   The WordPress rich content management system ca...
https://artifacthub.io/packages/helm/shubham-wo...      0.1.0           1.16.0                  A Helm chart for Kubernetes                       
https://artifacthub.io/packages/helm/bitnami-ak...      15.2.13         6.1.0                   WordPress is the world's most popular blogging ...
https://artifacthub.io/packages/helm/bitnami/wo...      19.0.4          6.4.2                   WordPress is the world's most popular blogging ...
https://artifacthub.io/packages/helm/camptocamp...      0.6.10          4.8.1                   Web publishing platform for building blogs and ...
https://artifacthub.io/packages/helm/sikalabs/w...      0.2.0                                   Simple Wordpress                                  
https://artifacthub.io/packages/helm/riftbit/wo...      12.1.16         5.8.1                   Web publishing platform for building blogs and ...
https://artifacthub.io/packages/helm/devops/wor...      0.12.0          1.16.0                  Wordpress helm chart                              
https://artifacthub.io/packages/helm/skywordpre...      0.1.0           1.16.0                  A Helm chart for WordPress                        
https://artifacthub.io/packages/helm/groundhog2...      0.10.2          6.4.2-apache            A Helm chart for Wordpress on Kubernetes          
https://artifacthub.io/packages/helm/rock8s/wor...      0.0.1           latest                  open source software you can use to create a be...
https://artifacthub.io/packages/helm/mcouliba/w...      0.1.0           1.16.0                  A Helm chart for Kubernetes                       
https://artifacthub.io/packages/helm/homeenterp...      0.5.0           5.9.3-php8.1-apache     Blog server                                       
https://artifacthub.io/packages/helm/wordpress-...      6.1.1-3.0rc5    master                  Lightweight Wordpress installation with additio...
https://artifacthub.io/packages/helm/wordpress-...      1.0.0           1.1                     This is a package for configuring wordpress and...
https://artifacthub.io/packages/helm/securecode...      4.4.0-alpha.2   4.0                     Insecure & Outdated Wordpress Instance: Never e...
https://artifacthub.io/packages/helm/wordpressm...      1.0.0                                   This is the Helm Chart that creates the Wordpre...
https://artifacthub.io/packages/helm/bitpoke/wo...      0.12.2          v0.12.2                 Bitpoke WordPress Operator Helm Chart             
https://artifacthub.io/packages/helm/bitpoke/wo...      0.12.4          v0.12.4                 Helm chart for deploying a WordPress site on Bi...
https://artifacthub.io/packages/helm/kube-wordp...      0.1.0           0.0.1-alpha             Helm Chart for Wordpress installation on MySQL ...
https://artifacthub.io/packages/helm/riotkit-or...      2.1.0-alpha4    v2.1-alpha4             Lightweight Wordpress installation with additio...
https://artifacthub.io/packages/helm/phntom/bin...      0.0.4           0.0.3                   www.binaryvision.co.il static wordpress           
https://artifacthub.io/packages/helm/gh-shessel...      2.1.12          6.1.1                   Web publishing platform for building blogs and ...
https://artifacthub.io/packages/helm/sikalabs/w...      0.1.2                                                                                     
https://artifacthub.io/packages/helm/gh-shessel...      6.3.1           6.3.1                   Web publishing platform for building blogs and ...
https://artifacthub.io/packages/helm/wordpressh...      0.1.0           1.16.0                  A Helm chart for Kubernetes                       
https://artifacthub.io/packages/helm/wordpress-...      0.1.0           1.1                     this is chart create the wordpress with suitabl...
https://artifacthub.io/packages/helm/wordpress-...      0.0.1                                   Helm Chart for wordpress-gatsby                   
https://artifacthub.io/packages/helm/bitpoke/bi...      1.8.14          1.8.14                  The Bitpoke App for WordPress provides a versat...
https://artifacthub.io/packages/helm/sonu-wordp...      1.0.0           2                       This is my custom chart to deploy wordpress and...
https://artifacthub.io/packages/helm/wordpress/...      0.2.0           1.1.0                   Wordpress for Kubernetes                          
https://artifacthub.io/packages/helm/uvaise-wor...      0.2.0           1.1.0                   Wordpress for Kubernetes                          
https://artifacthub.io/packages/helm/wordpress-...      1.0.0           2                       This is my custom chart to deploy wordpress and...
https://artifacthub.io/packages/helm/bitpoke/stack      0.12.4          v0.12.4                 Your Open-Source, Cloud-Native WordPress Infras...
https://artifacthub.io/packages/helm/securecode...      4.4.0-alpha.2   v3.8.25                 A Helm chart for the WordPress security scanner...
https://artifacthub.io/packages/helm/wordpresss...      1.1.0           5.8.2                   Web publishing platform for building blogs and ...
https://artifacthub.io/packages/helm/viveksahu2...      1.0.0           2                       This is my custom chart to deploy wordpress and...
https://artifacthub.io/packages/helm/six/wordress       0.2.0           1.1.0                   Wordpress for Kubernetes                          
https://artifacthub.io/packages/helm/jinchi-cha...      0.2.0           1.1.0                   Wordpress for Kubernetes                          
https://artifacthub.io/packages/helm/projet-dev...      0.1.0           1.16.0                  A Helm chart for wordpress deployed on Azure Ku...
https://artifacthub.io/packages/helm/wordpressm...      0.1.0           1.1                                                                       
 ```



 ##  List local helm chart repos
 ### It will also show the remote repos already added to your local helm environment
 ```bash
>helm repo list
Error: no repositories to show
## that means We do not have any repos added yet.
 ```

 ## Add bitnami repo to your local repo list

```bash
>helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories

>helm repo list 
NAME    URL                               
bitnami https://charts.bitnami.com/bitnami
 ```

 ## Get all the wordpress version available in repos added to your helm environment
 
```bash
 >helm search repo wordpress --versions
NAME                    CHART VERSION   APP VERSION     DESCRIPTION                                       
bitnami/wordpress       19.0.4          6.4.2           WordPress is the world's most popular blogging ...
bitnami/wordpress       19.0.3          6.4.2           WordPress is the world's most popular blogging ...
bitnami/wordpress       19.0.2          6.4.2           WordPress is the world's most popular blogging ...
bitnami/wordpress       19.0.1          6.4.2           WordPress is the world's most popular blogging ...
bitnami/wordpress       19.0.0          6.4.2           WordPress is the world's most popular blogging ...
bitnami/wordpress       18.1.30         6.4.2           WordPress is the world's most popular blogging ...
bitnami/wordpress       18.1.29         6.4.2           WordPress is the world's most popular blogging ...
...
 ```

## To see the installation instructions (README.MD) for a perticular chart
```bash
>helm show readme bitnami/wordpress --version 19.0.4

<!--- app-name: WordPress -->

# Bitnami package for WordPress

WordPress is the world's most popular blogging and content management platform. Powerful yet simple, everyone from students to global corporations use it to build beautiful, functional websites.

[Overview of WordPress](http://www.wordpress.org)

...

 ```


## To see the Values used in a perticular chart
```bash
>helm show values bitnami/wordpress --version 19.0.4

...
 ```




 ## Helm Hooks
 ### Helm hooks can be used to perform an activity before/after actual chart action. https://helm.sh/docs/topics/charts_hooks/. Hooks can even be actual K8s resources (deployement, service, batch,..). Hooks should be added under templates/hooks directory (for example helloworld/templates/hooks/pre-install.yml).

 ## Helm Test
 ### Kind of unit tests to vaidate your helm charts.
 ### When a helm chart is crated, there is already a default test config created under /templates/tests.
```bash
>vi /templates/tests/test-connection.yaml
 apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "sample-node-web-app.fullname" . }}-test-connection"
  labels:
    {{- include "sample-node-web-app.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": test
spec:
  containers:
    - name: wget
      image: busybox
      command: ['wget']
      args: ['{{ include "sample-node-web-app.fullname" . }}:{{ .Values.service.port }}']
  restartPolicy: Never
```

### Once helm chart is deployed, you can run above test. Above test is actually just making a wget to name:port

### Install helm chart
```bash
>helm install demohelloworld helloword
NAME: demohelloworld
LAST DEPLOYED: Sun Dec 24 19:37:44 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services demohelloworld)
  export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
```
### Test helm chart (which in tunr will perform >wget name:port)
```bash
> helm test demohelloworld
NAME: demohelloworld
LAST DEPLOYED: Sun Dec 24 19:37:44 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE:     demohelloworld-test-connection
Last Started:   Sun Dec 24 19:38:01 2023
Last Completed: Sun Dec 24 19:38:06 2023
Phase:          Succeeded
NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services demohelloworld)
  export NODE_IP=$(kubectl get no
```