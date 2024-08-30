1. Identify the number of containers created in the red pod.

3

```shell
k get pod red
NAME   READY   STATUS              RESTARTS   AGE
red    0/3     ContainerCreating   0          17s
```

2. Identify the name of the containers running in the blue pod.

teal & navy

```shell
k describe pod blue

Containers:
  teal:
    Container ID:  
    Image:         busybox
    Image ID:      
    Port:          <none>
    Host Port:     <none>
    Command:
      sleep
      4500
    State:          Waiting
      Reason:       ContainerCreating
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-tpd29 (ro)
  navy:
    Container ID:  
    Image:         busybox
    Image ID:      
    Port:          <none>
    Host Port:     <none>
    Command:
      sleep
      4500
    State:          Waiting
      Reason:       ContainerCreating
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-tpd29 (ro)
```

3. Create a multi-container pod with 2 containers.
Use the spec given below:

If the pod goes into the crashloopbackoff then add the command sleep 1000 in the lemon container.

- Name: yellow
- Container 1 Name: lemon
- Container 1 Image: busybox
- Container 2 Name: gold
- Container 2 Image: redis

```shell
k run yellow --image=busybox --dry-run=client -o yaml > yellow.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: yellow
spec:
  containers:
  - image: busybox
    name: lemon
  - image: redis
    name: gold
```

```shell
k create -f yellow.yaml 
pod/yellow created

k get po yellow 
NAME     READY   STATUS             RESTARTS     AGE
yellow   1/2     CrashLoopBackOff   1 (4s ago)   10s

vi yellow.yaml 
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: yellow
spec:
  containers:
  - image: busybox
    name: lemon
    command: ["sleep", "1000"]
  - image: redis
    name: gold
```

```shell
k replace --force -f yellow.yaml 
pod "yellow" deleted
pod/yellow replaced

k get po yellow
NAME     READY   STATUS    RESTARTS   AGE
yellow   2/2     Running   0          23s
```

4. We have deployed an application logging stack in the elastic-stack namespace. Inspect it.

Before proceeding with the next set of questions, please wait for all the pods in the elastic-stack namespace to be ready. This can take a few minutes.

```shell
k get all -n elastic-stack
NAME                 READY   STATUS    RESTARTS   AGE
pod/app              1/1     Running   0          7m19s
pod/elastic-search   1/1     Running   0          7m19s
pod/kibana           1/1     Running   0          7m18s

NAME                    TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)                         AGE
service/elasticsearch   NodePort   10.100.200.70    <none>        9200:30200/TCP,9300:30300/TCP   7m19s
service/kibana          NodePort   10.100.169.121   <none>        5601:30601/TCP                  7m18s
```

5. Once the pod is in a ready state, inspect the Kibana UI using the link above your terminal. There shouldn't be any logs for now.

We will configure a sidecar container for the application to send logs to Elastic Search.

NOTE: It can take a couple of minutes for the Kibana UI to be ready after the Kibana pod is ready.

You can inspect the Kibana logs by running:

kubectl -n elastic-stack logs kibana

```shell
kubectl -n elastic-stack logs kibana
```

6. Inspect the app pod and identify the number of containers in it.

It is deployed in the elastic-stack namespace.

1

```shell
k get po app -n elastic-stack
NAME   READY   STATUS    RESTARTS   AGE
app    1/1     Running   0          9m20s
```

7. The application outputs logs to the file /log/app.log. View the logs and try to identify the user having issues with Login.

Inspect the log file inside the pod.

USER5

```shell
k exec -it app -n elastic-stack -- less /log/app.log
[2024-08-30 13:08:29,397] INFO in event-simulator: USER2 is viewing page1
[2024-08-30 13:08:30,398] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAI
```

8. Edit the pod in the elastic-stack namespace to add a sidecar container to send logs to Elastic Search. Mount the log volume to the sidecar container.

Only add a new container. Do not modify anything else. Use the spec provided below.

Note: State persistence concepts are discussed in detail later in this course. For now please make use of the below documentation link for updating the concerning pod.

https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/

- Name: app
- Container Name: sidecar
- Container Image: kodekloud/filebeat-configured
- Volume Mount: log-volume
- Mount Path: /var/log/event-simulator/
- Existing Container Name: app
- Existing Container Image: kodekloud/event-simulator

```shell
k get po app -n elastic-stack -o yaml > app-pod.yaml

vi app-pod.yaml 
```

```yaml
  - image: kodekloud/filebeat-configured
    name: sidecar
    volumeMounts:
    - mountPath: /var/log/event-simulator/
      name: log-volume
```

```shell
k replace --force -f app-pod.yaml 
pod "app" deleted
pod/app replaced
```

9. Inspect the Kibana UI. You should now see logs appearing in the Discover section.

You might have to wait for a couple of minutes for the logs to populate. You might have to create an index pattern to list the logs. If not sure check this video: https://bit.ly/2EXYdHf

