# Lab 01- Review of Kubernetes Basic Concepts
---
- [Step 1 - Setting Up  a Kubernetes Cluster](#step-1---setting-up--a-kubernetes-cluster)
  - [1.1 -  Option 1- Installing Kubernetes Locally Using Docker Desktop for Windows](#11----option-1--installing-kubernetes-locally-using-docker-desktop-for-windows)
  - [1.2 - Option 2- Installing Kubernetes Locally Using Minikube](#12---option-2--installing-kubernetes-locally-using-minikube)
    - [_Install Minikube_](#install-minikube)
    - [_Verify the installation of Minikube_](#verify-the-installation-of-minikube)
  - [1.3 - Option 3- Building a  multi-nodes cluster using VirtualBox and Vagrant](#13---option-3--building-a--multi-nodes-cluster-using-virtualbox-and-vagrant)
    - [Prerequisites](#prerequisites)
    - [Bringing Up The cluster](#bringing-up-the-cluster)
    - [Connect to the Cluster Master and check that it works](#connect-to-the-cluster-master-and-check-that-it-works)
    - [Add the multi-node cluster in the list of cluster contexts](#add-the-multi-node-cluster-in-the-list-of-cluster-contexts)
    - [Useful Vagrant commands](#useful-vagrant-commands)
- [Step 4 - Exploring your Kubernetes Cluster](#step-4---exploring-your-kubernetes-cluster)
- [Step 5: Deploying your first application](#step-5-deploying-your-first-application)
    - [Create a Deployment](#create-a-deployment)
    - [Create a Service](#create-a-service)
    - [Scale the application](#scale-the-application)
    - [Doing it declaratively (Using YAML manifests)](#doing-it-declaratively-using-yaml-manifests)

# Step 1 - Setting Up  a Kubernetes Cluster

## 1.1 -  Option 1- Installing Kubernetes Locally Using Docker Desktop for Windows

Since a while, Docker Desktop also comes with built-in support for Kubernetes, meaning that you don't have to install anything special to spin a Kubernetes cluster on your machine.
Once Docker Desktop is up & running, just right click on the icon in the systray and choose **Settings**. You will see a new section called **Kubernetes**. 

 <img src="images/kubernetes-install-docker-desktop.png" width="650px" height="400px"/>
 
Check the **Enable Kubernetes option**. Docker Desktop will start the process to setup Kubernetes on your machine. This takes few minutes. After the operation is completed, you should see in the lower left corner the message Kubernetes is running marked by a green dot. To make things easier for later, make sure to check also the option **Deploy Docker Stacks to Kubernetes by default**.

Docker Desktop for Windows installs a single node cluster along with the CLI tool **kubectl**.

To verify that Kubernetes is indeed working by opening a command prompt. Feel free to choose the one you like best, like PowerShell or the standard Windows prompt. Enter the following command:

```shell
kubectl cluster-info
```
The Kubernetes cluster is indeed up & running. 

To check the version of Kubernetes, enter the following command.
```shell
kubectl version --short
```

## 1.2 - Option 2- Installing Kubernetes Locally Using Minikube

Minikube provides a local single-node Kubernetes cluster inside a Virtual Machine (VM) on your laptop/desktop.
The installation instructions for each operation system is provided <https://kubernetes.io/docs/tasks/tools/install-minikube/>. The kubectl CLI tool is not integrated in Minikube, thus you should install it seperately.  

### _Install Minikube_ 
Minikube requires an Hypervisor. If you have already installed Docker Desktop for Windows, **Hyper-V** should be activated already. Othewise, follow these instructions to activate on your Windows (https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install). It is possible to use another hypervisor such as [VirtualBox](https://www.virtualbox.org/wiki/Downloads).

* **Install Minikube using Chocolatey**
The easiest way to install Minikube on Windows is using [Chocolatey](https://chocolatey.org/) (run as an administrator):

    ```shell
    choco install minikube
    ```
  After Minikube has finished installing, close the current CLI session and restart a new shell nession. Minikube should have been added to your path automatically.

### _Verify the installation of Minikube_ 
To confirm successful installation of both a hypervisor and Minikube, you can run the following command to start up a local Kubernetes cluster. This command should be run as administrator.

```shell
minikube start --vm-driver=hyperv   # You are supposed to use hyperv hypervisor
<driver_name> can be virtualbox, docker, kvm2, vmware, ... 
```

Once minikube start finishes, run the command below to check the status of the cluster:

```shell
minikube status
```
If your cluster is running, the output from minikube status should be similar to:

```shell
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```
After you have confirmed whether Minikube is working with your chosen hypervisor, you can continue to use Minikube or you can stop your cluster. To stop your cluster, run:

```shell
minikube stop
```
If you you need to clear minikube's local state:

```shell
minikube delete
```
Now you have two local clustors available on your laptop/desktop : the Docker Desktop Kubernetes Cluster and Minikube. You can list all the clusters using the following command:

```shell
kubectl config get-contexts
CURRENT   NAME                 CLUSTER          AUTHINFO         NAMESPACE
*         docker-desktop       docker-desktop   docker-desktop
          minikube             minikube         minikube
```

To switch from a cluster to another you can do it either using the Docker System Tray.

 <img src="images/context-switching.png" width="350px" height="300px"/>

Or using the kubectl CLI.

```shell
kubectl config use-context docker-desktop
```

## 1.3 - Option 3- Building a  multi-nodes cluster using VirtualBox and Vagrant

This construction of the vluster is based on a `Vagrantfile` for the provisioning of 3 nodes Kubernetes cluster (1 master and 2 worker nodes) using `VirtualBox` and `Ubuntu 16.04`.

### Prerequisites

You need the following installed to use this playground.

- `Vagrant`, version 2.2 or better.
- `VirtualBox`, tested with Version 6.0
- Internet access, this playground pulls Vagrant boxes from the Internet as well as installs Ubuntu application packages from the Internet.

### Bringing Up The cluster

To bring up the cluster, change into the directory containing the Vagrantfile `C:\Workspace-Training-Kubernetes\infra-multi-nodes-cluster` and `vagrant up` !

```bash
cd infra-multi-nodes-cluster
vagrant up
```

Vagrant will start three machines. Each machine will have a NAT-ed network
interface, through which it can access the Internet, and a `private-network`
interface in the subnet 192.168.205.0/24. The private network is used for
intra-cluster communication.

The machines created are:

| NAME | IP ADDRESS | ROLE |
| --- | --- | --- |
| 01-k8s-master | `192.168.205.10` | Cluster Master |
| 01-k8s-node-1 | `192.168.205.11` | Cluster Worker |
| 01-k8s-node-2 | `192.168.205.12` | Cluster Worker |

As the cluster brought up the cluster master (**01-k8s-master**) will perform a `kubeadm init` and the cluster workers will perform a `kubeadmin join`.

### Connect to the Cluster Master and check that it works

This can be done either using :
(i) the vagrant command 
```
vagrant ssh 01-k8s-master # Launch this command from within the folder where the Vagrantfile exists; the SSH keys are stored in that folder.
```
or (ii) through the VS Code Remote SSH extension:
   - Display the VM SSH keys and copy them in clipboard :
```
vagrant ssh-config 
```
   - Start VS Code, then Menu View -> Command Palette -> Remote-SSH Open Config file, choose .ssh,and paste the SSH Keys.
   -  Menu View -> Comand Palette -> Remote-SSH Connect to Host, choose 01-k8s-master.

- Check that the cluster works :
```
  kubectl cluster-info
  kubectl get nodes
```  

### Add the multi-node cluster in the list of cluster contexts

If you run the following config command on Windows, you will  see only the DockerDoesktop and the Minikube clusters.
```
kubectl config get-contexts
```
Indications:
- Kube config files reside in `~/.kube` and they are named `config`
- Take care to save a backup copy each time you want to operate on the config file
- To merge two config files use this hint:
```
KUBECONFIG=~/.kube/config:~/someotherconfig 
kubectl config view --flatten > merged_config
```

### Useful Vagrant commands

```bash
#Create the cluster or start the cluster after a host reboot
vagrant up

#Get the SSH Keys for the Vagrant Boxes
vagrant ssh-config

#Execute provision in all the vagrant boxes
vagrant provision

#Execute provision in the Kubernetes node 1
vagrant provision k8s-node-1

#Open an ssh connection to the Kubernetes master
vagrant ssh k8s-head

#Open an ssh connection to the Kubernetes node 1
vagrant ssh k8s-node-1

#Open an ssh connection to the Kubernetes node 2
vagrant ssh k8s-node-2

#Stop all Vagrant machines (use vagrant up to start)
vagrant halt

#Destroy the cluster
vagrant destroy -f
```


# Step 4 - Exploring your Kubernetes Cluster

In this step, we will explore various components of Kubernetes Cluster.

**Tasks**
- Explorer the Kubectl Context and Configuration using the following commands:

```shell
kubectl config view           # Show Merged kubeconfig settings.
kubectl config view -o jsonpath='{.users[*].name}'   # get a list of users
kubectl config get-contexts                          # display list of contexts 
kubectl config current-context                       # display the current-context
```
- Explore the Kubernetes Context using the following commands
```shell
kubectl cluster-info         # Display addresses of the master and services
kubectl cluster-info dump    # Dump current cluster state to stdout
```
- Explore the existing resources
```shell
kubectl get nodes            # List all cluster nodes
kubectl get namespaces       # List all the namespaces
kubectl get deployments      # List all the namespaces in the default namespace
kubectl get services         # List all the services in the default namespace
kubectl get pods             # List all the pods in the default namespace
kubectl get deployments --all-namespaces     # List all the namespaces in all namespaces
kubectl get all --all-namespaces     # List all the resources in all namespaces
```

# Step 5: Deploying your first application

### Create a Deployment

Deployments are the recommended way to manage the creation and scaling of Pods. A Kubernetes Pod is a group of one or more Containers, tied together for the purposes of administration and networking. The Pod in this step has only one Container. A Kubernetes Deployment checks on the health of your Pod and restarts the Pod's Container if it terminates. 

- Use the `kubectl create` command to create a Deployment that manages a Pod. The Pod runs a Container based on the provided Docker image.

```shell
kubectl create deployment hello-kubernetes --image=dockercloud/hello-world
```
- View the Deployment:

```shell
kubectl get deployments
```
- View the Pod:

```shell
kubectl get pods
```
We can easily see this if we try to terminate the pod with the following command:

```shell
kubectl delete pods hello-kubernetes-779db5df9f-227hk
pod "hello-kubernetes-779db5df9f-227hk" deleted
 ```
Now try again to see the list of pods. If you're fast enough, you should see something like this:

```shell
kubectl get pods
NAME                        READY   STATUS              RESTARTS   AGE
hello-kubernetes-cfd4bd475-8mv2x   0/1     ContainerCreating   0          3s
```
As you can see, Kubernetes is already creating a new pod. If we try again in a few seconds, we'll see our pod back in the Running status. Can you understand what just happened? The deployment has specified a desired state of a single instance of our webserver application always up & running. As soon as we have killed the pod, Kubernetes has realized that the desired state wasn't respected anymore and, as such, it has created a new one. Smart, isn't it?

### Create a Service
By default, the Pod is only accessible by its internal IP address within the Kubernetes cluster. To make the container accessible from outside the Kubernetes virtual network, you have to expose the Pod as a Kubernetes Service.

- Expose the Pod to the public internet using the kubectl expose command:

```shell
kubectl expose deployment hello-kubernetes --type=LoadBalancer --port=8080 --target-port=80
```
The `--type=LoadBalancer` flag indicates that you want to expose your Service outside of the cluster.

View the Service you just created:

```shell
kubectl get services
```
The output is similar to:
```shell
NAME               TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-kubernetes   LoadBalancer   10.98.98.14     localhost     8080:31099/TCP   7m
```
Since we have specified LoadBalancer as type, Kubernetes will automatically assign an external IP, other than an internal IP, to our application. Since Kubernetes is running on our own machine, the external IP will be the localhost. As such, we can open a browser and type <http://localhost:8080> to see the landing page of the web application displayed:

<img src="images/landing_page.png" width="580px" height="350px"/>

### Scale the application
What if we want to scale our application up? The nice part of Kubernetes is that, exactly like what happened when we have killed the pod, we don't have to manually take care of creating / deleting the pods. We just need to specify which is the new desired state, by updating the definition of the deployment. For example, let's say that now we want 5 instances of hello-kubernetes to be always up & running. We can use the following command:

```shell
kubectl scale deployments hello-kubernetes --replicas=5
```
Now let's see again the list of available pods:

```shell
kubectl get pods
```
### Doing it declaratively (Using YAML manifests)

- This is the manifest of the deployment :
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: dockercloud/hello-world
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 80
```
Apply the manifest using the command
```shell
kubectl apply -f my-hello-deployment.yaml
```
- This is the manifest of the service :

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
  namespace: default
spec:
  selector:
    app: my-app
  type: LoadBalancer
  ports:
  - name: ports
    port: 8081
    targetPort: 80
    protocol: TCP
    nodePort: 32000
```
Apply the manifest using the command
```shell
kubectl apply -f my-hello-service.yaml
```