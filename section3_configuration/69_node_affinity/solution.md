1. How many Labels exist on node node01?

5

```shell
controlplane ~ ➜  k describe node node01
Name:               node01
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=node01
                    kubernetes.io/os=linux
```

2. What is the value set to the label key beta.kubernetes.io/arch on node01?

amd64

3. Apply a label color=blue to node node01

```shell
k label node -h

controlplane ~ ➜  kubectl label nodes node01 color=blue
node/node01 labeled
```

4. Create a new deployment named blue with the nginx image and 3 replicas.

- Name: blue
- Replicas: 3
- Image: nginx

```shell
controlplane ~ ➜  k create deploy blue --image=nginx --replicas=3
deployment.apps/blue created
```

5. Which nodes can the pods for the blue deployment be placed on?

controlplane and node01

Make sure to check taints on both nodes!

```shell
controlplane ~ ➜  k get deploy -o wide
NAME   READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES   SELECTOR
blue   3/3     3            3           47s   nginx        nginx    app=blue

controlplane ~ ➜  k get po -o wide
NAME                   READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
blue-c4566ffc8-jxfdt   1/1     Running   0          56s   10.244.0.4   controlplane   <none>           <none>
blue-c4566ffc8-n2plh   1/1     Running   0          56s   10.244.1.2   node01         <none>           <none>
blue-c4566ffc8-nphvk   1/1     Running   0          56s   10.244.1.3   node01         <none>           <none>

k describe node node01
Taints:             <none>

k describe node controlplane
Taints:             <none>
```

6. Set Node Affinity to the deployment to place the pods on node01 only.

- Name: blue
- Replicas: 3
- Image: nginx
- NodeAffinity: requiredDuringSchedulingIgnoredDuringExecution
- Key: color
- value: blue

[Schedule a Pod using required node affinity](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/)

```shell
controlplane ~ ➜  k edit deploy blue
deployment.apps/blue edited
```

```yaml
spec:
  template:
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: color
                operator: In
                values:
                - blue
```

7. Which nodes are the pods placed on now?

```shell
controlplane ~ ➜  k get po -o wide
NAME                   READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
blue-884849b5b-kk5z7   1/1     Running   0          35s   10.244.1.5   node01   <none>           <none>
blue-884849b5b-lzffq   1/1     Running   0          40s   10.244.1.4   node01   <none>           <none>
blue-884849b5b-pdqls   1/1     Running   0          33s   10.244.1.6   node01   <none>           <none>
```

8. Create a new deployment named red with the nginx image and 2 replicas, and ensure it gets placed on the controlplane node only.
Use the label key - node-role.kubernetes.io/control-plane - which is already set on the controlplane node.

- Name: red
- Replicas: 2
- Image: nginx
- NodeAffinity: requiredDuringSchedulingIgnoredDuringExecution
- Key: node-role.kubernetes.io/control-plane
- Use the right operator

```shell
controlplane ~ ➜  k create deploy red --image=nginx --replicas=2 --dry-run=client -o yaml > red-deployment.yaml

vi red-deployment.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: red
  name: red
spec:
  replicas: 2
  selector:
    matchLabels:
      app: red
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: red
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/control-plane
                operator: Exists
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

```shell
controlplane ~ ➜  k apply -f red-deployment.yaml 
deployment.apps/red created
```