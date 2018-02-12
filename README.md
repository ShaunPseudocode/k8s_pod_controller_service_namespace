### __Creating a namespace in Kubernetes.__

Below is a JSON file that contains/describes all the contents needed to create a namespace in Kubernetes. The structure of my JSON file looks like this: 

```json
{
    "kind": "Namespace",
    "apiVersion": "v1",
    "metadata": {
        "name": "shift-namespace",
        "labels": {
            "name":"shift-namespace"
        }
    }
}

```
After building the JSON file, open your terminal and enter the following command:
```console
$ kubectl create -f ./shift.json
```

The above command will build a namespace. To verify this, open your Kubernetes dashboard and navigate to the 'namespace' tab. To open your dashboard, open your terminal and type:
```console
$ minikube dashboard
```

### __Creating a Replication Controller with a single Pod.__

To create a Replication Controller in Kubernetes, you need to build a YAML file with the following contents:

```yml
apiVersion: v1
kind: ReplicationController
metadata:
  name: redis
spec:
  replicas: 1
  selector:
    app: redis-netcore
  template:
    metadata:
      name: redis-netcore
      labels:
        app: redis-netcore
    spec:
      containers:
      - name: redis-netcore
        image: shaunmavunda/sample_net_core:latest
        ports:
        - containerPort: 6379
        imagePullPolicy: Always
```
Followed by the following command:
```console
$ kubectl create -f redis-repl-ctl.yaml
```
In my Docker, I have an image called 'shaunmavunda/sample_net_core:latest' which I pulled from Docker Hub and I give it a label in my YAML file 'redis-netcore'. Assuming you have redis-cli installed on your machine, to get the 'containerPort' open your terminal and type:

```console
$ redis-cli
``` 

### __Creating a Service in Kubernetes.__

To create a Service in Kubernetes we need to build a YAML file with the following contents:

```yml
kind: Service
apiVersion: v1
metadata:
  name: redis-service
spec:
  selector:
    app: redis-netcore
  ports:
  - protocol: TCP
    port: 6379
    targetPort: 6379
    nodePort: 30379
  type: NodePort
  sessionAffinity: ClientIP
  sessionAffinityConfig: 
    clientIP:
      timeoutSeconds: 10800
```
In my YAML file, I specified the Kubernetes ocject type which is a 'Service'. To find the supported API version to succesfully build your Service using a YAML file, type:

```yml
$ kubectl api-versions
```
<br/>The above command will list all the supported API versions.<br/>
<br/>The metadata field helps in uniquely identifying a Kubernetes object(the Service).<br/>In order for our Service connect to the Pod we created using a Replication Controller, we need specify which Pod we want to service in the selector field. In my Replication Controller YAML file, I named my Pod 'redis-netcore', which means my current YAML file will service/connect to that Pod.<br/> As mentioned before, assuming you have redis-cli installed on your machine, to get the 'port' and the 'targetPort' open your terminal and type:

```console
$ redis-cli
``` 


### __Creating a single pod and pushing it to a namespace we created.__

To create and add a pod or a deployment to a namespace, first you need to create a YAML file:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: shift-redis
  labels:
    name: shift-redis
spec:
  containers:
    - image: shaunmavunda/sample_net_core:latest
      name: redis-netcore
```
```console
$ kubectl create -f shift-pod.yaml --namespace=shift-namespace
``` 

In this example, I pulled a random image from Google Cloud and labeled it 'redis' in my YAML file. You can choose to pull any image of your choice that contains your application. It could be from Google Cloud or Docker Hub. One of the main disadvantages of creating a Pod without using a Replication Controller is, once a Pod dies, or gets accidentally deleted.

### __Tainting a node, so the pod(s) shouldn't start again.__

```console
$ kubectl taint nodes minikube key=value:NoExecute
```
To untaint a node, run the previous command with a dash at the end, for example:

```console
$ kubectl taint nodes minikube key=value:NoExecute-
```
Tainting a node means killing your Pod(s), and losing the container image in it. A Replication Controller is capable of restarting failed Pods but this doesn't necessarily mean the image/data in the Pod will be revived as well. 

### __Persistent Volumes with Docker.__
First we need to create directory in our host machine by typing the following in our terminal:

```console
$ mkdir /nginxlogs
```
The start the volume and attach to the container:
```console
$ docker run -d -v ~/nginxlogs:/var/log/nginx -p 5000:80 -i nginx
```
The host path and the path created in the container are separated by a colon. Changes made in the host file will also reflect in the docker container file system.

To enter the filesystem inside a container, run the command:
```console
$ docker exec -it <container name> bash
```
Navigate to the container directory by typing:
```console
root@c05bc69d219e:/# ls /var/log/nginx
```
To verify your binds, type:
```console
$ docker inspect <container name>
```
The above command will return a file in a JSON format:

```json
"Binds": [
            "/home/shaunm/nginxlogs:/var/log/nginx"
            ],
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {
                "80/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "5000"
                    }
                ]
            },
```







