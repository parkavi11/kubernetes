## Sample7: Mount secret volumes to deployment 

- This sample runs simple ballerina hello world service with secret volume mounts.
- K8S secret are intended to hold sensitive information, such as passwords, OAuth tokens, and ssh keys.
- Putting this information in a secret is safer and more flexible than putting it verbatim in a pod definition or in a
  docker image.
- @kubernetes:Secret{} annotation will create k8s secrets. See [hello_world_secret_mount_k8s.bal](./hello_world_secret_mount_k8s.bal)  
- Following files will be generated from this sample.
    ``` 
    $> docker image
    hello_world_secret_mount_k8s:latest
    
    $> tree
    ├── README.md
    ├── hello_world_secret_mount_k8s.bal
    ├── hello_world_secret_mount_k8s.jar
    ├── security
        ├── ballerinaKeystore.p12
        └── ballerinaTruststore.p12
    ├── docker
        └── Dockerfile
    ├── kubernetes
    │   ├── hello-world-secret-mount-k8s-deployment
    │   │   ├── Chart.yaml
    │   │   └── templates
    │   │       └── hello_world_secret_mount_k8s.yaml
    │   └── hello_world_secret_mount_k8s.yaml
    └── secrets
        ├── MySecret1.txt
        ├── MySecret2.txt
        └── MySecret3.txt

    ```
### How to run:

1. Compile the  hello_world_secret_mount_k8s.bal file. Command to deploy kubernetes artifacts will be printed on build success.
```bash
$> ballerina build hello_world_secret_mount_k8s.bal
Compiling source
        hello_world_secret_mount_k8s.bal

Generating executables
        hello_world_secret_mount_k8s.jar

Generating artifacts...

        @kubernetes:Service                      - complete 1/1
        @kubernetes:Ingress                      - complete 1/1
        @kubernetes:Secret                       - complete 3/3
        @kubernetes:Deployment                   - complete 1/1
        @kubernetes:Docker                       - complete 2/2 
        @kubernetes:Helm                         - complete 1/1

        Run the following command to deploy the Kubernetes artifacts: 
        kubectl apply -f /Users/parkavi/Documents/Parkavi/BalKube/kubernetes/samples/sample7/kubernetes

        Run the following command to install the application using Helm: 
        helm install --name hello-world-secret-mount-k8s-deployment /Users/parkavi/Documents/Parkavi/BalKube/kubernetes/samples/sample7/kubernetes/hello-world-secret-mount-k8s-deployment```
```

2. hello_world_secret_mount_k8s.jar, Dockerfile, docker image and kubernetes artifacts will be generated: 
```bash
$> tree
.
├── README.md
├── hello_world_secret_mount_k8s.bal
├── hello_world_secret_mount_k8s.jar
├── security
    ├── ballerinaKeystore.p12
    └── ballerinaTruststore.p12
├── docker
    └── Dockerfile
├── kubernetes
│   ├── hello-world-secret-mount-k8s-deployment
│   │   ├── Chart.yaml
│   │   └── templates
│   │       └── hello_world_secret_mount_k8s.yaml
│   └── hello_world_secret_mount_k8s.yaml
└── secrets
    ├── MySecret1.txt
    ├── MySecret2.txt
    └── MySecret3.txt

```

3. Verify the docker image is created:
```bash
$> docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZE
hello_world_secret_mount_k8s   latest              53559c0cd4f4        55 seconds ago      194MB

```

4. Run kubectl command to deploy artifacts (Use the command printed on screen in step 1):
```bash
$> kubectl apply -f /Users/parkavi/Documents/Parkavi/BalKube/kubernetes/samples/sample7/kubernetes
service/helloworldep-svc created
ingress.extensions/helloworldep-ingress created
secret/helloworldep-secure-socket created
secret/private created
secret/public created
deployment.apps/hello-world-secret-mount-k8s-deployment created
```

5. Verify kubernetes deployment,service,secrets and ingress is deployed:
```bash
$> kubectl get pods
NAME                                                       READY     STATUS    RESTARTS   AGE
hello-world-secret-mount-k8s-deployment-7fb6b6f7f8-blwm2   1/1       Running   0          34s

$> kubectl get svc
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
helloworldep-svc   ClusterIP    10.105.238.19   <none>        9090/TCP         47s

$> kubectl get ingress
NAME                 HOSTS     ADDRESS   PORTS     AGE
helloworld-ingress   abc.com             80, 443   1m

$> kubectl get secrets
NAME                    TYPE                                 DATA      AGE
private                 Opaque                                1         1m
public                  Opaque                                2         1m
helloworldep-keystore   Opaque                                1         1m

```

6. Access the hello world service with curl command:

- **Using ingress:**
Add /etc/hosts entry to match hostname. 
_(127.0.0.1 is only applicable to docker for mac users. Other users should map the hostname with correct ip address 
from `kubectl get ingress` command.)_

```bash
$> curl https://abc.com/helloWorld/secret1 -k
Secret1 resource: Secret1

$> curl https://abc.com/helloWorld/secret2 -k
Secret2 resource: Secret2

$> curl https://abc.com/helloWorld/secret3 -k
Secret3 resource: Secret3
```

7. Undeploy sample:
```bash
$> kubectl delete -f /Users/hemikak/ballerina/dev/ballerinax/kubernetes/samples/sample7/kubernetes/
$> docker rmi hello_world_secret_mount_k8s

```
