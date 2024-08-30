1. Identify the pod that has an initContainer configured.

blue

```shell
k get po -o yaml | less
```

2. What is the image used by the initContainer on the blue pod?

busybox

```shell
k get po blue -o yaml
```

```yaml
  initContainers:
  - command:
    - sh
    - -c
    - sleep 5
    image: busybo
```

3. What is the state of the initContainer on pod blue?

Terminated

```shell
k describe po blue 

    State:          Terminated
      Reason:       Completed
```

4. Why is the initContainer terminated? What is the reason?

The process completed successfully

5. We just created a new app named purple. How many initContainers does it have?

2

```shell
k get po purple -o yaml | less
```

6. What is the state of the POD?

Pending

```shell
k describe po purple
Name:             purple
Status:           Pending
```

7. How long after the creation of the POD will the application come up and be available to users?

30 minutes
```
Init Containers:
  warm-up-1:
    Container ID:  containerd://8cfbe9b0f47d2027e30af2598d322aa795811dedc9755f00336f7adf99b4c34e
    Image:         busybox:1.28
    Image ID:      docker.io/library/busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      sleep 600
    State:          Running
      Started:      Fri, 30 Aug 2024 13:34:50 +0000
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-75cpv (ro)
  warm-up-2:
    Container ID:  
    Image:         busybox:1.28
    Image ID:      
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      sleep 1200
    State:          Waiting
      Reason:       PodInitializing
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-75cpv (ro)
```

8. Update the pod red to use an initContainer that uses the busybox image and sleeps for 20 seconds

Delete and re-create the pod if necessary. But make sure no other configurations change.

- Pod: red
- initContainer Configured Correctly

[Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)

```shell
k edit po red
```

```yaml
spec:
  containers:
  - command:
    - sh
    - -c
    - echo The app is running! && sleep 3600
    image: busybox:1.28
    imagePullPolicy: IfNotPresent
    name: red-container
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-62qj2
      readOnly: true
  initContainers:
  - name: init-red
    image: busybox
    command: ["sleep", "20"]
```

```shell
controlplane ~ ✖ k replace --force -f /tmp/kubectl-edit-66158036.yaml
```

9. A new application orange is deployed. There is something wrong with it. Identify and fix the issue.


Once fixed, wait for the application to run before checking solution.

- Issue fixed

```shell
k describe po orange
```

```
Init Containers:
  init-myservice:
    Container ID:  containerd://047cc4a536d302fd545007b313fd0412d1f16d7fd8ad99e2f5642a940fae7362
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:9ae97d36d26566ff84e8893c64a6dc4fe8ca6d1144bf5b87b2b85a32def253c7
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      sleeeep 2;
    State:          Terminated
      Reason:       Error
      Exit Code:    127
      Started:      Fri, 30 Aug 2024 13:43:10 +0000
      Finished:     Fri, 30 Aug 2024 13:43:10 +0000
    Last State:     Terminated
      Reason:       Error
      Exit Code:    127
      Started:      Fri, 30 Aug 2024 13:42:54 +0000
      Finished:     Fri, 30 Aug 2024 13:42:54 +0000
    Ready:          False
    Restart Count:  2
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-2pdln (ro)

Events:
  Type     Reason     Age               From               Message
  ----     ------     ----              ----               -------
  Warning  BackOff    4s (x3 over 20s)  kubelet            Back-off restarting failed container init-myservice in pod orange_default(64aeaf0f-3d3b-4a59-a16d-0eb8bbf16730)
```

```shell
k edit po orange
```

```yaml
  initContainers:
  - command:
    - sh
    - -c
    - sleeeep 2;
```

```yaml
  initContainers:
  - command:
    - sh
    - -c
    - sleep 2;
```

```shell
controlplane ~ ✖ k replace --force -f /tmp/kubectl-edit-534011094.yaml
pod "orange" deleted
pod/orange replaced
```
