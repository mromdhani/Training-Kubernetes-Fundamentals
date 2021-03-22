# Lab 05- Exposing Services with Ingress
---
- [Step 1- Choosing the right service when connecting applications](#step-1--choosing-the-right-service-when-connecting-applications)
- [Step 2- Setting Up Traefic Ingres Controller](#step-2--setting-up-traefic-ingres-controller)
- [Step3 - Using the Traefic Ingres Controller](#step3---using-the-traefic-ingres-controller)
  - [Host-based Routing¶](#host-based-routing)
  - [Path-based Routing¶](#path-based-routing)

# Step 1- Choosing the right service when connecting applications

- Requirements :
   - We are building a Spring Boot Microservice that provides REST endpoint and uses a Relational MySQL database in its backend. The microservice and database should be loosely coupled.
   - We want access the REST endpoind from outside the cluster. The access to the database should be restricted to from within the cluster.
- Starter Resources
   - The YAML manifests for the REST endpoint and the MySQL database are provided in these starter files :  `unit5-01-microservice-mysql-starter.yaml` and `unit5-01-microservice-rest-endpoint-starter.yaml`. The specification of service types are missing !
- Complete the specification the service types. Perform the deployment and test it.  To test the REST endpoint, use its relative path `/products`.

- The solution, if you need it,  in given in folder `unit5-step1-solution`.        

# Step 2- Setting Up Traefic Ingres Controller

An ingress is really just a set of rules to pass to a controller that is listening for them. You can deploy a bunch of ingress rules, but nothing will happen unless you have a controller that can process them. A LoadBalancer service could listen for ingress rules, if it is configured to do so.

Ingress sits between the public network (Internet) and the Kubernetes services that publicly expose our Api's implementation. Ingress is capable to provide Load Balancing, SSL termination, and name-based virtual hosting.
Ingress capabilities allows to securely expose multiple API's or Applications from a single domain name.

To set up an ingress, we need to configure a **Ingress Controller** is simply a pod that is configured to interpret ingress rules. The most popular ingress controllers supported by Kubernetes are nginx and [Traefik](https://docs.traefik.io/providers/kubernetes-ingress/), ...

- Install the[ Traefic Ingress Controller](https://doc.traefik.io/traefik/v1.7/user-guide/kubernetes/) (On Minikube)
  - Prerequisites : Setup ingress as an add-on. It can be enabled by the following command:

```shell
minikube addons enable ingress
```  
  - The, you will need to authorize Traefik to use the Kubernetes API.
```shell
kubectl apply -f https://raw.githubusercontent.com/traefik/traefik/v1.7/examples/k8s/traefik-rbac.yaml

```  
  - Deploy Traefik using a Deployment or DaemonSet. Here we are deplpoying it as Deployment:
```shell
kubectl apply -f https://raw.githubusercontent.com/traefik/traefik/v1.7/examples/k8s/traefik-deployment.yaml
```
  - Check the Pods: 
    ```shell
    kubectl --namespace=kube-system get pods
    ```

# Step3 - Using the Traefic Ingres Controller

Submitting an Ingress to the Cluster¶

Lets start by creating a Service and an Ingress that will expose the Traefik Web UI.

```shell
kubectl apply -f https://raw.githubusercontent.com/traefik/traefik/v1.7/examples/k8s/ui.yaml
```
## Host-based Routing¶

Now let's setup an entry in our `/etc/hosts` file to route `traefik-ui.minikube` to our cluster.

In production you would want to set up real DNS entries. You can get the IP address of your minikube instance by running minikube ip:

```shell
echo "$(minikube ip) traefik-ui.minikube" | sudo tee -a /etc/hosts
```
We should now be able to visit `traefik-ui.minikube` in the browser and view the Traefik web UI.

## Path-based Routing¶

Test sample application
  - Deploy test application:
    ```shell
    kubectl apply -f unit5-02-ingress-apple.yaml
    kubectl apply -f unit5-02-ingress-banana.yaml
    kubectl apply -f unit5-02-ingress-main.yaml
    ```
  - Test sample application
    ```shell
    $ curl -kL http://localhost/apple
    apple
    $ curl -kL http://localhost/banana
    banana
    ```

