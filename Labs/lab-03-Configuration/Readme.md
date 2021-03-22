# Lab 03- Configuration
---
- [Step 1 - ConfigMaps and Secrets](#step-1---configmaps-and-secrets)
    - [Creating ConfigMaps](#creating-configmaps)
    - [Accessing ConfigMaps](#accessing-configmaps)
    - [Creating Secrets](#creating-secrets)
    - [Configuring and Deploying MySQL](#configuring-and-deploying-mysql)
- [Step 2 - Managing resources for Containers and Pods](#step-2---managing-resources-for-containers-and-pods)

# Step 1 - ConfigMaps and Secrets
The 3rd factor (Configuration) of the [Twelve-Factor App principles](https://12factor.net/) states:

  > Configuration that varies between deployments should be stored in the environment.

This is where Kubernetes ConfigMaps and Secrets can help by supplying your deployment containers with the contextual and secretive information they require. Secrets and ConfigMaps behave similarly in Kubernetes, both in how they are created and because they can be exposed inside a container as mounted files or volumes or environment variables.

A core component of the Kubernetes management plane is **etcd**. Etcd is a high-available, distributed key/value store ideal for contextual environment settings and data. The ConfigMaps and Secrets are simply interfaces for managing this information in etcd.

In the following steps you will learn:

  - how to create configuration data in the form of ConfigMaps and Secrets,
  - how Pods make configuration accessible for applications in containers,
  - how secrets should remain secrets.

### Creating ConfigMaps
A ConfigMap is simple data associated with a unique key. They can be created and shared in the containers in the same ways as secrets. ConfigMaps are intended for non-sensitive data—configuration data—like config files and environment variables and are a great way to create customized running services from generic container images.
- **Create ConfigMaps from litteral values**
  
  You can write a `YAML` representation of the ConfigMap manually and load it into Kubernetes, or you can use the CLI  `kubectl create configmap` command to create it from the command line. 
  - **Create ConfigMap from CLI**
  
   The following example creates a ConfigMap using the CLI.
    ```shell
    kubectl create configmap mysql-config --from-literal=DB_NAME="ProductsDB" --from-literal=USER_NAME="kubernetes"
    ```
     Key/value configs are passed using the `--from-literal` option. It is possible to declare more than one `--from-literal` in order to pass multiple configuration entries in the same command. 
     
    - Check Kubernetes using `kubectl get configmap `after the create. 
    ```shell
      kubectl get configmap mysql-config 
    ```
    - To see the actual data, get it in YAML form.
    ```shell
      kubectl get configmap mysql-config -o yaml
    ```
    - Or, in description form
    ```shell
      kubectl describe configmap mysql-config 
    ```
    - Finally, to clean up delete the configmap.
    ```shell
      kubectl delete configmap mysql-config
    ```
  - **Create ConfigMap from YAML**
   A better way to define ConfigMaps is with a resource YAML file in this form.

   - Edit a yaml file named `unit3-01-configmaps.yaml` and initialize it as follows.
      ```yaml
      apiVersion: v1
      kind: ConfigMap
      metadata:
        name: mysql-config-yaml
        namespace: default
      data:
        DB_NAME: ConfluenceDB
        USER_NAME: kubernetes
        confluence.cnf: |-
          [mysqld]
            collation-server=utf8_bin
            default-storage-engine=INNODB 
            max_allowed_packet=256M           
      ``` 
    - Create the ConfigMap YAML resource using the following command
      ```shell
      kubectl create -f unit3-01-configmaps.yaml
      ```
    - Then, view it.

      ```shell
      kubectl describe configmap mysql-config-yaml
      ```
    The same ConfigMaps can also be explored in the Kubernetes Dashboard.
    - Clean up. Remove the configMap using `kubectl delete configmap command`.

      ```shell
      kubectl delete configmap mysql-config-yaml
      ```

- **Create ConfigMaps from files**
 
 You can also create ConfigMaps from from a file. To do this use the `--from-file` option of the `kubectl create configmap` command.  Your file should have a set of `key=value` pairs, one per line. If a value spans over more than one line, rely on backslash + end-of-line to escape the end of line. 
  - View the content of the given properties file which is located in th `config` folder `unit3-01-mysql.properties`
    ```yaml
    DB_NAME=ConfluenceDB
    USER_NAME=kubernetes 
    ```
  - Execute the following to create a ConfigMap from that configuration file:
    ```shell
    kubectl create configmap mysql-config-from-file --from-file=configs/unit3-01-mysql.properties
    ```
  - Now that we've created the ConfigMap, we can view it with:
    ```shell
    kubectl get configmaps
    ```
    We can get an even better view with:
    ```shell
    kubectl get configmap mysql-config-from-file -o yaml
    ```
  - Clean Up. Remove the ConfigMap using the following command:
    ```shell
    kubectl delete configmap mysql-config-from-file
    ```
  > **Note**: Kubernetes does not have know-how of how to ignore the **comments** and **blank lines** if we use `--from-file`. The `--from-env-file` option fixes that automatically. 
  It is a good practice to have your ConfigMaps in environment file format and use `--from-env-file` to create your ConfigMaps. For more information about the Docker environment file format, visit this [link](https://docs.docker.com/compose/env-file).  

- **Create ConfigMaps from directories**
 
  You can also create a ConfigMap from a directory. This is very similar to creating them from files. It can be very useful, as it allows you to separate out configuration into multiple directories, and then you can create an individual ConfigMp for each directory, and then quickly swap out configuration.
  - Let's create a new one from the `configs` directory.
  ```shell
  kubectl create configmap mysql-config-from-dir --from-file=configs
  ```
  At this point, Kubernetes will create a ConfigMap and populate it with all of the configuration from all files in the directory.
 - Let's view its content:
  ```shell
  kubectl get configmap mysql-config-from-dir  -o yaml
  ```
  - Clean Up. Remove the ConfigMap using the following command:
  ```shell
  kubectl delete configmap mysql-config-from-dir
  ```

### Accessing ConfigMaps
Once the configuration data is stored in ConfigMaps, the containers can access the data. Pods grant their containers access to the ConfigMaps through these three techniques:
   1. through the application command-line arguments,
   2. through the system environment variables accessible by the application,
   3. through a specific read-only file accessible by the application.

Let's explore these access techniques.

- **Accessing ConfigMaps through Command Line Arguments**
   
   This example shows how a Pod accesses configuration data from the ConfigMap by passing in the data through the command-line arguments when running the container. Upon startup, the application would reference these parameters from the program's command-line arguments.

  - Let's create a ConfigMap resource definition
   ```shell
    kubectl create configmap mysql-config-cli --from-env-file=configs/unit3-01-mysql.properties
   ```
  - Create the following Pod Definition. Name the Yaml file `unit3-01-configmaps-inject-cli.yaml` and set it as follows.
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: inject-config-via-cli
    spec:
      containers:
        - name: consuming-container
          image: k8s.gcr.io/busybox
          command: [ "/bin/sh", "-c", "echo $(PROPERTY_DB_NAME); echo $(PROPERTY_DB_USER); env" ]
          env:        
            - name: PROPERTY_DB_NAME
              valueFrom:
                configMapKeyRef:
                  name: mysql-config-cli
                  key: DB_NAME
            - name: PROPERTY_DB_USER
              valueFrom:
                configMapKeyRef:
                  name: mysql-config-cli
                  key: USER_NAME
          restartPolicy: Never
      ```  
  Run the following command to create the Pod
  ```shell
  kubectl apply -f unit3-01-configmaps-inject-cli.yaml
  ```
  - Inspect the log of the Pod to verify that the configuration has been applied.
  ```shell
  kubectl logs inject-config-via-cli
  ```
  - Clean Up. Remove the ConfigMap and the Pod.
  ```shell
  kubectl delete configmap mysql-config-cli 
  kubectl delete pod inject-config-via-cli
  ```
- **Accessing ConfigMaps Through Environment variables**
   
   This example shows how a Pod accesses configuration data from the ConfigMap by passing in the data as environmental parameters of the container. Upon startup, the application would reference these parameters as system environment variables.

  - Let's create a ConfigMap resource definition
   ```shell
    kubectl create configmap mysql-config-env --from-env-file=configs/unit3-01-mysql.properties
   ```
  - Create the following Pod Definition. Name the Yaml file `unit3-01-configmaps-inject-env.yaml` and set it as follows.
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: inject-config-via-env
    spec:
      containers:
        - name: consuming-container
          image: k8s.gcr.io/busybox      
          command: [ "/bin/sh", "-c", "env" ]
          envFrom:      
          - configMapRef:
              name: mysql-config-env
      restartPolicy: Never
    ```  
  Run the following command to create the Pod
  ```shell
  kubectl apply -f unit3-01-configmaps-inject-env.yaml
  ```
  - Inspect the log of the Pod to verify that the configuration has been applied.
  ```shell
  kubectl logs inject-config-via-env
  ```
  - Clean Up. Remove the ConfigMap and the Pod.
  ```shell
  kubectl delete configmap mysql-config-env 
  kubectl delete pod inject-config-via-env
  ```
- **Accessing ConfigMaps Through Volume Mounts**
   
   This example shows how a Pod accesses configuration data from the ConfigMap by reading from a file in a directory of the container. Upon startup, the application would reference these parameters by referencing the named files in the known directory.

  - Let's create a ConfigMap resource definition
   ```shell
    kubectl create configmap mysql-config-volume --from-env-file=configs/unit3-01-mysql.properties
   ```
  - Create the following Pod Definition. Name the Yaml file `unit3-01-configmaps-inject-volume.yaml` and set it as follows.
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: inject-config-via-volume
    spec:
      containers:
        - name: consuming-container
          image: k8s.gcr.io/busybox      
          command: [ "/bin/sh","-c","cat /etc/config/keys" ]
          volumeMounts:      
          - name: config-volume
            mountPath: /etc/config
      volumes:
        - name: config-volume
          configMap:
            name: mysql-config-volume
            items:
            - key: DB_NAME
              path: keys
      restartPolicy: Never
     ```  
  Run the following command to create the Pod
  ```shell
  kubectl apply -f unit3-01-configmaps-inject-volume.yaml
  ```
  - Inspect the log of the Pod to verify that the configuration has been applied.
  ```shell
  kubectl logs inject-config-via-volume
  ```
  - Clean Up. Remove the ConfigMap and the Pod.
  ```shell
  kubectl delete configmap mysql-config-volume 
  kubectl delete pod inject-config-via-volume
  ```

### Creating Secrets

Secrets are Kubernetes objects intended for storing a small amount of sensitive data. It is worth noting that Secrets are stored base64-encoded within Kubernetes, so they are not wildly secure. Make sure to have appropriate role-based access controls (RBAC) to protect access to Secrets. Even so, extremely sensitive Secrets data should probably be stored using something like HashiCorp Vault.
Both ConfigMaps and Secrets are stored in etcd, but the way you submit secrets is slightly different than ConfigMaps.

- **Create Secrets from CLI**
  
  To create secrets you can use the CLI  `kubectl create secret` command or you can write a `YAML` representation of the Secret manually and load it into Kubernetes.

 To create the Secret you should convert it to base64. To do this, there are many possiblities like the `base64` Unix Command, the `System.Convert` Utility in Pwershell, on you can simply use one of the free online base64 encoders.

    _Encoding/Decoding into base64 in Linux Shell_
    ```shell
    $ echo -n 'KubernetesRocks!' | base64    # For Encoding
      S3ViZXJuZXRlc1JvY2tzIQ==
    $ echo "TXlEYlBhc3N3MHJkCg==" | base64 --decode   # For Decoding
      KubernetesRocks!
    ```
    _Encoding/Decoding into base64 in Windows PowerShell_
    ```shell
    PS> [System.Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("KubernetesRocks!"))
        S3ViZXJuZXRlc1JvY2tzIQ==
    PS> [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String("S3ViZXJuZXRlc1JvY2tzIQ=="))
       KubernetesRocks!
    ```  
  - Let's create a secret using the encoded `base64` value of '`KubernetesRocks!`';
     ```shell
      kubectl create secret generic db-password --from-literal=password=S3ViZXJuZXRlc1JvY2tzIQ==
     ```
  - Check Kubernetes using `kubectl get secret `after the create. 
      ```shell
        kubectl get secret db-password 
      ```
  - To see the actual data, get it in YAML form.
  ```shell
    kubectl get secret db-password -o yaml
  ```
  - To decode the secret in Bash Shell
  ```shell
    kubectl get secret db-password -o 'go-template={{index .data "password"}}' | base64 --decode
  ```    
  - Finally, to clean up delete the secret.
  ```shell
    kubectl delete secret db-password
  ```
    > Note : Never confuse encoding with encryption, as they are two very different concepts. Values encoded are not encrypted. The encoding is to allow a wider variety of values for secrets. You can easily decode the text with a simple base64 command to reveal the original password text.

- **Create Secrets from YAML**
  
 A better way to define Secrets is with a resource YAML file in this form.
 - Create the following Secret YAML definition file. Name it `unit3-01-secrets.yaml` and set it as follows:
    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: mysql-secrets
    type: Opaque
    data:
      user-password: a3ViZXJuZXRlcw==       #kubernetes
      root-password: S3ViZXJuZXRlc1JvY2tzIQ==   # KubernetesRocks!
    ```
  Run the following command to create the Secret resource.
  ```shell
  kubectl apply -f unit3-01-secrets.yaml
  ```
 - Check Kubernetes using `kubectl get secret `after the create. 
    ```shell
      kubectl get secret mysql-secrets 
    ```
 - To decode the secret in Bash Shell
    ```shell
      kubectl get secret mysql-secrets -o 'go-template={{index .data "password"}}' | base64 --decode
    ```    
 - Finally, to clean up delete the secret.
  ```shell
    kubectl delete secret mysql-secrets
  ```
   **Hint** :  When first creating the YAML file you can skip using the base64 command and instead use the kubectl `--dry-run` feature which will generate the YAML file for you with the encoding.
    ```
    kubectl create secret generic db-password --from-literal=password=MyDbPassw0rd --dry-run -o yaml > my-secret.yaml
    ```
    If you view the content of `my-secret.yaml` you will see the base64 encoded value of the password.

### Configuring and Deploying MySQL

In order to deploy MySQL, you will need to mount the Secrets as environment variables and the ConfigMap as a file. First, though, you need to write a Deployment for MySQL so that you have something to work with. Create a file named `unit3-01-mysql-deployment-starter.yaml` with the following content.
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: mysql-deployment
    labels:
      app: mysql  
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: mysql
    template:
      metadata:
        labels:
          app: mysql
      spec:
        containers:
        - name: mysql
          image: quay.io/mromdhani/mysql:8.0
          ports:
          - containerPort: 3306
            protocol: TCP
          volumeMounts:
          - mountPath: /var/lib/mysql
            name: mysql-volume
        volumes:
        - emptyDir: {}
          name: mysql-volume
  ```
This is a bare-bones Kubernetes Deployment of the official MySQL 8.2 image from Docker Hub. If you'll try to deploy it, it fails and logs this message: `You need to specify one of MYSQL_ROOT_PASSWORD, MYSQL_ALLOW_EMPTY_PASSWORD and MYSQL_RANDOM_ROOT_PASSWORD"`.
Let' add  the Secrets and ConfigMap.

- **Deploy the ConfigMap and the Secret resources**

 Let's first deploy the ConfigMap and Secret Resources and Check that they have been deployed successfully.
  - Deploy the ConfigMap and Verify its deployment.
    ```shell
    kubectl create -f unit3-01-configmaps.yaml
    kubectl get configmap mysql-config
    ```
  - Deploy the Secrets and Verify its deployment.
    ```shell
    kubectl apply -f unit3-01-secrets.yaml
    kubectl get secret mysql-secrets
    ```
- **Add the Secrets to the Deployment as environment variables** 
   
   We have two Secrets that need to be added to the Deployment:
   - The MYSQL_ROOT_PASSWORD (The root password, to add as an environment variable)
   - The MYSQL_PASSWORD (The default user password, to add as an environment variable)
  Let's specify the Secret and the key you want by adding an **env list/array** to **the container spec** in the Deployment and setting the environment variable value to the value of the key in your Secret. In this case, the list contains two entries for for the passwords.
    ```yaml
    env:
      - name: MYSQL_ROOT_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysql-secrets
            key: root-password
      - name: MYSQL_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysql-secrets
            key: user-password
    ```
- **Add the ConfigMaps to the Deployment** 
   
  We have to add these configutra:
   - The MYSQL_DATABASE (The default database to create, to add as an environment varaiable)
   - The MYSQL_USER (The default user to create, to add as an environment variable)
   - The cnf configuration entry (To mount as a volume under `/etc/mysql/conf.d/` within the container)
   
     - Let's Specify the MYSQL_DATABASE and MYSQL_USER variables by adding them to the existing **env** list/array of **the container spec**.  
      ```yaml
      - name: MYSQL_DATABASE
        valueFrom:
          configMapKeyRef:
            name: mysql-config
            key: DB_NAME
      - name: MYSQL_USER
        valueFrom:
          configMapKeyRef:
            name: mysql-config
            key: USER_NAME
      ```
    - Let's add a new volume definition under **volumes** section. This volume will be initialized from the ConfigMap (the .cnf key) and will be used to define a new mount path under the  **mountPath** the existing **volumeMounts** section. This mountPath will point to `/etc/mysql/conf.d`.  
        ```yaml
           volumeMounts:
            - mountPath: /var/lib/mysql
              name: mysql-volume
            - mountPath: /etc/mysql/conf.d
              name: mysql-config-volume
        volumes:
          - emptyDir: {}
            name: mysql-volume
          - configMap:
              name: mysql-config
              items:
              - key: confluence.cnf
                path: confluence.cnf
            name: mysql-config-volume
        ```
- **Apply the deploymenet and check the configs and the secrets** 
  - Apply the deployment
    ```shell
    kubectl apply -f unit3-01-mysql-deployment-starter.yaml
    ```
  - Check that the deployment has succeeded and view the MySQLPod state
    ```shell
    kubectl get deploy
    kubectl get pods
    ```
  - Hop into the Pod and check that secrets and the configuration has been applied
    ```shell
    kubectl exec -it [PodName] sh  
    ```
    From within the Pod shell run
    ```shell
    # env
    # cat /etc/mysql/conf.d/confluence.cnf  
    ```
  - Clean Up. Remove the Secret, the ConfigMap and the Deployment
    ```shell
    kubectl delete secret mysql-secrets 
    kubectl delete cm  mysql-config
    kubectl delete deploy mysql-deployment 
    ```

# Step 2 - Managing resources for Containers and Pods
By default, containers run with unbounded compute resources on a Kubernetes cluster. With resource quotas, cluster administrators can restrict resource consumption and creation on a namespace basis.  

As a cluster administrator, you might want to impose restrictions on resources that Pods can use. When you specify a Pod, you can optionally specify how much of each resource a Container needs. The most common resources to specify are CPU and memory (RAM). These resources play a critical role in how the scheduler allocate pods to nodes.
Kubernetes uses the requests & limits structure to control resources such as CPU and memory.
  - **Requests** are what the container is guaranteed to get. For example, If a container requests a resource, Kubernetes will only schedule it on a node that can give it that resource.
  - **Limits**, on the other hand, is the resource threshold a container never exceed. The container is only allowed to go up to the limit, and then it is restricted.

Let's specify resources for the ngix container. For example, we specify a request of `0.25 cpu` and `64MiB` (64* 2 exp(20) bytes) of memory and a limit of `0.5 cpu` and `256MiB` of memory.

- Create a Deployment by editing a new YAML file. Name it `unit3-04-limit-resources.yaml`. Initialize it with the following content :

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-app-limits
    labels:
      app: nginx-app-limits
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: nginx-app-limits
    template:
      metadata:
        labels:
          app : nginx-app-limits
      spec:
        containers:
          - name: nginx
            image: nginx
            resources:
              requests: # Initial requests
                cpu: "250m" # Equivalent to 0.25 CPU (Quater CPU)
                memory: "64Mi" # Equivalent to 64 * 2 of power(20)             
              limits:  # Hard limits
                cpu: "500m"
                memory: "256Mi"              
  ```
- Check that these resource limits are taken into account. Get the list of pods using `kubectl get pods`. Then, get the detailed descrition of one of the pods using the `kubectl describe pod [pod_name]`.  
- Clean Up. Delete the Deployment using the `kubectl delete deployment nginx-app-limits`
- Create a **LimitRange** and a Pod
There is a concern that one Pod or Container could monopolize all available resources. A LimitRange is a policy to constrain resource allocations (to Pods or Containers) in a namespace.
  - Create a namespace called `constraints-mem-example`
```shell
  kubectl create namespace constraints-mem-example
```
  - Create a LimitRange specification in a file named `unit3-03-limit-range.yaml`.
    ```yaml
    apiVersion: v1
    kind: LimitRange
    metadata:
      name: mem-min-max
    spec:
      limits:
      - max:
          memory: 1Gi
        min:
          memory: 500Mi
        type: Container
    ```
  - Apply the LimitRange to the namespace `constraints-mem-example`:
    ```shell
      kubectl apply -f unit3-03-limit-range.yaml --namespace=constraints-mem-example  
    ```
  - View detailed information about the LimitRange:
    ```
      kubectl get limitrange mem-min-max --namespace=constraints-mem-example --output=yaml
    ```
- Attempt to create a Pod that exceeds the maximum memory constraint
  - Create a new Yaml manifest for a Pod that has one Container. The Container specifies a memory request of `800 MiB` and a memory limit of `1.5 GiB`.  Named it `unit3-04-pod-exceeding-range.yaml` and initialize it as follows:
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: constraints-mem-exceeding
    spec:
      containers:
      - name: constraints-mem-exceeding
        image: nginx
        resources:
          limits:
            memory: "1.5Gi"
          requests:
            memory: "800Mi"
    ```
  - Attempt to create the Pod:  
  ```shell
  kubectl apply -f unit3-04-pod-exceeding-range.yaml --namespace=constraints-mem-example
  ```
  The output shows that the Pod does not get created, because the Container specifies a memory limit that is too large:
  ```shell
  Error from server (Forbidden): error when creating "unit3-04-pod-exceeding-range.yaml": pods "constraints-mem-exceeding" is forbidden: maximum memory usage per Container is 1Gi, 
  but limit is 1536Mi
  ```
- Clean Up. Delete the the Pod, The LimiRange, and the NameSpace objects.
  ```
  kubectl delete pod constraints-mem-exceeding  --namespace=constraints-mem-example
  kubectl delete limitrange mem-min-max --namespace=constraints-mem-example
  kubectl delete ns constraints-mem-example
  ```

