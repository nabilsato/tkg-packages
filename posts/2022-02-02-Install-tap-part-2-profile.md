---
layout: post
autore: "Evangelista Tragni"
title: Install TAP on a Tanzu Kubernetes Grid Service Cluster - Part 2 - Profile and Packages
comments: false
category: blog
---



## Install TAP Package Repository

Before you install the packages, ensure that you have completed the prerequisites, accepted the EULA, and installed the Tanzu CLI with any required plug-ins

Set up environment variables for use during the installation.

```
export INSTALL_REGISTRY_HOSTNAME=registry.tanzu.vmware.com
export INSTALL_REGISTRY_USERNAME=<YOUR-TANZU-NETWORK-USER>
export INSTALL_REGISTRY_PASSWORD=<YOUR-TANZU-NETWORK-PASSWORD>
export SCAN_NAMESPACE=grype
export DEV_NAMESPACE=workload
export MY_REGISTRY_SERVER=<YOUR-REGISTRY-HOSTNAME> 
export MY_REGISTRY_USERNAME=<YOUR-REGISTRY-USERNAME>
export MY_REGISTRY_PASSWORD=<YOUR-REGISTRY-PASSWORD>
```
> **Note** Use the same account with which you accpted the EULA


Create a namespace called tap-install for deploying any packages.

Create a namespace called grype to deploy the scan template. 

Create a namespace called workload in which we will run the applications

Run the following commands:

```
kubectl create ns tap-install
kubectl create ns $SCAN_NAMESPACE
kubectl create ns $DEV_NAMESPACE
```
This namespace keeps the objects grouped together logically.

Create a registry secret by running:
```
tanzu secret registry add tap-registry \
  --username ${INSTALL_REGISTRY_USERNAME} --password ${INSTALL_REGISTRY_PASSWORD} \
  --server ${INSTALL_REGISTRY_HOSTNAME} \
  --export-to-all-namespaces --yes --namespace tap-install
```
Add Tanzu Application Platform package repository to the cluster by running:
```
tanzu package repository add tanzu-tap-repository \
  --url registry.tanzu.vmware.com/tanzu-application-platform/tap-packages:1.0.1 \
  --namespace tap-install
```
Your output will be like this :
> ```
> \ Adding package repository 'tanzu-tap-repository'...
> 
> Added package repository 'tanzu-tap-repository'
> ```

Get the status of the Tanzu Application Platform package repository, and ensure the status updates to Reconcile succeeded by running:


```
tanzu package repository get tanzu-tap-repository -n tap-install
```
For example:

> ```
> - Retrieving repository tap...
> NAME:          tanzu-tap-repository
> VERSION:       121657971
> REPOSITORY:    registry.tanzu.vmware.com/tanzu-application-platform/tap-packages
> TAG:           1.0.1
> STATUS:        Reconcile succeeded
> REASON:
> ```

This step can take up to 5-10 minutes


List the available packages by running:
```
tanzu package available list -n tap-install
```

This wil be the result:
> ```
> / Retrieving available packages...
>   NAME                                                 DISPLAY-NAME                                                              SHORT-DESCRIPTION
>   accelerator.apps.tanzu.vmware.com                    Application Accelerator for VMware Tanzu                                  Used to create new projects and configurations.
>   api-portal.tanzu.vmware.com                          API portal                                                                A unified user interface to enable search, discovery and try-out of API endpoints at ease.
>   run.appliveview.tanzu.vmware.com                     Application Live View for VMware Tanzu                                    App for monitoring and troubleshooting running apps
>   build.appliveview.tanzu.vmware.com                   Application Live View Conventions for VMware Tanzu                        Application Live View convention server
>   buildservice.tanzu.vmware.com                        Tanzu Build Service                                                       Tanzu Build Service enables the building and automation of containerized software workflows securely and at scale.
>   cartographer.tanzu.vmware.com                        Cartographer                                                              Kubernetes native Supply Chain Choreographer.
>   cnrs.tanzu.vmware.com                                Cloud Native Runtimes                                                     Cloud Native Runtimes is a serverless runtime based on Knative
>   controller.conventions.apps.tanzu.vmware.com         Convention Service for VMware Tanzu                                       Convention Service enables app operators to consistently apply desired runtime configurations to fleets of workloads.
>   controller.source.apps.tanzu.vmware.com              Tanzu Source Controller                                                   Tanzu Source Controller enables workload create/update from source code.
>   developer-conventions.tanzu.vmware.com               Tanzu App Platform Developer Conventions                                  Developer Conventions
>   grype.scanning.apps.tanzu.vmware.com                 Grype Scanner for Supply Chain Security Tools - Scan                      Default scan templates using Anchore Grype
>   image-policy-webhook.signing.apps.tanzu.vmware.com    Image Policy Webhook                                                     The Image Policy Webhook allows platform operators to define a policy that will use cosign to verify signatures of container images
>   learningcenter.tanzu.vmware.com                      Learning Center for Tanzu Application Platform                            Guided technical workshops
>   ootb-supply-chain-basic.tanzu.vmware.com             Tanzu App Platform Out of The Box Supply Chain Basic                      Out of The Box Supply Chain Basic.
>   ootb-supply-chain-testing-scanning.tanzu.vmware.com  Tanzu App Platform Out of The Box Supply Chain with Testing and Scanning  Out of The Box Supply Chain with Testing and Scanning.
>   ootb-supply-chain-testing.tanzu.vmware.com           Tanzu App Platform Out of The Box Supply Chain with Testing               Out of The Box Supply Chain with Testing.
>   ootb-templates.tanzu.vmware.com                      Tanzu App Platform Out of The Box Templates                               Out of The Box Templates.
>   scanning.apps.tanzu.vmware.com                       Supply Chain Security Tools - Scan                                        Scan for vulnerabilities and enforce policies directly within Kubernetes native Supply Chains.
>   metadata-store.apps.tanzu.vmware.com                          Tanzu Supply Chain Security Tools - Store                                 The Metadata Store enables saving and querying image, package, and vulnerability data.
>   service-bindings.labs.vmware.com                     Service Bindings for Kubernetes                                           Service Bindings for Kubernetes implements the Service Binding Specification.
>   services-toolkit.tanzu.vmware.com                    Services Toolkit                                                          The Services Toolkit enables the management, lifecycle, discoverability and connectivity of Service Resources (databases, message queues, DNS records, etc.).
>   spring-boot-conventions.tanzu.vmware.com             Tanzu Spring Boot Conventions Server                                      Default Spring Boot convention server.
>   tap-gui.tanzu.vmware.com                             Tanzu Application Platform GUI                                            web app graphical user interface for Tanzu Application Platform
>   tap.tanzu.vmware.com                                 Tanzu Application Platform                                                Package to install a set of TAP components to get you started based on your use case.
>   workshops.learningcenter.tanzu.vmware.com            Workshop Building Tutorial                                             
> ```

## Install TAP Profiles

Tanzu Application Platform can be installed through predefined profiles or through individual packages. In this case we will explains how to install a profile.

For every case of troubleshooting I suggest to search the individual packages installation for the packages that gave you the error

Tanzu Application Platform contains the following two profiles:

* Full 
* Light 

In this case we install a `full profile`

List version information for the package by running:
```
tanzu package available list tap.tanzu.vmware.com -n tap-install
```

Create a `tap-values.yml` file by using the following example 

<!--codeinclude-->
[Full Profile](/images/Tap-static/full-profile.yaml)
<!--/codeinclude-->

Where:

`KP-DEFAULT-REPO` is a writable repository in your registry. Tanzu Build Service dependencies are written to this location. Examples:

-   Harbor has the form kp_default_repository: **my-harbor.io/my-project/build-service**

-   Dockerhub has the form kp_default_repository: **my-dockerhub-user/build-service** or kp_default_repository: **index.docker.io/my-user/build-service**

-   Google Cloud Registry has the form kp_default_repository: **gcr.io/my-project/build-service**

>  For my example is:
> 
>   ${MY_REGISTRY_SERVER}/MY_PROJECT/BUILD-SERVICE

`KP-DEFAULT-REPO-USERNAME` is the username that can write to KP-DEFAULT-REPO. You should be able to docker push to this location with this credential.

>  For my example is:
> 
>   ${MY_REGISTRY_USERNAME}


`KP-DEFAULT-REPO-PASSWORD` is the password for the user that can write to KP-DEFAULT-REPO.

>  For my example is:
> 
>   ${MY_REGISTRY_PASSWORD}

`DESCRIPTOR-NAME` is the name of the descriptor to import automatically. Current available options at time of release:

- **tap-1.0.0-full** contains all dependencies, and is for production use.

- **tap-1.0.0-light** smaller footprint used for speeding up installs. Requires Internet access on the cluster.

`SERVER-NAME` is the hostname of the registry server. Examples:

-   Harbor has the form server: **my-harbor.io**

-   Dockerhub has the form server: **index.docker.io**
     
-   Google Cloud Registry has the form server: **gcr.io**

>  For my example is:
> 
>   ${MY_REGISTRY_SERVER}

`REPO-NAME` is where workload images are stored in the registry. Images are written to SERVER-NAME/REPO-NAME/workload-name. Examples:

-   Harbor has the form repository: **my-project/supply-chain**

-   Dockerhub has the form repository: **my-dockerhub-user**

-   Google Cloud Registry has the form repository: **my-project/supply-chain**

>  For my example is:
> 
>   MY_PROJECT/SUPPLY-CHAIN

`DOMAIN-NAME` has a value such as **learningcenter.<your.domain>**

>  For my example is:
> 
>  desotech.local

`INGRESS-DOMAIN` is the subdomain for the host name that you point at the tanzu-shared-ingress service’s External IP address. **<your.domain>**

>  For my example is:
> 
>  learningcenter.desotech.local

`GIT-CATALOG-URL` is the path to the catalog-info.yaml catalog definition file from either the included Blank catalog  or a Backstage-compliant catalog that you’ve already built and posted on the Git infrastructure you specified in the Integration section.

-   For Blank catalog download **Blank Tanzu Application Platform GUI Catalog** from [Tanzu Network](https://network.tanzu.vmware.com/). 

-   Upload the content to a new public repository

-   Take the URL to the **catalog-info.yaml** from your Repository

`SCAN-NAMESPACE` is the namespace where you want the ScanTemplates to be deployed to. This is the namespace where the scanning feature is going to run. This namespace will be already avaible so 

>  For my example is:
> 
>   ${SCAN_NAMESPACE}


`TARGET-REGISTRY-CREDENTIALS-SECRET` is the name of the secret that contains the credentials to pull an image from the registry for scanning. If built images are pushed to the same registry as the Tanzu Application Platform images, this can reuse the tap-registry secret created in step 3 of Add the Tanzu Application Platform package repository.

>  For my example is:
> 
> 
> -   Create a  secret that  contain your credentials 
> 
>     ```
>     tanzu secret registry add registry-credentials --server ${MY_REGISTRY_SERVER} --username ${MY_REGISTRY_USERNAME} --password ${MY_REGISTRY_PASSWORD} --export-to-all-namespaces --yes --namespace ${SCAN_NAMESPACE}
>     ```
> 
> 

`contour:` By default, Contour uses NodePort as the service type. This will make it to set the service type to LoadBalancer,



In the end it will look like this:

> ```yaml
> profile: full
> ceip_policy_disclosed: true # Installation fails if this is set to 'false'
> buildservice:
>   kp_default_repository: r.deso.tech/tap-build-service/build-service
>   kp_default_repository_username: 'MY_REGISTRY_USERNAME'
>   kp_default_repository_password: 'MY_REGISTRY_PASSWORD'
>   tanzunet_username: TANZU-NETWORK-USERNAME
>   tanzunet_password: TANZU-NETWORK-PASSWORD
>   descriptor_name: tap-1.0.0-full
>   enable_automatic_dependency_updates: true
> supply_chain: basic
> 
> 
> cnrs:
>   domain_name: desotech.local
> 
> ootb_supply_chain_basic:
>   registry:
>     server: "r.deso.tech"
>     repository: "tap-supply-chain/supply-chain"
>   gitops:
>     ssh_secret: ""
> 
> learningcenter:
>   ingressDomain: learningcenter.desotech.local
> 
> tap_gui:
>   service_type: ClusterIP
>   ingressEnabled: "true"
>   ingressDomain: desotech.local
>   app_config:
>     app:
>       baseUrl: http://tap-gui.desotech.local
>     catalog:
>       locations:
>         - type: url
>           target: https://github.com/YOUR_USERNAME/REPOSITORY/catalog-info.yaml"
>     backend:
>       baseUrl: http://tap-gui.desotech.local
>       cors:
>         origin: http://tap-gui.desotech.local
> 
> metadata_store:
>   app_service_type: LoadBalancer # (optional) Defaults to LoadBalancer. Change to NodePort for distributions that don't support LoadBalancer
> 
> grype:
>   namespace: grype #Alredy avaible
>   targetImagePullSecret: registry-credentials #secret already create
> 
> contour:
>   envoy:
>     service:
>       type: LoadBalancer
> ```


## Set Up a Namespace For Workload

We have already creeate a namespace called Workload that we will use like a DEV_NAMESPACE and place all the application workload to it.

Verify that the secrt `registry-credentials` is already present on the `workload` namespace:
```
tanzu secret registry list -n workload
```
> ```
> | Retrieving registry secrets...
>   NAME                  REGISTRY       EXPORTED           AGE
>   registry-credentials  r.deso.tech    to all namespaces  6d21h
>   tap-registry          10.10.1.3, +2  not exported       6d17h
> ```

> - If  the secret is not here , run the following command.
>   
> ```
> tanzu secret registry add registry-credentials --server ${MY_REGISTRY_SERVER} --username ${MY_REGISTRY_USERNAME} --password ${MY_REGISTRY_PASSWORD} --export-to-all-namespaces --yes --namespace ${DEV_NAMESPACE}
> ```
Add placeholder read secrets, a service account, and RBAC rules to the developer namespace.

This will rewrite the `.dockerconfigjson` value of the **tap-registry** with an empty crypted value. This change will apply only on the `${DEV_NAMESPACE}`

Run the following command:


```bash
kubectl -n ${DEV_NAMESPACE} apply -f - <<EOF 
apiVersion: v1
kind: Secret
metadata:
  name: tap-registry
  annotations:
    secretgen.carvel.dev/image-pull-secret: ""
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: e30K
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: default
secrets:
  - name: registry-credentials
imagePullSecrets:
  - name: registry-credentials
  - name: tap-registry
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: default
rules:
- apiGroups: [source.toolkit.fluxcd.io]
  resources: [gitrepositories]
  verbs: ['*']
- apiGroups: [source.apps.tanzu.vmware.com]
  resources: [imagerepositories]
  verbs: ['*']
- apiGroups: [carto.run]
  resources: [deliverables, runnables]
  verbs: ['*']
- apiGroups: [kpack.io]
  resources: [images]
  verbs: ['*']
- apiGroups: [conventions.apps.tanzu.vmware.com]
  resources: [podintents]
  verbs: ['*']
- apiGroups: [""]
  resources: ['configmaps']
  verbs: ['*']
- apiGroups: [""]
  resources: ['pods']
  verbs: ['list']
- apiGroups: [tekton.dev]
  resources: [taskruns, pipelineruns]
  verbs: ['*']
- apiGroups: [tekton.dev]
  resources: [pipelines]
  verbs: ['list']
- apiGroups: [kappctrl.k14s.io]
  resources: [apps]
  verbs: ['*']
- apiGroups: [serving.knative.dev]
  resources: ['services']
  verbs: ['*']
- apiGroups: [servicebinding.io]
  resources: ['servicebindings']
  verbs: ['*']
- apiGroups: [services.apps.tanzu.vmware.com]
  resources: ['resourceclaims']
  verbs: ['*']
- apiGroups: [scanning.apps.tanzu.vmware.com]
  resources: ['imagescans', 'sourcescans']
  verbs: ['*']
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: default
subjects:
  - kind: ServiceAccount
    name: default
EOF
```

> ```
> secret/tap-registry configured
> serviceaccount/default configured
> role.rbac.authorization.k8s.io/default created
> rolebinding.rbac.authorization.k8s.io/default created
> ```

## Install All Packages


Run the command below to install all avaible packages with thee values of the tap-value.yaml

```
tanzu package install tap -p tap.tanzu.vmware.com -v 1.0.1 --values-file tap-values.yml -n tap-install --poll-timeout 30m
```
 
![image](/images/Tap-static/package-tap-install.png)

>this may take up to 10-15 min because it installs several packages on your cluster

Verify that all the necessary packages in the profile are installed successfully  by running:

```
tanzu package installed list -A
```
All the packages MUST have the `Reconcile Succeeded` 

![image](/images/Tap-static/reconcile-succeeded.png)

> If some packages don't result succeeded you can search the reason with 
> ```
> tanzu packages installed get <PACKAGE_NAME> -n tap-install
> ```

List all the service on your cluster and take the EXTERNAL_IP_service_type to connect to your TAP gui.

> If you don't have a Record DNS alredy created ensure that from your student machine you are able to resolve `tap-gui.your.domain` ---> `<EXTERNAL_IP_service_type>`, populate your `/etc/hosts` if needed.

Open a tab on your browser and navigate to it. In my example it is:

```
tap-gui.desotech.local
```


Great!! You have installed a Tanzu Application Platform oveer a Tanzu Kubernetes Grid Service cluster

Well Done!

---
Evangelista Tragni

Desotech srl