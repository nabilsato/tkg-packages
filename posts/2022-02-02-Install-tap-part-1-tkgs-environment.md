---
layout: post
autore: "Evangelista Tragni"
title: Install TAP on a Tanzu Kubernetes Grid Service Cluster
comments: false
category: blog
---

# Tanzu Application Platform

VMware Tanzu Application Platform is a modular, application detecting platform that provides a rich set of developer tools and a paved path to production to build and deploy software quickly and securely on any compliant public cloud or on-premises Kubernetes cluster.

Tanzu Application Platform delivers a superior developer experience for enterprises building and deploying cloud-native applications on Kubernetes. It enables application teams to get to production faster by automating source-to-production pipelines. It clearly defines the roles of developers and operators so they can work collaboratively and integrate their efforts.

Customers can simplify workflows in both the inner loop and outer loop of Kubernetes-based app development with Tanzu Application Platform while creating supply chains.

Inner Loop:
* The inner loop describes a developer’s development cycle of iterating on code.
* Inner loop activities include coding, testing, and debugging before making a commit.
* On cloud-native or Kubernetes platforms, developers in the inner loop often build container images and connect their apps to all necessary services and APIs to deploy them to a development environment.

Outer Loop:

* The outer loop describes how operators deploy apps to production and maintain them over time.
* On a cloud-native platform, outer loop activities include building container images, adding container security, and configuring continuous integration and continuous delivery (CI/CD) pipelines.
* Outer loop activities are challenging in a Kubernetes-based development environment due to app delivery platforms being constructed from various third-party and open source components with numerous configuration options.

Tanzu Application Platform provides a default set of components that automates pushing an app to staging and production on Kubernetes, removing the pain points for both inner and outer loops. In addition, it allows the operators to customize the platform by replacing Tanzu Application Platform components with other products.


![image](/images/Tap-static/tap-api.png)



## Prerequisites

The following are strictly required to install Tanzu Application Platform. Please ensure that you have succsessfully completed all the requirement


1. A `Tanzu Network` account to download Tanzu Application Platform packages.

2. A `container image registry`, such as Harbor or Docker Hub for application images, base images, and runtime dependencies
> If installing using the `lite` descriptor for Tanzu Build Service, 1 GB of available storage is recommended.
>
> If installing using the `full` descriptor for Tanzu Build Service, which is intended for production use and offline environments, 10 GB of available storage is recommended.

3. Registry credentials with push and write access made available to Tanzu Application Platform to store images

4. Latest version of Chrome, Firefox, or Edge. Tanzu Application Platform GUI currently does not support Safari browser

5. `Git repository` for the Tanzu Application Platform GUI’s software catalogs, along with a token allowing read access.

6. `Tanzu Application Platform GUI Blank Catalog` from the Tanzu Application section of Tanzu Network
> To install, navigate to Tanzu Network. Under the list of available files to download, there is a folder titled tap-gui-catalogs-latest. Inside that folder is a compressed archive titled Tanzu Application Platform GUI Blank Catalog. You must extract that catalog to the preceding Git repository of choice. This serves as the configuration location for your Organization’s Catalog inside Tanzu Application Platform GUI.

7. `Kubernetes cluster` versions 1.20, 1.21, or 1.22 

8. To deploy all Tanzu Application Platform packages, your cluster must have at least:

 - 8 GB of RAM across all nodes available to Tanzu Application Platform

 - 8 CPUs for i9 (or equivalent) available to Tanzu Application Platform components

 - 12 CPUs for i7 (or equivalent) available to Tanzu Application Platform components

 - 12 GB of RAM is available to build and deploy applications, including Minikube VMware recommends 16 GB of RAM for an optimal experience.

 - 70 GB of disk space available per node

9. For the full profile, or use of Security Chain Security Tools - Store, your cluster must have a configured default `StorageClass`.

10. `Pod Security Policies` must be configured so that Tanzu Application Platform controller pods can run as root

11. The `Kubernetes CLI`, kubectl, v1.20, v1.21 or v1.22, installed and authenticated with administrator rights for your target cluster.


## Accept the EULAs

An end-user license agreement is a legal contract entered into between a software developer or vendor and the user of the software, often where the software has been purchased by the user from an intermediary such as a retailer. A EULA specifies in detail the rights and restrictions which apply to the use of the software.

Sign in to Tanzu Network from the link below. You will sign in with the same account later to download several packages. Remember the account with which you accept EULA.

[Tanzu Network](https://network.tanzu.vmware.com/)


Select the `Click here to sign the EULA` link in the yellow warning box under the release drop down as seen in the following screen shot.

![image](/images/Tap-static/EULAS-1.png)

Click `Agree` at the bottom of the page so that you accept all the EULA's for TAP


##  Prepare your Environment

By using vSphere with Tanzu you can turn a vSphere cluster to a platform for running Kubernetes workloads in dedicated resource pools. Once enabled on a vSphere cluster, vSphere with Tanzu creates a Kubernetes control plane directly in the hypervisor layer. You can then run Kubernetes containers by deploying vSphere Pods, or you can create upstream Kubernetes clusters through the VMware Tanzu Kubernetes Grid Service and run your applications inside these clusters.

In this case we will install TAP over a Tanzu Kubernetes cluster.

A Tanzu Kubernetes cluster is a full distribution of the open-source Kubernetes container orchestration platform that is built, signed, and supported by VMware. You can provision and operate Tanzu Kubernetes clusters on the Supervisor Cluster by using the Tanzu Kubernetes Grid Service. A Supervisor Cluster is a vSphere cluster that is enabled with vSphere with Tanzu.

This specific case grant us a full installed vSphere with Tanzu installation, we have to  deploy a new Tanzu kubernetes Grid Service cluster in the namespace alredy create for us.

Lets' start with the download  of Kubernetes CLI Tools on our machine.

if you have access to the vCenter Server, get the link as follows:

* Log in to the `vCenter Server` using the vSphere Client.
* Navigate to `vSphere Cluster > Namespaces` and select the vSphere Namespace where you are working. In the example shown below, "hpe-test" is the namespace that we created for our Tanzu Kubernetes clusters.
* Select the `Summary` tab and locate the Status area on this page.
* Select Open beneath the `Link to CLI Tools` heading to open the download page. Or, you can Copy the link.

![image](/images/Tap-static/cli-tools.png)

Navigate to the Kubernetes CLI Tools download URL for your environment. Refere to the prerequisites section above for guidance on how to locate the download URL.

![image](/images/Tap-static/download-cli.png)

Select your Operating System and click on `Download the vsphere-plugin.zip` button .

Add the location of both executables to your system's PATH variable

To verify the installation of the vSphere Plugin for kubectl, run the command 

```
kubectl vsphere
```

> ```
> vSphere Plugin for kubectl.
> 
> Usage:
>   kubectl-vsphere [command]
> 
> Available Commands:
>   help        Help about any command
>   login       Authenticate user with vCenter Namespaces
>   logout      Destroys current sessions with all vCenter Namespaces clusters.
>   version     Prints the version of the plugin.
> 
> Flags:
>   -h, --help                     help for kubectl-vsphere
>       --request-timeout string   Request timeout for HTTP client.
>   -v, --verbose int              Print verbose logging information.
> 
> Use "kubectl-vsphere [command] --help" for more information about a command.
> ```

Now `login on the Supervisor Cluster`:

```bash 
kubectl vsphere login --vsphere-username <your_vsphere_username> --server <your_supervisor_ip> --insecure-skip-tls-verify
```

For the next step we have to insert our personal `Password` for the vSphere  Environment:

> ```
> KUBECTL_VSPHERE_PASSWORD environment variable is not set. Please enter the password below
> Password:
> ```

We will see successfull login and the output will show us the contexts avaible on our configuration
> ```
> Logged in successfully.
> 
> You have access to the following contexts:
>    <your_supervisor_ip>
>    <vsphere_with_tanzu_namespace_> 
>      
>
> If the context you wish to use is not in this list, you may need to try
> logging in again later, or contact your cluster administrator.
> 
> To change context, use `kubectl config use-context <workload name>
> ```

Choose one of the previously seen contexts to build our TKGS cluster 

```bash
kubectl config use-context  <vsphere_with_tanzu_namespace_>
```
This will be the result 

> ```
> Switched to context "<vsphere_with_tanzu_namespace_>".
> ```

Open a file named `tkgs-cluster.yaml`

```
vi tkgs-cluster.yaml
```

Paste and customize the following code on your editor :

<!--codeinclude-->
[tkgs-cluster.yaml](/images/Tap-static/tkgs-cluster.yaml)
<!--/codeinclude-->

[Download `tkgs-cluster.yaml` here](/images/Tap-static/tkgs-cluster.yaml).


Where:
* `<vsphere_with_tanzu_namespace_>` This will be the vSphere with Tanzu namespace which you select as context for the previous command.

* `<your_VM_class>` This parameter will find resolution inside the settings of your vSphere with Tanzu Namespace from the workload management interface of the vCenter. Ensure that the same VM class will be avaible from th vCenter console.

* `<your_storage_class>`This parameter, like the previous one,  will find resolution inside the settings of your vSphere with Tanzu Namespace from the workload management interface of the vCenter. Ensure that the same VM Storage class will be avaible from th vCenter console.

* `name` Remember to change the name for each component for future deployment of  Tanzu Kubernetes cluster

![image](/images/Tap-static/vm_class.png)

![image](/images/Tap-static/Storage_class.png)


At the  end your configuration will look like this below:

![image](/images/Tap-static/tkgs-yaml.png)

Save your file and apply the manifest to build your TKGS cluster

```
kubectl apply -f tkgs-cluster.yaml
```

> ```
> tanzukubernetescluster.run.tanzu.vmware.com/tkgs-cluster-desotech-tap created
> ```

list all the avaible Tanzu KG Cluster:

```
kubectl get tkc 
```

This will be your Output:

![image](/images/Tap-static/tkc.png)

Wait 5-10 minutes until you see the cluster in `ready true`  state


## Install TAP Essentials

To change the context and see avaible the TKC previously created, customize and run this command below:

```bash
kubectl vsphere login --insecure-skip-tls-verify --server <your_supervisor_ip> --vsphere-username <your_vsphere_username> --tanzu-kubernetes-cluster-namespace <vsphere_with_tanzu_namespace_> --tanzu-kubernetes-cluster-name <tanzu_cluster_name>
```

For the next step we have to insert our personal `Password` for the vSphere  Environment:

> ```
> KUBECTL_VSPHERE_PASSWORD environment variable is not set. Please enter the password below
> Password:
> ```

We will see successfull login and the output will show us the contexts avaible on our configuration
This time we have a **new context** avaible that is the new Tanzu kubernetes cluster , created before.

> ```
> Logged in successfully.
> 
> You have access to the following contexts:
>    <your_supervisor_ip>
>    <vsphere_with_tanzu_namespace_> 
>    <tanzu_cluster_name>  
>
> If the context you wish to use is not in this list, you may need to try
> logging in again later, or contact your cluster administrator.
> 
> To change context, use `kubectl config use-context <workload name>
> ```

Now To change context run :

```
kubectl config use-context <tanzu_cluster_name>
```
> ```
> Switched to context "<tanzu_cluster_name>".
> ```

Open a tab on your browser and sign in to [Tanzu Network](https://network.tanzu.vmware.com/).

![image](/images/Tap-static/tanzu-network.png)

Navigate to `Cluster Essentials for VMware Tanzu` on Tanzu Network.

Download `tanzu-cluster-essentials-darwin-amd64-1.0.0.tgz` (for OS X) or `tanzu-cluster-essentials-linux-amd64-1.0.0.tgz` (for Linux) 

![image](/images/Tap-static/tanzu-cluster-essentials.png)

unpack the TAR file into tanzu-cluster-essentials directory:

```
mkdir $HOME/tanzu-cluster-essentials
tar -xvf tanzu-cluster-essentials-darwin-amd64-1.0.0.tgz -C $HOME/tanzu-cluster-essentials
```

Configure and run install.sh, which installs kapp-controller and secretgen-controller on your cluster:

```
export INSTALL_BUNDLE=registry.tanzu.vmware.com/tanzu-cluster-essentials/cluster-essentials-bundle@sha256:82dfaf70656b54dcba0d4def85ccae1578ff27054e7533d08320244af7fb0343
export INSTALL_REGISTRY_HOSTNAME=registry.tanzu.vmware.com
export INSTALL_REGISTRY_USERNAME=<YOUR-TANZU-NETWORK-USER>
export INSTALL_REGISTRY_PASSWORD=<YOUR-TANZU-NETWORK-PASSWORD>
```
```
cd $HOME/tanzu-cluster-essentials
./install.sh
```
> Where TANZU-NET-USER and TANZU-NET-PASSWORD are credentials for Tanzu Network with which you have accepted the EULA previously .

Install the kapp CLI onto your $PATH:
```
sudo cp $HOME/tanzu-cluster-essentials/kapp /usr/local/bin/kapp
```
Return to the home directory 

```
cd 
```

## Install the Tanzu CLI

If applicable is reccomended to to perform a clean installation of Tanzu CLI

Create a directory named tanzu by running:
```
mkdir $HOME/tanzu
```

Sign in to [Tanzu Network](https://network.tanzu.vmware.com/).

![image](/images/Tap-static/tanzu-network.png)

Navigate to `Tanzu Application Platform` on Tanzu Network.

Click the `tanzu-cli-v0.10.0` folder.

![image](/images/Tap-static/tanzu-cli.png)

Download `tanzu-framework-bundle-mac`  (for OS X) or `tanzu-framework-bundle-linux` (for Linux) 

![image](/images/Tap-static/tanzu-framework-bundle.png)

unpack the TAR file into the tanzu directory by running:

```
tar -xvf tanzu-framework-linux-amd64.tar -C $HOME/tanzu
```

Set env var TANZU_CLI_NO_INIT to true to assure the local downloaded versions of the CLI core and plug-ins are installed:

```
export TANZU_CLI_NO_INIT=true
```

Move to the Tanzu folder

```
cd $HOME/tanzu
```
Install the CLI core by running:

```
sudo install cli/core/v0.10.0/tanzu-core-linux_amd64 /usr/local/bin/tanzu
```

Confirm installation of the CLI core by running:
```
tanzu version
```

Expected output: 

> ```
> version: v0.10.0
> buildDate: 2021-11-03
> sha: fd96bebe
> ```

## Install Tanzu CLI Plug-ins


If it hasn’t been done already, set env var TANZU_CLI_NO_INIT to true to assure the locally downloaded plug-ins are installed
```
export TANZU_CLI_NO_INIT=true
```

Move to your tanzu directory: Install the local versions of the plug-ins you downloaded by running:
```
cd $HOME/tanzu
```
Install the local versions of the plug-ins you downloaded by running:

```
tanzu plugin install --local cli all
```

Check the plug-in installation status by running:
```
tanzu plugin list
```

The Output will be:

> ```
>   NAME                LATEST VERSION  DESCRIPTION                                                        REPOSITORY  VERSION  STATUS
>   accelerator                         Manage accelerators in a Kubernetes cluster                                    v1.0.0   installed
>   apps                                Applications on Kubernetes                                                     v0.4.0   installed
>   cluster             v0.16.0         Kubernetes cluster operations                                      core        v0.10.0  upgrade available
>   kubernetes-release  v0.16.0         Kubernetes release operations                                      core        v0.10.0  upgrade available
>   login               v0.16.0         Login to the platform                                              core        v0.10.0  upgrade available
>   management-cluster  v0.16.0         Kubernetes management cluster operations                           core        v0.10.0  upgrade available
>   package             v0.16.0         Tanzu package management                                           core        v0.10.0  upgrade available
>   pinniped-auth       v0.16.0         Pinniped authentication operations (usually not directly invoked)  core        v0.10.0  upgrade available
>   secret              v0.16.0         Tanzu secret management                                            core        v0.10.0  upgrade available
>   services                            Discover Service Types and manage Service Instances (ALPHA)                    v0.1.1   installed
> ```

Tanzu Application Platform requires cluster-admin privileges. Running commands associated with the additional plug-ins can have unintended side effects.

In my case I need to run this command to grant admin privilegs:
```
kubectl create clusterrolebinding default-tkg-admin-privileged-binding --clusterrole=psp:vmware-system-privileged --group=system:authenticated
```

Expect to see the following:

> ```
>clusterrolebinding.rbac.authorization.k8s.io/default-tkg-admin-privileged-binding created
> ```

---
Evangelista Tragni

Desotech srl