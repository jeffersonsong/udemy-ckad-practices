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

3. How about now? How many ReplicaSets do you see?

We just made a few changes!

1

```shell
k get rs
NAME              DESIRED   CURRENT   READY   AGE
new-replica-set   4         4         0       9s
```

4. How many PODs are DESIRED in the new-replica-set?

4

5. What is the image used to create the pods in the new-replica-set?

busybox777

```shell
k describe rs new-replica-set | grep "Image:"
    Image:      busybox777
```

6. How many PODs are READY in the new-replica-set?

0

```shell
k get rs
NAME              DESIRED   CURRENT   READY   AGE
new-replica-set   4         4         0       85s
```

7. Why do you think the PODs are not ready?

The image BUSYBOX777 doesn't exist.

```shell
k get po
NAME                    READY   STATUS             RESTARTS   AGE
new-replica-set-h728v   0/1     ImagePullBackOff   0          2m12s
new-replica-set-v7lmh   0/1     ImagePullBackOff   0          2m12s
new-replica-set-5wtln   0/1     ImagePullBackOff   0          2m12s
new-replica-set-grvbl   0/1     ImagePullBackOff   0          2m12s

k describe po new-replica-set-h728v
Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Pulling    68s (x4 over 2m38s)  kubelet            Pulling image "busybox777"
  Warning  Failed     68s (x4 over 2m38s)  kubelet            Failed to pull image "busybox777": failed to pull and unpack image "docker.io/library/busybox777:latest": failed to resolve reference "docker.io/library/busybox777:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
```

8. Delete any one of the 4 PODs.

```shell
k get po
NAME                    READY   STATUS             RESTARTS   AGE
new-replica-set-h728v   0/1     ImagePullBackOff   0          3m38s
new-replica-set-v7lmh   0/1     ImagePullBackOff   0          3m38s
new-replica-set-grvbl   0/1     ImagePullBackOff   0          3m38s
new-replica-set-5wtln   0/1     ImagePullBackOff   0          3m38s

k delete po new-replica-set-h728v
pod "new-replica-set-h728v" deleted

k get po
NAME                    READY   STATUS             RESTARTS   AGE
new-replica-set-v7lmh   0/1     ImagePullBackOff   0          3m51s
new-replica-set-grvbl   0/1     ImagePullBackOff   0          3m51s
new-replica-set-5wtln   0/1     ImagePullBackOff   0          3m51s
new-replica-set-ctwts   0/1     ErrImagePull       0          4s
```

9. How many PODs exist now?

4

10. Why are there still 4 PODs, even after you deleted one?

ReplicaSet ensures that desired number of PODs always run

11. Create a ReplicaSet using the replicaset-definition-1.yaml file located at /root/.

There is an issue with the file, so try to fix it.

- Name: replicaset-1

```shell
k apply -f replicaset-definition-1.yaml 
error: resource mapping not found for name: "replicaset-1" namespace: "" from "replicaset-definition-1.yaml": no matches for kind "ReplicaSet" in version "v1"
ensure CRDs are installed first

k api-resources | grep replicaset
replicasets                         rs           apps/v1                           true         ReplicaSet

k apply -f replicaset-definition-1.yaml 
replicaset.apps/replicaset-1 created
```

12. Fix the issue in the replicaset-definition-2.yaml file and create a ReplicaSet using it.
This file is located at /root/.

Name: replicaset-2

```shell
k apply -f replicaset-definition-2.yaml 
The ReplicaSet "replicaset-2" is invalid: spec.template.metadata.labels: Invalid value: map[string]string{"tier":"nginx"}: `selector` does not match template `labels`

vi replicaset-definition-2.yaml 

k apply -f replicaset-definition-2.yaml 
replicaset.apps/replicaset-2 created

13. Delete the two newly created ReplicaSets - replicaset-1 and replicaset-2

- Delete: replicaset-2
- Delete: replicaset-1

k delete rs replicaset-1
replicaset.apps "replicaset-1" deleted

k delete rs replicaset-2
replicaset.apps "replicaset-2" deleted
```

14. Fix the original replica set new-replica-set to use the correct busybox image.

Either delete and recreate the ReplicaSet or Update the existing ReplicaSet and then delete all PODs, so new ones with the correct image will be created.

- Replicas: 4

```shell
k edit rs new-replica-set 
replicaset.apps/new-replica-set edited

k get po
NAME                    READY   STATUS             RESTARTS   AGE
new-replica-set-grvbl   0/1     ImagePullBackOff   0          10m
new-replica-set-v7lmh   0/1     ImagePullBackOff   0          10m
new-replica-set-5wtln   0/1     ImagePullBackOff   0          10m
new-replica-set-ctwts   0/1     ImagePullBackOff   0          7m2s

k scale rs new-replica-set --replicas=0
replicaset.apps/new-replica-set scaled

k get po
No resources found in default namespace.

k scale rs new-replica-set --replicas=4
replicaset.apps/new-replica-set scaled

k get po
NAME                    READY   STATUS    RESTARTS   AGE
new-replica-set-mg26d   1/1     Running   0          8s
new-replica-set-qzzm5   1/1     Running   0          8s
new-replica-set-rf95f   1/1     Running   0          8s
new-replica-set-rfj9q   1/1     Running   0          8s
```

15. Scale the ReplicaSet to 5 PODs.
Use kubectl scale command or edit the replicaset using kubectl edit replicaset.

- Replicas: 5

```shell
k edit rs new-replica-set 
replicaset.apps/new-replica-set edited

k get po
NAME                    READY   STATUS    RESTARTS   AGE
new-replica-set-mg26d   1/1     Running   0          72s
new-replica-set-qzzm5   1/1     Running   0          72s
new-replica-set-rf95f   1/1     Running   0          72s
new-replica-set-rfj9q   1/1     Running   0          72s
new-replica-set-np4l7   1/1     Running   0          11s
```

16. Now scale the ReplicaSet down to 2 PODs.

Use the kubectl scale command or edit the replicaset using kubectl edit replicaset.

- Replicas: 2

```shell
k scale rs new-replica-set --replicas=2
replicaset.apps/new-replica-set scaled

k get po
NAME                    READY   STATUS        RESTARTS   AGE
new-replica-set-rf95f   1/1     Running       0          114s
new-replica-set-rfj9q   1/1     Running       0          114s
new-replica-set-mg26d   1/1     Terminating   0          114s
new-replica-set-np4l7   1/1     Terminating   0          53s
new-replica-set-qzzm5   1/1     Terminating   0          114s
```
