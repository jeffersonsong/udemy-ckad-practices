1. A pod called rabbit is deployed. Identify the CPU requirements set on the Pod

in the current(default) namespace

1

```shell
k describe po rabbit

Containers:
  cpu-stress:
    Limits:
      cpu:  2
    Requests:
      cpu:  1
```

2. Delete the rabbit Pod.

Once deleted, wait for the pod to fully terminate.

- Delete Pod rabbit

```shell
k delete po rabbit
pod "rabbit" deleted
```

3. Another pod called elephant has been deployed in the default namespace. It fails to get to a running state. Inspect this pod and identify the Reason why it is not running.

OOMKilled

```shell
k get po elephant
NAME       READY   STATUS      RESTARTS      AGE
elephant   0/1     OOMKilled   2 (17s ago)   20s

k describe po elephant

Containers:
  mem-stress:
    Container ID:  containerd://e5f07d681a27fadab4d7f25be0b67dca642125f8368360159ab9bb3fad79dcb3
    Image:         polinux/stress
    Image ID:      docker.io/polinux/stress@sha256:b6144f84f9c15dac80deb48d3a646b55c7043ab1d83ea0a697c09097aaad21aa
    Port:          <none>
    Host Port:     <none>
    Command:
      stress
    Args:
      --vm
      1
      --vm-bytes
      15M
      --vm-hang
      1
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       OOMKilled
      Exit Code:    1
      Started:      Fri, 30 Aug 2024 17:59:58 +0000
      Finished:     Fri, 30 Aug 2024 17:59:58 +0000
    Ready:          False
    Restart Count:  2
    Limits:
      memory:  10Mi
    Requests:
      memory:     5Mi

Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Warning  BackOff    12s (x4 over 39s)  kubelet            Back-off restarting failed container mem-stress in pod elephant_default(082bac90-e298-4e33-b97b-d47d81339a51)
```

4. The status OOMKilled indicates that it is failing because the pod ran out of memory. Identify the memory limit set on the POD.

5. The elephant pod runs a process that consumes 15Mi of memory. Increase the limit of the elephant pod to 20Mi.

Delete and recreate the pod if required. Do not modify anything other than the required fields.

- Pod Name: elephant
- Image Name: polinux/stress
- Memory Limit: 20Mi

```shell
k edit po elephant
```

```yaml
  containers:
  - args:
    - --vm
    - "1"
    - --vm-bytes
    - 15M
    - --vm-hang
    - "1"
    command:
    - stress
    image: polinux/stress
    imagePullPolicy: Always
    name: mem-stress
    resources:
      limits:
        memory: 20Mi
      requests:
        memory: 5Mi
```

```shell
k replace --force -f /tmp/kubectl-edit-1023833296.yaml
pod "elephant" deleted
pod/elephant replaced
```

6. Inspect the status of POD. Make sure it's running

```shell
k get po elephant
NAME       READY   STATUS    RESTARTS   AGE
elephant   1/1     Running   0          39s
```

7. Delete the elephant Pod.

Once deleted, wait for the pod to fully terminate.

- Delete Pod elephant

```shell
k delete po elephant
pod "elephant" deleted
```