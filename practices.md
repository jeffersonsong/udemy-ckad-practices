# Kubernetes Certified Application Developer (CKAD) with Tests - Practice Tests

## Section 2: Core Concepts

### [19. Pods](https://uklabs.kodekloud.com/topic/pods-4/)

```shell
k get po
NAME            READY   STATUS    RESTARTS   AGE
newpods-kp9jh   1/1     Running   0          54s

k get po -o wide
NAME            READY   STATUS    RESTARTS   AGE    IP           NODE           NOMINATED NODE   READINESS GATES
newpods-kp9jh   1/1     Running   0          2m8s   10.42.0.9    controlplane   <none>           <none>

k describe po webapp

k run nginx --image=nginx

k edit po nginx
k replace --force -f /tmp/kubectl-edit-3603704817.yaml

k delete pod nginx

k expose pod redis --port=6379 --name=redis-service --type=ClusterIP

k api-resources | grep pod
pods                                po           v1                                true         Pod
```

### [23. ReplicaSets](https://uklabs.kodekloud.com/topic/replicasets-2/)

```shell
k get rs
NAME              DESIRED   CURRENT   READY   AGE
new-replica-set   4         4         0       9s

k describe rs new-replica-set
```

```shell
k apply -f replicaset-definition-1.yaml
```

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-1
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
```

```shell
k api-resources | grep replicaset
replicasets                         rs           apps/v1                           true         ReplicaSet

k edit rs new-replica-set 

k delete rs replicaset-1

k scale rs new-replica-set --replicas=4

k expose rs nginx --port=80 --target-port=8000
```

### [25. Deployments](https://uklabs.kodekloud.com/topic/deployments-5/)

```shell
k get deploy

k create deployment httpd-frontend --image=httpd:2.4-alpine --replicas=3

k get all
NAME                                  READY   STATUS    RESTARTS   AGE
pod/httpd-frontend-5bf76dfdf9-5nzsr   1/1     Running   0          4s
pod/httpd-frontend-5bf76dfdf9-5zbgz   1/1     Running   0          4s
pod/httpd-frontend-5bf76dfdf9-ghm6x   1/1     Running   0          4s

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/httpd-frontend   3/3     3            3           4s

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/httpd-frontend-5bf76dfdf9   3         3         3       4s

k scale deploy httpd-frontend --replicas=1

k delete deploy httpd-frontend

k expose deployment httpd-frontend --port=80 --target-port=8000

k api-resources | grep deployment
deployments                         deploy       apps/v1                           true         Deployment
```

### [30. Namespaces](https://uklabs.kodekloud.com/topic/namespaces-3/)

```shell
k get ns
k create ns dev-ns

k api-resources | grep namespace 
namespaces                          ns           v1                                false        Namespace
```

### [33. Imperative Commands](https://uklabs.kodekloud.com/topic/imperative-commands/)


## Section 3: Configuration

### [38. Docker Images](https://uklabs.kodekloud.com/topic/practice-test-docker-images-2/)

```shell
docker image ls

docker image ls webapp-color
```

Dockerfile
```
FROM python:3.6-alpine 
RUN pip install flask
COPY . /opt/
EXPOSE 8080
WORKDIR /opt
ENTRYPOINT ["python", "app.py"]
```

```shell
docker build -t webapp-color .

docker run -p 8282:8080 webapp-color

docker ps

docker exec -it 94db58651d02 cat /etc/os-release
```

### [42. Commands and Arguments](https://uklabs.kodekloud.com/topic/commands-and-arguments/)

Dockerfile
```
FROM python:3.6-alpine
RUN pip install flask
COPY . /opt/
EXPOSE 8080
WORKDIR /opt
ENTRYPOINT ["python", "app.py"]
CMD ["--color", "red"]
```

webapp-color-pod.yaml
```yaml
apiVersion: v1 
kind: Pod 
metadata:
  name: webapp-green
  labels:
      name: webapp-green 
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    command: ["python", "app.py"]
    args: ["--color", "pink"]
```

### [46. ConfigMaps](https://uklabs.kodekloud.com/topic/configmaps-2/)

[Configure a Pod to Use a ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)

```shell
k get cm

k describe cm db-config

kubectl create configmap webapp-config-map --from-literal=APP_COLOR=darkblue --from-literal=APP_OTHER=disregard

k api-resources | grep configmap
configmaps                          cm           v1                                true         ConfigMap
```

```yaml
  containers:
  - env:
    - name: APP_COLOR
      value: green
```

```yaml
    env:
    - name: APP_COLOR
      valueFrom:
        configMapKeyRef:
          name: webapp-config-map
          key: APP_COLOR
```

```yaml
    envFrom:
    - configMapRef:
        name: webapp-config-map
```

### [51. Secrets](https://uklabs.kodekloud.com/topic/secrets-4/)

[Configure all key-value pairs in a Secret as container environment variables](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#configure-all-key-value-pairs-in-a-secret-as-container-environment-variables)

```shell
k get secret

k describe secret dashboard-token 

k create secret generic db-secret --from-literal=DB_Host=sql01 \
  --from-literal=DB_User=root \
  --from-literal=DB_Password=password123

k create secret tls webhook-server-tls --cert=/root/keys/webhook-server-tls.crt \ 
  --key=/root/keys/webhook-server-tls.key -n webhook-demo

k api-resources | grep secret   
secrets                                          v1                                true         Secret
```

```yaml
    envFrom:
    - secretRef:
        name: db-secret
```

### [56. Security Contexts](https://uklabs.kodekloud.com/topic/security-contexts-3/)

[Set the security context for a Pod](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#set-the-security-context-for-a-pod)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  securityContext: 
    runAsUser: 1001
  containers:
  - name: web
    securityContext:
      runAsUser: 1002
      capabilities:
        add: ["SYS_TIME"]
```

### [59. Service Acccount](https://uklabs.kodekloud.com/topic/service-account-2/)

```shell
k create sa dashboard-sa

kubectl create token dashboard-sa

k api-resources | grep serviceaccount
serviceaccounts                     sa           v1                                true         ServiceAccount
```

[Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

Pod Spec
```yaml
spec:
  template:
    spec:
      serviceAccountName: dashboard-sa
      containers:
```


### [62. Resource Requirements](https://uklabs.kodekloud.com/topic/resource-limits-2/)

Container Spec
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

### [65. Taints and Toleration](https://uklabs.kodekloud.com/topic/taints-and-tolerations-3/)

[Taints and Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)

```shell
kubectl taint nodes node01 spray=mortein:NoSchedule
```

```yaml
apiVersion: v1
kind: Node
metadata:
  annotations: {}
  labels: {}
  name: node01
spec:
  podCIDR: 10.244.1.0/24
  podCIDRs:
  - 10.244.1.0/24
  taints:
  - effect: NoSchedule
    key: spray
    value: mortein
```

Pod Spec
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
k describe node controlplane
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
```

### [69. Node Affinity](https://uklabs.kodekloud.com/topic/node-affinity-3/)
[Schedule a Pod using required node affinity](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/)

```shell
kubectl label nodes node01 color=blue
```

Pod spec
```yaml
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

```yaml
            - matchExpressions:
              - key: node-role.kubernetes.io/control-plane
                operator: Exists
```

## Section 4: Multi-Container Pods
### [77. Multi-Container Pods](https://uklabs.kodekloud.com/topic/multi-container-pods-3/)

[Pods with multiple containers](https://kubernetes.io/docs/concepts/workloads/pods/#how-pods-manage-multiple-containers)
[Communicate Between Containers in the Same Pod Using a Shared Volume](https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/)

Pod Spec
```yaml
spec:
  containers:
  - image: busybox
    name: lemon
  - image: redis
    name: gold
```

```shell
k get po yellow 
NAME     READY   STATUS             RESTARTS     AGE
yellow   1/2     CrashLoopBackOff   1 (4s ago)   10s
```

### [79. Init Containers](https://uklabs.kodekloud.com/topic/init-containers-3/)
  
[Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)

Pod Spec
```yaml
spec:
  containers:
  - name: red-container
  initContainers:
  - name: init-red
    image: busybox
    command: ["sleep", "20"]
```

## Section 5: Observability

### [83. Readiness and Liveness Probes](https://uklabs.kodekloud.com/topic/readiness-probes-2/)

[Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

Container Spec
```yaml
spec:
  containers:
  - name: simple-webapp
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
    livenessProbe:
      httpGet:
        path: /live
        port: 8080
      periodSeconds: 1
      initialDelaySeconds: 80
```

### [86. Container Logging](https://uklabs.kodekloud.com/topic/logging-2/)

```shell
k logs webapp-2
```

### [89. Monitoring](https://uklabs.kodekloud.com/topic/monitoring-2/)

```shell
k top node
NAME           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
controlplane   270m         0%     1126Mi          0%        
node01         23m          0%     303Mi           0%   

k top pod
NAME       CPU(cores)   MEMORY(bytes)   
elephant   14m          32Mi            
lion       1m           18Mi            
rabbit     107m         252Mi   
```

## Section 6: POD Design
### [92. Labels, Selectors and Annotations](https://uklabs.kodekloud.com/topic/labels-and-selectors-2/)
```shell
k get po --selector="env=prod,bu=finance,tier=frontend" 

k get po --show-labels
```

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
   name: replicaset-1
spec:
   replicas: 2
   selector:
      matchLabels:
        tier: front-end
   template:
     metadata:
       labels:
        tier: front-end
     spec:
       containers:
       - name: nginx
         image: nginx
```

### [97. Rolling Updates & Rollbacks](https://uklabs.kodekloud.com/topic/rolling-updates-rollbacks-2/)

```shell
kubectl set image deployment/frontend simple-webapp=kodekloud/webapp-color:v2
```

```yaml
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
```

```yaml
  strategy:
    type: Recreate
```

### [101. Deployment strategies](https://uklabs.kodekloud.com/topic/practice-test-de…ent-strategies-2/)

```shell
k get deploy -o wide
NAME          READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS     IMAGES                      SELECTOR
frontend      5/5     5            5           5m25s   webapp-color   kodekloud/webapp-color:v1   app=frontend
frontend-v2   1/1     1            1           110s    webapp-color   kodekloud/webapp-color:v2   app=frontend
```

### [105. Jobs & CronJobs](https://uklabs.kodekloud.com/topic/jobs-and-cronjobs/)

- [Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/)
- [CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)

```shell
k create job throw-dice-job --image=kodekloud/throw-dice

k get job throw-dice-job -w

k describe job throw-dice-job

k delete job throw-dice-job 

k create cronjob throw-dice-cron-job --image=kodekloud/throw-dice --schedule="30 21 * * *"

k api-resources | grep "job\|cronjob"
cronjobs                            cj           batch/v1                          true         CronJob
jobs                                             batch/v1                          true         Job
```

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: throw-dice-job
spec:
  backoffLimit: 20
  completions: 3
  parallelism: 3
  template:
    metadata:
      labels:
        job-name: throw-dice-job
    spec:
      containers:
      - image: kodekloud/throw-dice
        imagePullPolicy: Always
        name: throw-dice-job
      restartPolicy: Never
```


## Section 7. Services & Networking
### [109. Services](https://uklabs.kodekloud.com/topic/kubernetes-services/)

```shell
k get svc

k get svc -o wide

k get svc kubernetes --show-labels
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   LABELS
kubernetes   ClusterIP   10.43.0.1    <none>        443/TCP   26m   component=apiserver,provider=kubernetes

k describe svc kubernetes 

k api-resources | grep "service"     
services                            svc          v1                                true         Service

k create -f service-definition-1.yaml 
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
  namespace: default
spec:
  ports:
  - nodePort: 30080
    port: 8080
    targetPort: 8080
  selector:
    name: simple-webapp
  type: NodePort
```

### [113. Ingress Networking - 1](https://uklabs.kodekloud.com/topic/ingress-networking-1/)

[Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
  - [Ingress - Path types](https://kubernetes.io/docs/concepts/services-networking/ingress/#path-types)
  - [Ingress - The Ingress resource](https://kubernetes.io/docs/concepts/services-networking/ingress/#the-ingress-resource)
[kubectl create ingress](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_create/kubectl_create_ingress/)
  
```shell
k get ingress -A

k describe ingress ingress-wear-watch -n app-space

kubectl create ingress ingress-wear-watch   --rule="/wear*=wear-service:8080" \
  --rule="/watch*=video-service:8080" \ 
  --annotation nginx.ingress.kubernetes.io/rewrite-target=/ -n app-space

k api-resources | grep ingress  
ingresses                           ing          networking.k8s.io/v1              true         Ingress
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  creationTimestamp: null
  name: ingress-wear-watch
  namespace: app-space
spec:
  rules:
  - http:
      paths:
      - backend:
          service:
            name: wear-service
            port:
              number: 8080
        path: /wear
        pathType: Prefix
      - backend:
          service:
            name: video-service
            port:
              number: 8080
        path: /watch
        pathType: Prefix
status:
  loadBalancer: {}
```

### [116. Ingress Networking - 2](https://uklabs.kodekloud.com/topic/ingress-networking-2/)

### [120. Network Policies](https://uklabs.kodekloud.com/topic/network-policies-4/)
[Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

```shell
k get netpol -A

k get pod --selector="name=payroll"

k apply -f internal-policy.yaml 

k describe netpol payroll-policy
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          name: payroll
    ports:
    - protocol: TCP
      port: 8080
  - to:
    - podSelector:
        matchLabels:
          name: mysql
    ports:
    - protocol: TCP
      port: 3306
```

## Section 8. State Persistence
### [126. Persistent Volumes](https://uklabs.kodekloud.com/topic/persistent-volumes-5/)

[Configure a Pod to Use a PersistentVolume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)
[Volumes - hostPath configuration example](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath-configuration-example)

```shell
k get pv

k get pvc

k delete pvc claim-log-1 

k api-resources | grep "persistentvolume"
persistentvolumeclaims              pvc          v1                                true         PersistentVolumeClaim
persistentvolumes                   pv           v1                                false        PersistentVolume
```

Pod Spec
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - image: kodekloud/event-simulator
    name: event-simulator
    volumeMounts:
    - mountPath: /log
      name: log-volume
  volumes:
  - name: log-volume
    hostPath:
      path: /var/log/webapp
```

Pod Spec
```yaml
spec:
  volumes:
  - name: log-volume
    persistentVolumeClaim:
      claimName: claim-log-1
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
  storageClassName: ""
  capacity:
    storage: 100Mi
  persistentVolumeReclaimPolicy: Retain
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /pv/log
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteMany
  volumeMode: Filesystem
  volumeName: pv-log
  resources:
    requests:
      storage: 50Mi
```
  
### [130. Storage Class](https://uklabs.kodekloud.com/topic/storage-class-2/)

[Configure a Pod to Use a PersistentVolume for Storage - Create a PersistentVolumeClaim](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolumeclaim)

```shell
k api-resources | grep storageclass
storageclasses                      sc           storage.k8s.io/v1                 false        StorageClass

k get sc

k get sc local-storage 

k describe sc
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
spec:
  storageClassName: local-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

Pod Spec
```yaml
  volumes:
  - name: pv-volume
    persistentVolumeClaim:
     claimName: local-pvc
```

```shell
vi delayed-volume-sc.yaml

k create -f delayed-volume-sc.yaml 

k get sc delayed-volume-sc
```

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: delayed-volume-sc
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Retain # default value is Delete
volumeBindingMode: WaitForFirstConsumer
```

## Section 9: Security
### [140. KubeConfig](https://uklabs.kodekloud.com/topic/practice-test-kubeconfig-4/)

```shell
ls ~/.kube/config

k config view

k config view --kubeconfig=my-kube-config

k config use-context research --kubeconfig=my-kube-config
```

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://controlplane:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
```

### [145. Role Based Access Controls](https://uklabs.kodekloud.com/topic/practice-test-ro…ccess-controls-4/)

```shell
k cluster-info dump | grep authorization-mode
                            "--authorization-mode=Node,RBAC",

k get role

k describe role kube-proxy -n kube-system
Name:         kube-proxy
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources   Non-Resource URLs  Resource Names  Verbs
  ---------   -----------------  --------------  -----
  configmaps  []                 [kube-proxy]    [get]


k get rolebinding -n kube-system

k describe rolebinding kube-proxy -n kube-system

k auth can-i --as dev-user get pods

kubectl create role developer --verb=list,create,delete,get --resource=pods

kubectl create rolebinding dev-user-binding --role=developer --user=dev-user
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: "2024-08-29T17:27:14Z"
  name: developer
  namespace: blue
  resourceVersion: "589"
  uid: c802b247-727b-4fe8-85e7-8a92e9d040ed
rules:
- apiGroups:
  - ""
  resourceNames:
  - blue-app
  - dark-blue-app
  resources:
  - pods
  verbs:
  - get
  - watch
  - create
  - delete
```


### [148. Cluster Roles](https://uklabs.kodekloud.com/topic/practice-test-cluster-roles-4/)

```shell
k get clusterrole,clusterrolebinding

k describe clusterrole cluster-admin
Name:         cluster-admin
Labels:       kubernetes.io/bootstrapping=rbac-defaults
Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  *.*        []                 []              [*]
             [*]                []              [*]

kubectl create clusterrole storage-admin --verb=get,list,watch --resource=persistentvolumes,storageclasses

kubectl create clusterrolebinding michelle-storage-admin --clusterrole=storage-admin --user=michelle
```

### [151. Admission Controllers](https://uklabs.kodekloud.com/topic/labs-admission-controllers-5/)

- [Admission Controllers Reference](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)

```shell
k cluster-info dump | grep admission
                            "--enable-admission-plugins=NodeRestriction",

vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

```yaml
spec:
  containers:
  - command:
    - kube-apiserver
    - --enable-admission-plugins=NodeRestriction
```
to
```yaml
spec:
  containers:
  - command:
    - kube-apiserver
    - --enable-admission-plugins=NodeRestriction,NamespaceAutoProvision
    - --disable-admission-plugins=DefaultStorageClass
```

```shell
ps -ef | grep kube-apiserver | grep admission-plugins
root       14491   14278  0 18:46 ?        00:00:03 kube-apiserver --advertise-address=192.39.26.6 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction,NamespaceAutoProvision --disable-admission-plugins=DefaultStorageClass --enable-bootstrap-token-auth=true --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key --etcd-servers=https://127.0.0.1:2379 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6443 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-account-signing-key-file=/etc/kubernetes/pki/sa.key --service-cluster-ip-range=10.96.0.0/12 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
```

### [154. Validating and Mutating Admission Controllers](https://uklabs.kodekloud.com/topic/labs-validating-and-mutating-admission-controllers-5/)

[Dynamic Admission Control](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/)

```shell
kubectl create secret tls webhook-server-tls --cert=/root/keys/webhook-server-tls.crt --key=/root/keys/webhook-server-tls.key -n webhook-demo 
```

webhook-configuration.yaml 
```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
metadata:
  name: demo-webhook
webhooks:
  - name: webhook-server.webhook-demo.svc
    clientConfig:
      service:
        name: webhook-server
        namespace: webhook-demo
        path: "/mutate"
      caBundle: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURQekNDQWllZ0F3SUJBZ0lVSCtOdjR4R3BndzVoNHNlWTUyemlYMVN5MDFZd0RRWUpLb1pJaHZjTkFRRUwKQlFBd0x6RXRNQ3NHQTFVRUF3d2tRV1J0YVhOemFXOXVJRU52Ym5SeWIyeHNaWElnVjJWaWFHOXZheUJFWlcxdgpJRU5CTUI0WERUSTBNRGd5T1RFNE5USXlORm9YRFRJME1Ea3lPREU0TlRJeU5Gb3dMekV0TUNzR0ExVUVBd3drClFXUnRhWE56YVc5dUlFTnZiblJ5YjJ4c1pYSWdWMlZpYUc5dmF5QkVaVzF2SUVOQk1JSUJJakFOQmdrcWhraUcKOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXcxTjhTWkpJdjUxZDhjYWZ2Qi9PNSs4SmxwSlVRbXdWczRGZAptUldQeDU1alI2dTRTdmcrK2VJekNQbXZIbXcwQUtwVnRyQ3c5QVF2QUZSVFBuQmJFNURZSmNOTnZtTDVPRytQClpSekpnMXl4Y2ZhUlFOWWR1VmY0d1YrZnc5eGNkVExRdWRaeWJPOGFScmt3RlJPM2lBYTM0bElwYkh1bEtjcnMKRy84NCtEV2xDVzUreTJJUTRTNUtWOWNoU1hSKzNNSW13M2NwcGxkQzZuYWNudUVGSUdxbVdDcHFqclo1Q2l1dQpaT3RKMkN4K09pY2syVnJrOVBPSnRxYjN3OXpvRHIwYTNDYWVkT04xZzFkdDRxeEVUeTV6QTZyejVFVVBsaVY2Ci8ydVdWSWh6Wm9sNjVGTEJYWTQyMFNsMWlCeDVwYk9wbnZ6V0ErUDZPRGRXdkdhbUx3SURBUUFCbzFNd1VUQWQKQmdOVkhRNEVGZ1FVWjgwaWxiZkRhVmpiSEN6MkFKRzdVY0hadHVFd0h3WURWUjBqQkJnd0ZvQVVaODBpbGJmRAphVmpiSEN6MkFKRzdVY0hadHVFd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBTkJna3Foa2lHOXcwQkFRc0ZBQU9DCkFRRUFEL0hwYTJTY2Z5djdCVEplbnBGZGtqWTY1dExYZ09HU1BZREJyMlVESHhibGE0V091eThVZDRTNXdyTjIKT1M0dTBEZm1pRU01Ny9tUWM1RVNSQ3N5SDcweEZzZnl5NEM5dnZySjhMMmVnUFNLMEpmc1JmaTdWVzN3UWpoVQpISml3cmRqa3VtZHdCOVl5NVloRHloYmJKUTYxd3BSaGlDMDFhVzVBRFg4MmhTQS93anBodFpTa3pKWndZVVV3CmJKU3NSYlpxbGVDenJPWWMvc0ozQzR1SHI3UnNXZ0UxdFJJSzBXSmdOLzVKYXlHYzlHcHQ0YjFOdGN6TW1WTm0KUWkwMVdTdGdkUW4ydW10QTl6S25aS0ltMmRyajZBdnE1SFErU0xJNHhpOEJLUWpOdHdVUXNPYWovZHpNN1h5RApibHlQcUFBeStPcXNwU2FrcTlHRW1oZXFXZz09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    rules:
      - operations: [ "CREATE" ]
        apiGroups: [""]
        apiVersions: ["v1"]
        resources: ["pods"]
    admissionReviewVersions: ["v1beta1"]
    sideEffects: None
```
```shell
k create -f webhook-configuration.yaml 

k get mutatingwebhookconfiguration
```

### [158. API Versions/Deprecations](https://uklabs.kodekloud.com/topic/lab-api-versions-deprecations-2/)

- [The Kubernetes API](https://kubernetes.io/docs/concepts/overview/kubernetes-api/)
- [API Overview](https://kubernetes.io/docs/reference/using-api/)
- [Kubernetes Deprecation Policy](https://kubernetes.io/docs/reference/using-api/deprecation-policy/)
- [Kubernetes API Concepts](https://kubernetes.io/docs/reference/using-api/api-concepts/)
- [API Overview - Enabling or disabling API groups](https://kubernetes.io/docs/reference/using-api/#enabling-or-disabling)
- [Install and Set Up kubectl on Linux - Install kubectl convert plugin](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-convert-plugin)

```shell
k api-resources | grep "deployments\|replicasets\|cronjobs\|customresourcedefinitions"
customresourcedefinitions           crd,crds     apiextensions.k8s.io/v1           false        CustomResourceDefinition
deployments                         deploy       apps/v1                           true         Deployment
replicasets                         rs           apps/v1                           true         ReplicaSet
cronjobs                            cj           batch/v1                          true         CronJob
```

```shell
k proxy
Starting to serve on 127.0.0.1:8001
```

```shell
curl http://localhost:8001/apis/authorization.k8s.io
```

```json
{
  "kind": "APIGroup",
  "apiVersion": "v1",
  "name": "authorization.k8s.io",
  "versions": [
    {
      "groupVersion": "authorization.k8s.io/v1",
      "version": "v1"
    }
  ],
  "preferredVersion": {
    "groupVersion": "authorization.k8s.io/v1",
    "version": "v1"
  }
}
```

```shell
vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

```yaml
    - --runtime-config=rbac.authorization.k8s.io/v1alpha1
```
```shell
ps aux | grep kube-apiserver
root       15716  0.0  0.0 1343644 112936 ?      Ssl  19:50   0:00 kube-apiserver --advertise-address=192.40.232.9 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction --enable-bootstrap-token-auth=true --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key --etcd-servers=https://127.0.0.1:2379 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6443 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-account-signing-key-file=/etc/kubernetes/pki/sa.key --service-cluster-ip-range=10.96.0.0/12 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key --runtime-config=rbac.authorization.k8s.io/v1alpha1
```

```shell
kubectl convert -f ingress-old.yaml --output-version networking.k8s.io/v1 > ingress-new.yaml
```

### [161. Custom Resource Definition](https://uklabs.kodekloud.com/topic/practice-test-custom-resource-definition-2/)
    - [Extend the Kubernetes API with CustomResourceDefinitions](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/)

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: internals.datasets.kodekloud.com 
spec:
  group: datasets.kodekloud.com 
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                internalLoad:
                  type: string
                range:
                  type: integer
                percentage:
                  type: string
  scope: Namespaced
  names:
    plural: internals
    singular: internal
    kind: Internal
    shortNames:
    - int
```

```shell
k create -f crd.yaml 

k api-resources | grep internals
internals                           int          datasets.kodekloud.com/v1         true         Internal
```

## Section 10: Helm Fundamentals
### [166. Install Helm](https://uklabs.kodekloud.com/topic/labs-install-helm-2/)

```shell
cat /etc/os-release 
PRETTY_NAME="Ubuntu 22.04.4 LTS"
NAME="Ubuntu"

helm --help

helm version
```

### [169. Helm Concepts](https://uklabs.kodekloud.com/topic/labs-helm-concepts-2/)
```shell
helm search hub wordpress

helm repo add bitnami https://charts.bitnami.com/bitnami

helm search repo joomla
NAME            CHART VERSION   APP VERSION     DESCRIPTION                                       
bitnami/joomla  20.0.4          5.1.2           DEPRECATED Joomla! is an award winning open sou...

helm repo list
NAME            URL                                                 
bitnami         https://charts.bitnami.com/bitnami

helm install bravo bitnami/drupal

helm list
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
bravo   default         1               2024-08-29 20:48:31.929059416 +0000 UTC deployed        drupal-20.0.1   11.0.1    

helm uninstall bravo

helm pull bitnami/apache --untar
helm install mywebapp .
```

## Section 13: Lightning Labs
### [Lightning Lab - 1](https://uklabs.kodekloud.com/topic/lightning-lab-1-4/)
### [Lightning Lab - 2](https://uklabs.kodekloud.com/topic/lightning-lab-2-2/)


## Section 14: Mock Exams
- [Mock Exam - 1](https://uklabs.kodekloud.com/topic/mock-exam-1-5/)
- [Mock Exam - 2](https://uklabs.kodekloud.com/topic/mock-exam-2-5/)
