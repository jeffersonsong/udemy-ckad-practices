1. How many pods exist on the system?

In the current(default) namespace.

0

```shell
k get po
No resources found in default namespace.
```

2. Create a new pod with the nginx image.

- Image name: nginx

```shell
k run nginx --image=nginx
pod/nginx created
```

3. How many pods are created now?

Note: We have created a few more pods. So please check again.

4

```shell
k get po
NAME            READY   STATUS    RESTARTS   AGE
newpods-kp9jh   1/1     Running   0          54s
newpods-2zb8v   1/1     Running   0          54s
newpods-4bf4q   1/1     Running   0          54s
nginx           1/1     Running   0          22s
```

4. What is the image used to create the new pods?

You must look at one of the new pods in detail to figure this out.

busybox

```shell
k describe po newpods-kp9jh | grep Image
    Image:         busybox
```

5. Which nodes are these pods placed on?

You must look at all the pods in detail to figure this out.

controlplane

```shell
k get po -o wide
NAME            READY   STATUS    RESTARTS   AGE    IP           NODE           NOMINATED NODE   READINESS GATES
newpods-kp9jh   1/1     Running   0          2m8s   10.42.0.9    controlplane   <none>           <none>
newpods-2zb8v   1/1     Running   0          2m8s   10.42.0.10   controlplane   <none>           <none>
newpods-4bf4q   1/1     Running   0          2m8s   10.42.0.11   controlplane   <none>           <none>
nginx           1/1     Running   0          96s    10.42.0.12   controlplane   <none>           <none>
```

6. How many containers are part of the pod webapp?

Note: We just created a new POD. Ignore the state of the POD for now.

2

```shell
k get po webapp
NAME     READY   STATUS         RESTARTS   AGE
webapp   1/2     ErrImagePull   0          17s
```

7. What images are used in the new webapp pod?

You must look at all the pods in detail to figure this out.

nginx & agentx

```shell
k describe po webapp | grep "Image:"
    Image:          nginx
    Image:          agentx
```

8. What is the state of the container agentx in the pod webapp?

Wait for it to finish the ContainerCreating state

```shell
k describe po webapp | less

  agentx:
    State:          Waiting
      Reason:       ImagePullBackOff

Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Pulling    70s (x4 over 2m37s)  kubelet            Pulling image "agentx"
  Warning  Failed     69s (x4 over 2m36s)  kubelet            Failed to pull image "agentx": failed to pull and unpack image "docker.io/library/agentx:latest": failed to resolve reference "docker.io/library/agentx:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed      
```

9. Why do you think the container agentx in pod webapp is in error?

Try to figure it out from the events section of the pod.

A Docker image with this name doesn't exist on Docker Hub.

10 . What does the READY column in the output of the kubectl get pods command indicate?

Running Containers in POD /Total Containers in POD

11. Delete the webapp Pod.

Once deleted, wait for the pod to fully terminate.

- Name: webapp

```shell
k delete pod webapp
pod "webapp" deleted
```

12. Create a new pod with the name redis and the image redis123.

Use a pod-definition YAML file. And yes the image name is wrong!

- Name: redis
- Image name: redis123

```shell
k run redis --image=redis123 --dry-run=client -o yaml > redis-pod.yaml

k apply -f redis-pod.yaml 
pod/redis created
```

13. Now change the image on this pod to redis.

Once done, the pod should be in a running state.

- Name: redis
- Image name: redis

```shell
k edit po redis
pod/redis edited
```
