1. How many StorageClasses exist in the cluster right now?

1

```shell
k api-resources | grep storageclass
storageclasses                      sc           storage.k8s.io/v1                 false        StorageClass

k get sc
NAME                   PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-path (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  7m59s
```

2. How about now? How many Storage Classes exist in the cluster?

We just created a few new Storage Classes. Inspect them.

```shell
k get sc
NAME                        PROVISIONER                     RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-path (default)        rancher.io/local-path           Delete          WaitForFirstConsumer   false                  8m47s
local-storage               kubernetes.io/no-provisioner    Delete          WaitForFirstConsumer   false                  13s
portworx-io-priority-high   kubernetes.io/portworx-volume   Delete          Immediate              false                  13s
```

3. What is the name of the Storage Class that does not support dynamic volume provisioning?
- [x] local-storage
- [ ] glusterfs-sc
- [ ] portWorx-Storageclass
- [ ] nfs-sc

```shell
k describe sc
Name:                  local-path
IsDefaultClass:        Yes
Annotations:           defaultVolumeType=local,objectset.rio.cattle.io/applied=H4sIAAAAAAAA/4yRz47UMAyHXwX53JYpnamqSBxg0V4QEhJoObuJOzVN4ypxi0areXeUMqDhwJ9j8ov9xZ+fARd+ophYAhhIKhHPVE1dqlhebjUUMHFwYODTj+jBY0pQwEyKDhXBPAOGIIrKElI+Ohpw9fokfp3p82UhMODFoocCpP9KVhNpFVkqi6qeMokz4i+5fAsUy/M2gYGpSXfJVhcv3nNwr984J+GfLQLOv/5T3sb9r6K0oM2V09pTmS5JaYbipzCbrVQ5ioGUdnmcypuJco/BgMaV4FqAx5787upP3BHTCAbqrhmak21Pw9Db5tAe20MzHJuhPnUH19m2w1cOe3fMTX+bbEEd8+USZeO8XIpgIGKwI8UMuHtWQMwD8PxRPNsLGHhHnjRr2fYdvuXgOJw/iMuAL8j6KPGRY9IHCWmdKcL1ewAAAP//KQ1Ko0kCAAA,objectset.rio.cattle.io/id=,objectset.rio.cattle.io/owner-gvk=k3s.cattle.io/v1, Kind=Addon,objectset.rio.cattle.io/owner-name=local-storage,objectset.rio.cattle.io/owner-namespace=kube-system,storageclass.kubernetes.io/is-default-class=true
Provisioner:           rancher.io/local-path
Parameters:            <none>
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     WaitForFirstConsumer
Events:                <none>


Name:            local-storage
IsDefaultClass:  No
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{},"name":"local-storage"},"provisioner":"kubernetes.io/no-provisioner","volumeBindingMode":"WaitForFirstConsumer"}

Provisioner:           kubernetes.io/no-provisioner
Parameters:            <none>
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     WaitForFirstConsumer
Events:                <none>


Name:            portworx-io-priority-high
IsDefaultClass:  No
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{},"name":"portworx-io-priority-high"},"parameters":{"priority_io":"high","repl":"1","snap_interval":"70"},"provisioner":"kubernetes.io/portworx-volume"}

Provisioner:           kubernetes.io/portworx-volume
Parameters:            priority_io=high,repl=1,snap_interval=70
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     Immediate
Events:                <none>
```

4. What is the Volume Binding Mode used for this storage class (the one identified in the previous question)?
- [x] WaitForFirstConsumer
- [ ] Immediate

```shell
k get sc local-storage 
NAME            PROVISIONER                    RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-storage   kubernetes.io/no-provisioner   Delete          WaitForFirstConsumer   false                  4m1s
```

5. What is the Provisioner used for the storage class called portworx-io-priority-high?
- [ ] local-volume
- [x] portworx-volume
- [ ] no-provisioner
- [ ] ceph-volue

```shell
k get sc portworx-io-priority-high
NAME                        PROVISIONER                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
portworx-io-priority-high   kubernetes.io/portworx-volume   Delete          Immediate           false                  5m42s
```

6. Is there a PersistentVolumeClaim that is consuming the PersistentVolume called local-pv?
- [x] NO
- [ ] YES

```shell
k get pvc
No resources found in default namespace.
```

7. Let's fix that. Create a new PersistentVolumeClaim by the name of local-pvc that should bind to the volume local-pv.

Inspect the pv local-pv for the specs.


- PVC: local-pvc
- Correct Access Mode?
- Correct StorageClass Used?
- PVC requests volume size = 500Mi?

[Configure a Pod to Use a PersistentVolume for Storage - Create a PersistentVolumeClaim](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolumeclaim)

```shell
k get pv local-pv
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS    VOLUMEATTRIBUTESCLASS   REASON   AGE
local-pv   500Mi      RWO            Retain           Available           local-storage   <unset>                          8m44s

vi local-pvc.yaml

k create -f local-pvc.yaml 
persistentvolumeclaim/local-pvc created

k get pvc
NAME        STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS    VOLUMEATTRIBUTESCLASS   AGE
local-pvc   Pending                                      local-storage   <unset>                 6s
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

8. What is the status of the newly created Persistent Volume Claim?
Pending

9. Why is the PVC in a pending state despite making a valid request to claim the volume called local-pv?
- [ ] PVC Specs are incorrect
- [ ] Storage Class not found
- [x] A Pod consuimg the volume is not scheduled
- [ ] Persistent Volume and Claim mismatch

Inspect the PVC events.

```shell
k describe pvc local-pvc 
Name:          local-pvc
Namespace:     default
StorageClass:  local-storage
Status:        Pending
Volume:        
Labels:        <none>
Annotations:   <none>
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      
Access Modes:  
VolumeMode:    Filesystem
Used By:       <none>
Events:
  Type    Reason                Age                From                         Message
  ----    ------                ----               ----                         -------
  Normal  WaitForFirstConsumer  11s (x2 over 16s)  persistentvolume-controller  waiting for first consumer to be created before binding
```

10. The Storage Class called local-storage makes use of VolumeBindingMode set to WaitForFirstConsumer. This will delay the binding and provisioning of a PersistentVolume until a Pod using the PersistentVolumeClaim is created.

11. Create a new pod called nginx with the image nginx:alpine. The Pod should make use of the PVC local-pvc and mount the volume at the path /var/www/html.

- The PV local-pv should be in a bound state.
- Pod created with the correct Image?
- Pod uses PVC called local-pvc?
- local-pv bound?
- nginx pod running?
- Volume mounted at the correct path?

[Configure a Pod to Use a PersistentVolume for Storage - Create a Pod](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-pod)

```shell
k run nginx --image=nginx:alpine --dry-run=client -o yaml > nginx.yaml
vi nginx.yaml 

k create -f nginx.yaml 
pod/nginx created
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  volumes:
  - name: pv-volume
    persistentVolumeClaim:
     claimName: local-pvc
  containers:
  - image: nginx:alpine
    name: nginx
    volumeMounts:
    - mountPath: "/var/www/html"
      name: pv-volume
```

12.  What is the status of the local-pvc Persistent Volume Claim now?
Bound

```shell
k get pvc local-pvc 
NAME        STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS    VOLUMEATTRIBUTESCLASS   AGE
local-pvc   Bound    local-pv   500Mi      RWO            local-storage   <unset>                 11m
```

13. Create a new Storage Class called delayed-volume-sc that makes use of the below specs:
- provisioner: kubernetes.io/no-provisioner
- volumeBindingMode: WaitForFirstConsumer
- Storage Class uses: kubernetes.io/no-provisioner ?
- Storage Class volumeBindingMode set to WaitForFirstConsumer ?

[Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/)

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: delayed-volume-sc
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Retain # default value is Delete
volumeBindingMode: WaitForFirstConsumer
```

```shell
vi delayed-volume-sc.yaml

k create -f delayed-volume-sc.yaml 
storageclass.storage.k8s.io/delayed-volume-sc created

k get sc delayed-volume-sc 
NAME                PROVISIONER                    RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
delayed-volume-sc   kubernetes.io/no-provisioner   Retain          WaitForFirstConsumer   false                  7s
```