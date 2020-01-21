## Sample14: Kubernetes Hello World with namespace

- This sample runs simple ballerina hello world service in ballerina namespace.
- Following files will be generated from this sample.
    ``` 
    $> docker image
    hello_world_k8s_namespace:latest
    
    $> tree
    ├── README.md
    ├── hello_world_k8s_namespace.bal
    ├── hello_world_k8s_namespace.jar
    ├── docker
        └── Dockerfile
    └── kubernetes
        ├── hello-world-k8s-namespace-deployment
        │   ├── Chart.yaml
        │   └── templates
        │       └── hello_world_k8s_namespace.yaml
        └── hello_world_k8s_namespace.yaml
    ```
### How to run:

1. Compile the  hello_world_k8s_namespace.bal file. Command to deploy kubernetes artifacts will be printed on build success.
```bash
$> ballerina build hello_world_k8s_namespace.bal
Compiling source
        hello_world_k8s_namespace.bal

Generating executables
        hello_world_k8s_namespace.jar

Generating artifacts...

        @kubernetes:Service                      - complete 1/1
        @kubernetes:Ingress                      - complete 1/1
        @kubernetes:Deployment                   - complete 1/1
        @kubernetes:Docker                       - complete 2/2 
        @kubernetes:Helm                         - complete 1/1

        Run the following command to deploy the Kubernetes artifacts: 
        kubectl apply -f /Users/parkavi/Documents/Parkavi/BalKube/kubernetes/samples/sample14/kubernetes

        Run the following command to install the application using Helm: 
        helm install --name hello-world-k8s-namespace-deployment /Users/parkavi/Documents/Parkavi/BalKube/kubernetes/samples/sample14/kubernetes/hello-world-k8s-namespace-deployment

```

2. hello_world_k8s_namespace.jar, Dockerfile, docker image and kubernetes artifacts will be generated: 
```bash
$> tree
.
├── README.md
├── hello_world_k8s_namespace.bal
├── hello_world_k8s_namespace.jar
├── docker
    └── Dockerfile
└── kubernetes
    ├── hello-world-k8s-namespace-deployment
    │   ├── Chart.yaml
    │   └── templates
    │       └── hello_world_k8s_namespace.yaml
    └── hello_world_k8s_namespace.yaml
```

3. Verify the docker image is created:
```bash
$> docker images
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
hello_world_k8s_namespace      latest              df83ae43f69b        2 minutes ago        102MB

```

4. Create a namespace as `ballerina` in Kubernetes.
```bash
$> kubectl create namespace ballerina
namespace "ballerina" created
```

5. Run kubectl command to deploy artifacts (Use the command printed on screen in step 1):
```bash
$>  kubectl apply -f /Users/parkavi/Documents/Parkavi/BalKube/kubernetes/samples/sample14/kubernetes
service/hello created
ingress.extensions/helloep-ingress created
deployment.apps/hello-world-k8s-namespace-deployment created
```

6. Verify kubernetes deployment,service and ingress is running:
```bash
$> kubectl get pods -n ballerina
NAME                                                READY     STATUS    RESTARTS   AGE
hello-world-k8s-namespace-deployment-54768647ff-m64v9   1/1       Running   0          4s


$> kubectl get svc -n ballerina
NAME      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
hello     ClusterIP   10.102.82.244   <none>        9090/TCP   16s

$> kubectl get ingress -n ballerina
NAME              HOSTS     ADDRESS   PORTS     AGE
helloep-ingress   abc.com             80        21s
```

7. Access the hello world service with curl command:

- **Using ingress**

Add /etc/hosts entry to match hostname.
_(127.0.0.1 is only applicable to docker for mac users. Other users should map the hostname with correct ip address 
from `kubectl get ingress` command.)_
 ```
 127.0.0.1 abc.com
 ```
Use curl command with hostname to access the service.
```bash
$> curl http://abc.com/helloWorld/sayHello
Hello, World from service helloWorld !
```

8. Undeploy sample:
```bash
$> kubectl delete -f ./kubernetes/
service "hello" deleted
ingress.extensions "helloep-ingress" deleted
deployment.apps "hello-world-k8s-namespace-deployment" deleted
$> kubectl delete namespace ballerina
namespace "ballerina" deleted
$> docker rmi hello_world_k8s_namespace

```