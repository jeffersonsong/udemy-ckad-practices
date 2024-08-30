1. What is the user used to execute the sleep process within the ubuntu-sleeper pod?

In the current(default) namespace.

root

```shell
k get pod

k exec -it ubuntu-sleeper -- /bin/sh
# ps aux | grep sleep
root           1  0.0  0.0   2692  1100 ?        Ss   17:21   0:00 sleep 4800
root          67  0.0  0.0   3524  1692 pts/0    S+   17:23   0:00 grep sleep
```

2. dit the pod ubuntu-sleeper to run the sleep process with user ID 1010.

Note: Only make the necessary changes. Do not modify the name or image of the pod.

- Pod Name: ubuntu-sleeper
- Image Name: ubuntu
- SecurityContext: User 1010

[Set the security context for a Pod](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#set-the-security-context-for-a-pod)

```shell
k edit po ubuntu-sleeper
```

```yaml
  securityContext: 
    runAsUser: 1010
```

controlplane ~ ✖ k replace --force -f /tmp/kubectl-edit-3429113325.yaml

3. A Pod definition file named multi-pod.yaml is given. With what user are the processes in the web container started?

The pod is created with multiple containers and security contexts defined at the Pod and Container level.

1002

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-pod
spec:
  securityContext:
    runAsUser: 1001
  containers:
  -  image: ubuntu
     name: web
     command: ["sleep", "5000"]
     securityContext:
      runAsUser: 1002

  -  image: ubuntu
     name: sidecar
     command: ["sleep", "5000"]
```

4. With what user are the processes in the sidecar container started?

The pod is created with multiple containers and security contexts defined at the Pod and Container level.

1001

5. Update pod ubuntu-sleeper to run as Root user and with the SYS_TIME capability.

Note: Only make the necessary changes. Do not modify the name of the pod.

- Pod Name: ubuntu-sleeper
- Image Name: ubuntu
- SecurityContext: Capability SYS_TIME

Is run as a root user?

```shell
k edit po ubuntu-sleeper
```

```yaml
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    imagePullPolicy: Always
    name: ubuntu
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-vdxmg
      readOnly: true
    securityContext:
      capabilities:
        add: ["SYS_TIME"]
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: controlplane
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext:
    runAsUser: 0
```

```shell
k replace --force -f /tmp/kubectl-edit-450178159.yaml
```

6. Now update the pod to also make use of the NET_ADMIN capability.

Note: Only make the necessary changes. Do not modify the name of the pod.

- Pod Name: ubuntu-sleeper
- Image Name: ubuntu
- SecurityContext: Capability SYS_TIME
- SecurityContext: Capability NET_ADMIN

```shell
k edit po ubuntu-sleeper
```

```yaml
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    imagePullPolicy: Always
    name: ubuntu
    resources: {}
    securityContext:
      capabilities:
        add:
        - SYS_TIME
        - NET_ADMIN
```

```shell
k replace --force -f /tmp/kubectl-edit-2592888941.yaml
```

