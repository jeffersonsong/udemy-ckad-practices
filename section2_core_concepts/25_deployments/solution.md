1. How many PODs exist on the system?

In the current(default) namespace.

0

```shell
k get po
No resources found in default namespace.
```

2. How many ReplicaSets exist on the system?

In the current(default) namespace.

0

```shell
k get rs
No resources found in default namespace.
```

3. How many Deployments exist on the system?

In the current(default) namespace.

0

```shell
k get deploy
No resources found in default namespace.
```

4. How many Deployments exist on the system now?

We just created a Deployment! Check again!

1

```shell
k get deploy
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
frontend-deployment   0/4     4            0           8s
```

5. How many ReplicaSets exist on the system now?

1

```shell
k get rs
NAME                             DESIRED   CURRENT   READY   AGE
frontend-deployment-6469748456   4         4         0       30s
```

6. How many PODs exist on the system now?

4

```shell
k get po
NAME                                   READY   STATUS             RESTARTS   AGE
frontend-deployment-6469748456-lfx8j   0/1     ImagePullBackOff   0          52s
frontend-deployment-6469748456-4kwb4   0/1     ImagePullBackOff   0          52s
frontend-deployment-6469748456-jlvcz   0/1     ImagePullBackOff   0          52s
frontend-deployment-6469748456-788w5   0/1     ErrImagePull       0          52s
```

7. Out of all the existing PODs, how many are ready?

0

8. What is the image used to create the pods in the new deployment?

busybox888

```shell
k describe pod frontend-deployment-6469748456-lfx8j | grep "Image:"
    Image:         busybox888
```

9. Why do you think the deployment is not ready?

The image BUSYBOX888 doesn't exist

```shell
k describe pod frontend-deployment-6469748456-lfx8j 

Events:
  Type     Reason     Age                 From               Message
  ----     ------     ----                ----               -------
  Normal   Pulling    47s (x4 over 2m8s)  kubelet            Pulling image "busybox888"
  Warning  Failed     47s (x4 over 2m8s)  kubelet            Failed to pull image "busybox888": failed to pull and unpack image "docker.io/library/busybox888:latest": failed to resolve reference "docker.io/library/busybox888:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
```

10.  Create a new Deployment using the deployment-definition-1.yaml file located at /root/.
There is an issue with the file, so try to fix it.

- Name: deployment-1

```shell
k apply -f deployment-definition-1.yaml 
Error from server (BadRequest): error when creating "deployment-definition-1.yaml": deployment in version "v1" cannot be handled as a Deployment: no kind "deployment" is registered for version "apps/v1" in scheme "k8s.io/apimachinery@v1.30.0-k3s1/pkg/runtime/scheme.go:100"

k apply -f deployment-definition-1.yaml 
deployment.apps/deployment-1 created

k get deploy
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
frontend-deployment   0/4     4            0           5m16s
deployment-1          2/2     2            2           12s
```

11. Create a new Deployment with the below attributes using your own deployment definition file.
- Name: httpd-frontend;
- Replicas: 3;
- Image: httpd:2.4-alpine

```shell
k create deployment httpd-frontend --image=httpd:2.4-alpine --replicas=3 --dry-run=client -o yaml > httpd-frontend-deployment.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: httpd-frontend
  name: httpd-frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: httpd-frontend
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: httpd-frontend
    spec:
      containers:
      - image: httpd:2.4-alpine
        name: httpd
        resources: {}
status: {}
```

```shell
k apply -f httpd-frontend-deployment.yaml 
deployment.apps/httpd-frontend created
```
