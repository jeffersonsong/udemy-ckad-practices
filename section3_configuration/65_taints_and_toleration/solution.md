1. How many nodes exist on the system?

Including the controlplane node.

2

```shell
k get node -A 
NAME           STATUS   ROLES           AGE     VERSION
controlplane   Ready    control-plane   5m53s   v1.30.0
node01         Ready    <none>          5m5s    v1.30.0
```

2. Do any taints exist on node01 node?

No

```shell
k describe node node01 | less
Taints:             <none>
```

3. Create a taint on node01 with key of spray, value of mortein and effect of NoSchedule

- Key = spray
- Value = mortein
- Effect = NoSchedule

```shell
k taint node -h

kubectl taint nodes node01 spray=mortein:NoSchedule
node/node01 tainted
```

4. Create a new pod with the nginx image and pod name as mosquito.

Image name: nginx

```shell
k run mosquito --image=nginx
pod/mosquito created
```

5. What is the state of the POD?

Pending

```shell
k get po mosquito
NAME       READY   STATUS    RESTARTS   AGE
mosquito   0/1     Pending   0          26s
```

6. Why do you think the pod is in a pending state?

POD Mosquito cannot tolerate taint Mortein

```shell
k describe po mosquito
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  54s   default-scheduler  0/2 nodes are available: 1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }, 1 node(s) had untolerated taint {spray: mortein}. preemption: 0/2 nodes are available: 2 Preemption is not helpful for scheduling.
```

7. Create another pod named bee with the nginx image, which has a toleration set to the taint mortein.

- Image name: nginx
- Key: spray
- Value: mortein
- Effect: NoSchedule
- Status: Running

[Taints and Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)

```shell
k run bee --image=nginx --dry-run=client -o yaml > bee-pod.yaml

vi bee-pod.yaml 
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: bee
spec:
  containers:
  - image: nginx
    name: bee
  tolerations:
  - key: "spray"
    operator: "Equal"
    value: "mortein"
    effect: "NoSchedule"
```

```shell
k apply -f bee-pod.yaml 
pod/bee created

k get pod bee
NAME   READY   STATUS    RESTARTS   AGE
bee    1/1     Running   0          8s
```

8. Notice the bee pod was scheduled on node node01 despite the taint.

```shell
k get pod -o wide
NAME       READY   STATUS    RESTARTS   AGE     IP           NODE     NOMINATED NODE   READINESS GATES
bee        1/1     Running   0          58s     10.244.1.2   node01   <none>           <none>
mosquito   0/1     Pending   0          5m46s   <none>       <none>   <none>           <none>
```

9. Do you see any taints on controlplane node?

Yes - NoSchedule

```shell
k describe node controlplane

Taints:             node-role.kubernetes.io/control-plane:NoSchedule
```

10. Remove the taint on controlplane, which currently has the taint effect of NoSchedule.
- Node name: controlplane

```shell
k edit node controlplane 
node/controlplane edited
```

Remove
```yaml
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/control-plane
```

11. What is the state of the pod mosquito now?

Running

```shell
k get pod -o wide
NAME       READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
bee        1/1     Running   0          4m36s   10.244.1.2   node01         <none>           <none>
mosquito   1/1     Running   0          9m24s   10.244.0.4   controlplane   <none>           <none>
```

12. Which node is the POD mosquito on now?

controlplane
