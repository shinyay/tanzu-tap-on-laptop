# Running Tanzu Application Platform Locally on My Mac

- [Tutorial on Tanzu Developer Center](https://tanzu.vmware.com/developer/guides/tanzu-application-platform-local-devloper-install)

## Description

## Demo

## Features

- feature:1
- feature:2

## Requirement

## Usage

## Installation

### 0. Prerequisite

- Hardware
  - 8 threads
  - 16 GB of RAM
  - 40 GB of free disk space.
- Software
  - Minikube
  - Kubectl
- Account
  - [Docker Hub](https://hub.docker.com/)
  - [Tanzu Network](https://network.tanzu.vmware.com/)

#### 0. Remove Old Tanzu CLI

```shell
rm -rf $HOME/tanzu/cli        # Remove previously downloaded cli files
sudo rm /usr/local/bin/tanzu  # Remove CLI binary (executable)
rm -rf ~/.config/tanzu/       # current location # Remove config directory
rm -rf ~/.tanzu/              # old location # Remove config directory
rm -rf ~/.cache/tanzu         # remove cached catalog.yaml
rm -rf ~/Library/Application\ Support/tanzu-cli/* # Remove plug-ins
```

### 1. Download and Install the Tanzu CLI

Instrall Tanzu CLI

```shell
mkdir $HOME/tanzu
tar -xvf tanzu-framework-darwin-amd64.tar -C $HOME/tanzu
set -x TANZU_CLI_NO_INIT true
cd $HOME/tanzu
sudo install cli/core/v0.11.1/tanzu-core-darwin_amd64  /usr/local/bin/tanzu
```

Check Tanzu CLI Version

```shell
tanzu version
```

```shell
version: v0.11.1
buildDate: 2022-02-14
sha: 4d578570
```

Install Tanzu CLI Plugins

```shell
tanzu plugin install --local cli all
```

```shell
Installing plugin 'package:v0.11.1'
Installing plugin 'secret:v0.11.1'
Installing plugin 'apps:v0.4.1'
Installing plugin 'accelerator:v1.0.1'
Installing plugin 'services:v0.1.2'
✔  successfully installed 'all' plugin
```

```shell
tanzu plugin list
```

```shell
  NAME                DESCRIPTION                                                        SCOPE       DISCOVERY  VERSION  STATUS
  login               Login to the platform                                              Standalone  default    v0.11.1  not installed
  management-cluster  Kubernetes management-cluster operations                           Standalone  default    v0.11.1  not installed
  package             Tanzu package management                                           Standalone  default    v0.11.1  installed
  pinniped-auth       Pinniped authentication operations (usually not directly invoked)  Standalone  default    v0.11.1  not installed
  secret              Tanzu secret management                                            Standalone  default    v0.11.1  installed
  services            Discover Service Types and manage Service Instances (ALPHA)        Standalone             v0.1.2   installed
  accelerator         Manage accelerators in a Kubernetes cluster                        Standalone             v1.0.1   installed
  apps                Applications on Kubernetes                                         Standalone             v0.4.1   installed
```

### 2. Run Minikube

Start Minikube with 8 CPUs, 12 GB RAM, and version 1.22 of Kubernetes

```shell
minikube start --cpus='8' --memory='12g' --kubernetes-version='1.22.6'
```

```shell
minikube ip
```

Add a new Hostname entry for TAP

```shell
sudo /etc/hosts
```

```shell
<your-minikube-ip-address> tap-gui.example.com tanzu-java-web-app.default.apps.example.com
```

Open the Minikube tunnel

```shell
minikube tunnel
```

### 3. Install Cluster Essentials for VMware Tanzu onto Minikube

Download

- [Cluster Essentials for VMware Tanzu](https://network.tanzu.vmware.com/products/tanzu-cluster-essentials)
  - tanzu-cluster-essentials-darwin-amd64-1.0.0.tgz

```shell
mkdir $HOME/tanzu/essential
tar xzvf tanzu-cluster-essentials-darwin-amd64-1.0.0.tgz -C $HOME/tanzu/essential
```

```shell
set -x INSTALL_BUNDLE registry.tanzu.vmware.com/tanzu-cluster-essentials/cluster-essentials-bundle@sha256:82dfaf70656b54dcba0d4def85ccae1578ff27054e7533d08320244af7fb0343
set -x INSTALL_REGISTRY_HOSTNAME registry.tanzu.vmware.com
set -x INSTALL_REGISTRY_USERNAME <tanzunet-username>
set -x INSTALL_REGISTRY_PASSWORD <tanzunet-password>
```

Before install.sh

```shell
NAMESPACE     NAME                               READY   STATUS    RESTARTS        AGE
kube-system   coredns-78fcd69978-52ss9           1/1     Running   0               3m43s
kube-system   etcd-minikube                      1/1     Running   0               3m56s
kube-system   kube-apiserver-minikube            1/1     Running   0               3m58s
kube-system   kube-controller-manager-minikube   1/1     Running   0               3m57s
kube-system   kube-proxy-dp68m                   1/1     Running   0               3m44s
kube-system   kube-scheduler-minikube            1/1     Running   0               3m57s
kube-system   storage-provisioner                1/1     Running   1 (3m13s ago)   3m55s
```

```shell
cd $HOME/tanzu/essential
./install.sh
```

Check the current state

```shell
kubectl get pods -o wide --all-namespaces
```

```shell
NAMESPACE              NAME                                    READY   STATUS    RESTARTS        AGE
kapp-controller        kapp-controller-57645fd7bc-zl5fc        1/1     Running   0               61s
kube-system            coredns-78fcd69978-52ss9                1/1     Running   0               5m29s
kube-system            etcd-minikube                           1/1     Running   0               5m42s
kube-system            kube-apiserver-minikube                 1/1     Running   0               5m44s
kube-system            kube-controller-manager-minikube        1/1     Running   0               5m43s
kube-system            kube-proxy-dp68m                        1/1     Running   0               5m30s
kube-system            kube-scheduler-minikube                 1/1     Running   0               5m43s
kube-system            storage-provisioner                     1/1     Running   1 (4m59s ago)   5m41s
secretgen-controller   secretgen-controller-5cd48899cd-7kzq4   1/1     Running   0               18s
```

### 4. Install the Tanzu Application Platform onto Minikube

```shell
# Create Tanzu Application Platform Install Environment Variables
set -x TAP_VERSION "1.0.2"
set -x TAP_NAMESPACE "tap-install"

# Set the developer’s ‘push’ capable docker container registry details
set -x DOCKER_SERVER "https://index.docker.io/v1/"
set -x DOCKER_USERNAME '' # < insert your docker username
set -x DOCKER_PASSWORD '' # < insert your docker password
```

Create the Kubernetes namespace used by the Tanzu Application Platform installation

```shell
kubectl create ns $TAP_NAMESPACE 
```

Add the credentials used to pull the Tanzu Application Platform images from the Tanzu Network

```shell
tanzu secret registry add tap-registry \
  --username $INSTALL_REGISTRY_USERNAME \
  --password $INSTALL_REGISTRY_PASSWORD \
  --server $INSTALL_REGISTRY_HOSTNAME \
  --namespace $TAP_NAMESPACE \
  --export-to-all-namespaces \
  --yes 
```

Add a repository record for the Tanzu Application Platform package repository

```shell
tanzu package repository add tanzu-tap-repository \
  --url $INSTALL_REGISTRY_HOSTNAME/tanzu-application-platform/tap-packages:$TAP_VERSION \
  --namespace $TAP_NAMESPACE 
```

```shell
tanzu package repository get tanzu-tap-repository --namespace $TAP_NAMESPACE 
```

```shell
NAME:          tanzu-tap-repository
VERSION:       1213
REPOSITORY:    registry.tanzu.vmware.com/tanzu-application-platform/tap-packages
TAG:           1.0.2
STATUS:        Reconcile succeeded
REASON:
```

```shell
tanzu package available list --namespace $TAP_NAMESPACE 
```

```shell
  NAME                                                 DISPLAY-NAME                                                              SHORT-DESCRIPTION                                                                                                                                              LATEST-VERSION
  accelerator.apps.tanzu.vmware.com                    Application Accelerator for VMware Tanzu                                  Used to create new projects and configurations.                                                                                                                1.0.2
  api-portal.tanzu.vmware.com                          API portal                                                                A unified user interface to enable search, discovery and try-out of API endpoints at ease.                                                                     1.0.9
  build.appliveview.tanzu.vmware.com                   Application Live View Conventions for VMware Tanzu                        Application Live View convention server                                                                                                                        1.0.2
  buildservice.tanzu.vmware.com                        Tanzu Build Service                                                       Tanzu Build Service enables the building and automation of containerized software workflows securely and at scale.                                             1.4.3
  cartographer.tanzu.vmware.com                        Cartographer                                                              Kubernetes native Supply Chain Choreographer.                                                                                                                  0.2.2
  cnrs.tanzu.vmware.com                                Cloud Native Runtimes                                                     Cloud Native Runtimes is a serverless runtime based on Knative                                                                                                 1.1.1
  controller.conventions.apps.tanzu.vmware.com         Convention Service for VMware Tanzu                                       Convention Service enables app operators to consistently apply desired runtime configurations to fleets of workloads.                                          0.5.1
  controller.source.apps.tanzu.vmware.com              Tanzu Source Controller                                                   Tanzu Source Controller enables workload create/update from source code.                                                                                       0.2.1
  developer-conventions.tanzu.vmware.com               Tanzu App Platform Developer Conventions                                  Developer Conventions                                                                                                                                          0.5.0
  fluxcd.source.controller.tanzu.vmware.com            Flux Source Controller                                                    The source-controller is a Kubernetes operator, specialised in artifacts acquisition from external sources such as Git, Helm repositories and S3 buckets.      0.16.3
  grype.scanning.apps.tanzu.vmware.com                 Grype for Supply Chain Security Tools - Scan                              Default scan templates using Anchore Grype                                                                                                                     1.0.1
  image-policy-webhook.signing.apps.tanzu.vmware.com   Image Policy Webhook                                                      Image Policy Webhook enables defining of a policy to restrict unsigned container images.                                                                       1.0.2
  learningcenter.tanzu.vmware.com                      Learning Center for Tanzu Application Platform                            Guided technical workshops                                                                                                                                     0.1.1
  metadata-store.apps.tanzu.vmware.com                 Supply Chain Security Tools - Store                                       Post SBoMs and query for image, package, and vulnerability metadata.                                                                                           1.0.2
  ootb-delivery-basic.tanzu.vmware.com                 Tanzu App Platform Out of The Box Delivery Basic                          Out of The Box Delivery Basic.                                                                                                                                 0.6.1
  ootb-supply-chain-basic.tanzu.vmware.com             Tanzu App Platform Out of The Box Supply Chain Basic                      Out of The Box Supply Chain Basic.                                                                                                                             0.6.1
  ootb-supply-chain-testing-scanning.tanzu.vmware.com  Tanzu App Platform Out of The Box Supply Chain with Testing and Scanning  Out of The Box Supply Chain with Testing and Scanning.                                                                                                         0.6.1
  ootb-supply-chain-testing.tanzu.vmware.com           Tanzu App Platform Out of The Box Supply Chain with Testing               Out of The Box Supply Chain with Testing.                                                                                                                      0.6.1
  ootb-templates.tanzu.vmware.com                      Tanzu App Platform Out of The Box Templates                               Out of The Box Templates.                                                                                                                                      0.6.1
  run.appliveview.tanzu.vmware.com                     Application Live View for VMware Tanzu                                    App for monitoring and troubleshooting running apps                                                                                                            1.0.2
  scanning.apps.tanzu.vmware.com                       Supply Chain Security Tools - Scan                                        Scan for vulnerabilities and enforce policies directly within Kubernetes native Supply Chains.                                                                 1.0.1
  service-bindings.labs.vmware.com                     Service Bindings for Kubernetes                                           Service Bindings for Kubernetes implements the Service Binding Specification.                                                                                  0.6.1
  services-toolkit.tanzu.vmware.com                    Services Toolkit                                                          The Services Toolkit enables the management, lifecycle, discoverability and connectivity of Service Resources (databases, message queues, DNS records, etc.).  0.5.1
  spring-boot-conventions.tanzu.vmware.com             Tanzu Spring Boot Conventions Server                                      Default Spring Boot convention server.                                                                                                                         0.3.0
  tap-gui.tanzu.vmware.com                             Tanzu Application Platform GUI                                            web app graphical user interface for Tanzu Application Platform                                                                                                1.0.2
  tap-telemetry.tanzu.vmware.com                       Telemetry Collector for Tanzu Application Platform                        Tanzu Application Plaform Telemetry                                                                                                                            0.1.4
  tap.tanzu.vmware.com                                 Tanzu Application Platform                                                Package to install a set of TAP components to get you started based on your use case.                                                                          1.0.2
  tekton.tanzu.vmware.com                              Tekton Pipelines                                                          Tekton Pipelines is a framework for creating CI/CD systems.                                                                                                    0.30.1
  workshops.learningcenter.tanzu.vmware.com            Workshop Building Tutorial                                                Workshop Building Tutorial                                                                                                                                     0.1.1
```

Download a configuration file template

```shell
curl -o tap-values.yml https://raw.githubusercontent.com/benwilcock/tanzu-application-platform-scripts/main/minikube-win/template-tap-values.yml
```

Customize your configuration file template

```yaml
profile: light
ceip_policy_disclosed: true # Installation fails if this is set to 'false'

buildservice:
  kp_default_repository: "index.docker.io/DOCKER_USERNAME/build-service"
  kp_default_repository_username: "DOCKER_USERNAME"
  kp_default_repository_password: "DOCKER_PASSWORD"
  tanzunet_username: "INSTALL_REGISTRY_USERNAME"
  tanzunet_password: "INSTALL_REGISTRY_PASSWORD"
  enable_automatic_dependency_updates: true

supply_chain: basic

ootb_supply_chain_basic:
  registry:
    server: "index.docker.io"
    repository: "DOCKER_USERNAME"
  gitops:
    ssh_secret: ""

tap_gui:
  service_type: ClusterIP
  ingressEnabled: "true"
  ingressDomain: "example.com"
  app_config:
    app:
      baseUrl: http://tap-gui.example.com
    catalog:
      locations:
        - type: url
          target: https://github.com/benwilcock/tap-gui-blank-catalog/blob/main/catalog-info.yaml
    backend:
      baseUrl: http://tap-gui.example.com
      cors:
        origin: http://tap-gui.example.com

metadata_store:
  app_service_type: LoadBalancer # (optional) Defaults to LoadBalancer. Change to NodePort for distributions that don't support LoadBalancer

contour:
  envoy:
    service:
      type: LoadBalancer

cnrs:
  domain_name: "apps.example.com"
```

Install the Tanzu Application Platform onto Minikube

```shell
tanzu package install tap -p tap.tanzu.vmware.com -v $TAP_VERSION \
  --values-file tap-values.yml \
  --namespace $TAP_NAMESPACE
```

It takes around 15 mins to succeed

```shell
 Added installed package 'tap'

________________________________________________________
Executed in  848.52 secs    fish           external
   usr time    1.16 secs    0.15 millis    1.16 secs
   sys time    0.89 secs    1.37 millis    0.89 secs
```

Troubleshooting

```shell
tanzu package installed list -A
```

```shell
  cnrs                      cnrs.tanzu.vmware.com                         1.1.1            Reconcile succeeded                                                   tap-install
  contour                   contour.tanzu.vmware.com                      1.18.2+tap.1     Reconcile failed: Error (see .status.usefulErrorMessage for details)  tap-install
  conventions-controller    controller.conventions.apps.tanzu.vmware.com  0.5.1            Reconcile succeeded                                                   tap-install
```

```shell
tanzu package installed get contour --namespace $TAP_NAMESPACE
```

```shell
NAME:                    contour
PACKAGE-NAME:            contour.tanzu.vmware.com
PACKAGE-VERSION:         1.18.2+tap.1
STATUS:                  Reconcile failed: Error (see .status.usefulErrorMessage for details)
CONDITIONS:              [{ReconcileFailed True  Error (see .status.usefulErrorMessage for details)}]
USEFUL-ERROR-MESSAGE:    I0328 00:42:46.375812    6478 request.go:665] Waited for 1.049046456s due to client-side throttling, not priority and fairness, request: GET:https://10.96.0.1:443/apis/internal.packaging.carvel.dev/v1alpha1?timeout=32s
I0328 00:42:56.377390    6478 request.go:665] Waited for 4.592470949s due to client-side throttling, not priority and fairness, request: GET:https://10.96.0.1:443/apis/bindings.labs.vmware.com/v1alpha1/provisionedservices?labelSelector=kapp.k14s.io%2Fapp%3D1648425534876913546
kapp: Error: Timed out waiting after 5m0s
```

Completed

```shell
  NAME                      PACKAGE-NAME                                  PACKAGE-VERSION  STATUS               NAMESPACE
  accelerator               accelerator.apps.tanzu.vmware.com             1.0.2            Reconcile succeeded  tap-install
  appliveview               run.appliveview.tanzu.vmware.com              1.0.2            Reconcile succeeded  tap-install
  appliveview-conventions   build.appliveview.tanzu.vmware.com            1.0.2            Reconcile succeeded  tap-install
  buildservice              buildservice.tanzu.vmware.com                 1.4.3            Reconcile succeeded  tap-install
  cartographer              cartographer.tanzu.vmware.com                 0.2.2            Reconcile succeeded  tap-install
  cert-manager              cert-manager.tanzu.vmware.com                 1.5.3+tap.1      Reconcile succeeded  tap-install
  cnrs                      cnrs.tanzu.vmware.com                         1.1.1            Reconcile succeeded  tap-install
  contour                   contour.tanzu.vmware.com                      1.18.2+tap.1     Reconcile succeeded  tap-install
  conventions-controller    controller.conventions.apps.tanzu.vmware.com  0.5.1            Reconcile succeeded  tap-install
  developer-conventions     developer-conventions.tanzu.vmware.com        0.5.0            Reconcile succeeded  tap-install
  fluxcd-source-controller  fluxcd.source.controller.tanzu.vmware.com     0.16.3           Reconcile succeeded  tap-install
  ootb-delivery-basic       ootb-delivery-basic.tanzu.vmware.com          0.6.1            Reconcile succeeded  tap-install
  ootb-supply-chain-basic   ootb-supply-chain-basic.tanzu.vmware.com      0.6.1            Reconcile succeeded  tap-install
  ootb-templates            ootb-templates.tanzu.vmware.com               0.6.1            Reconcile succeeded  tap-install
  service-bindings          service-bindings.labs.vmware.com              0.6.1            Reconcile succeeded  tap-install
  services-toolkit          services-toolkit.tanzu.vmware.com             0.5.1            Reconcile succeeded  tap-install
  source-controller         controller.source.apps.tanzu.vmware.com       0.2.1            Reconcile succeeded  tap-install
  spring-boot-conventions   spring-boot-conventions.tanzu.vmware.com      0.3.0            Reconcile succeeded  tap-install
  tap                       tap.tanzu.vmware.com                          1.0.2            Reconcile succeeded  tap-install
  tap-gui                   tap-gui.tanzu.vmware.com                      1.0.2            Reconcile succeeded  tap-install
  tap-telemetry             tap-telemetry.tanzu.vmware.com                0.1.4            Reconcile succeeded  tap-install
  tekton-pipelines          tekton.tanzu.vmware.com                       0.30.1           Reconcile succeeded  tap-install
```

Checkout UI

- [GUI](http://tap-gui.example.com/)
![TAP-GUI](https://user-images.githubusercontent.com/3072734/160526497-a46e65ba-938d-4f43-a6a7-30b0ad0676d2.png)
![Catalog](https://user-images.githubusercontent.com/3072734/160527799-2dd30cdc-0055-4eb5-9566-3ec3cf6c59b7.png)

## 5. Create A Developer Workspace

Add a developer namespace

```shell
set -x TAP_DEV_NAMESPACE "developer"
kubectl create ns $TAP_DEV_NAMESPACE
```

## References

## Licence

Released under the [MIT license](https://gist.githubusercontent.com/shinyay/56e54ee4c0e22db8211e05e70a63247e/raw/34c6fdd50d54aa8e23560c296424aeb61599aa71/LICENSE)

## Author

[shinyay](https://github.com/shinyay)
- twitter: https://twitter.com/yanashin18618
