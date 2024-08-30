1. In this lab, you will get hands-on practice with creating Kubernetes objects imperatively.

All the questions in this lab can be done imperatively. However, for some questions, you may need to first create the YAML file using imperative methods. You can then modify the YAML according to the need and create the object using kubectl apply -f command.

2. Deploy a pod named nginx-pod using the nginx:alpine image.
Use imperative commands only.
- Name: nginx-pod
- Image: nginx:alpine

```shell
k run nginx-pod --image=nginx:alpine
pod/nginx-pod created
```

3. Deploy a redis pod using the redis:alpine image with the labels set to tier=db.

Either use imperative commands to create the pod with the labels. Or else use imperative commands to generate the pod definition file, then add the labels before creating the pod using the file.

- Pod Name: redis
- Image: redis:alpine
- Labels: tier=db

```shell
k run -h

k run redis --image=redis:alpine --labels=tier=db
pod/redis created
```

4. Create a service redis-service to expose the redis application within the cluster on port 6379.

Use imperative commands.

- Service: redis-service
- Port: 6379
- Type: ClusterIP

```shell
k expose -h

kubectl expose pod redis --port=6379 --name=redis-service --type=ClusterIP
service/redis-service exposed
```

5. Create a deployment named webapp using the image kodekloud/webapp-color with 3 replicas.
Try to use imperative commands only. Do not create definition files.
- Name: webapp
- Image: kodekloud/webapp-color
- Replicas: 3

```shell
k create deploy webapp --image=kodekloud/webapp-color --replicas=3 
deployment.apps/webapp created
```

6. Create a new pod called custom-nginx using the nginx image and run it on container port 8080.
- Pod created correctly?

```shell
k run custom-nginx --image=nginx --port=8080 --dry-run=client -o yaml

k run custom-nginx --image=nginx --port=8080
pod/custom-nginx created
```

7. Create a new namespace called dev-ns.
Use imperative commands.

- Namespace created?

```shell
k create ns dev-ns
namespace/dev-ns created
```

8. Create a new deployment called redis-deploy in the dev-ns namespace with the redis image. It should have 2 replicas.
Use imperative commands.

- 'redis-deploy' created in the 'dev-ns' namespace?
- replicas: 2

```shell
k create deploy redis-deploy -n dev-ns --image=redis --replicas=2 
deployment.apps/redis-deploy created
```

9. Create a pod called httpd using the image httpd:alpine in the default namespace. Next, create a service of type ClusterIP by the same name (httpd). The target port for the service should be 80.

Try to do this with as few steps as possible.

- 'httpd' pod created with the correct image?
- 'httpd' service is of type 'ClusterIP'?
- 'httpd' service uses correct target port 80?
- 'httpd' service exposes the 'httpd' pod?

```shell
k run httpd --image=httpd:alpine
k expose -h

k expose pod httpd --port=80 --target-port=80 --type=ClusterIP --dry-run=client -o yaml

k expose pod httpd --port=80 --target-port=80 --type=ClusterIP
```
