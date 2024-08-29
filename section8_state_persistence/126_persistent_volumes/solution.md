References:
- [Configure a Pod to Use a PersistentVolume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)
- [Volumes - hostPath configuration example](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath-configuration-example)


1. We have deployed a POD. Inspect the POD and wait for it to start running.
In the current(default) namespace.

```shell
k get po
NAME     READY   STATUS    RESTARTS   AGE
webapp   1/1     Running   0          8s
```

2. The application stores logs at location /log/app.log. View the logs.

You can exec in to the container and open the file:

```shell
kubectl exec webapp -- cat /log/app.log
```

3. If the POD was to get deleted now, would you be able to view these logs.
- [ ] YES
- [x] NO

4. Configure a volume to store these logs at /var/log/webapp on the host.

Use the spec provided below.
- Name: webapp
- Image Name: kodekloud/event-simulator
- Volume HostPath: /var/log/webapp
- Volume Mount: /log

[Volumes - hostPath configuration example](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath-configuration-example)

```shell
k edit po webapp
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - image: kodekloud/event-simulator
    name: event-simulator
    ...
    volumeMounts:
    ...
    - mountPath: /log
      name: log-volume
  ...
  volumes:
  ...
  - name: log-volume
    hostPath:
      path: /var/log/webapp
```

```shell
k replace --force -f /tmp/kubectl-edit-569298634.yaml
```

5. Create a Persistent Volume with the given specification.
- Volume Name: pv-log
- Storage: 100Mi
- Access Modes: ReadWriteMany
- Host Path: /pv/log
- Reclaim Policy: Retain

[Configure a Pod to Use a PersistentVolume for Storage - Create a PersistentVolume](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume)

```shell
vi pv-log.yaml

k apply -f pv-log.yaml 
persistentvolume/pv-log created
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

6. Let us claim some of that storage for our application. Create a Persistent Volume Claim with the given specification.

- Volume Name: claim-log-1
- Storage Request: 50Mi
- Access Modes: ReadWriteOnce

[Configure a Pod to Use a PersistentVolume for Storage - Create a PersistentVolumeClaim](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolumeclaim)

```shell
vi claim-log-1.yaml 

k create -f claim-log-1.yaml 
persistentvolumeclaim/claim-log-1 created
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  volumeName: pv-log
  resources:
    requests:
      storage: 50Mi
```

1. What is the state of the Persistent Volume Claim?

- [ ] Available
- [ ] Bound
- [x] Pending
- [ ] Created

```shell
k get pvc
NAME          STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
claim-log-1   Pending   pv-log   0                                        <unset>                 7s
```

8. What is the state of the Persistent Volume?
- [ ] Pending
- [x] Available
- [ ] Created
- [ ] Bound

```shell
k get pv
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM                 STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pv-log   100Mi      RWX            Retain           Released   default/claim-log-1                  <unset>   
```

9. Why is the claim not bound to the available Persistent Volume?
- [x] Access Modes Mismatch
- [ ] Capacity Mismatch
- [ ] PV and PVC name mismatch
- [ ] Relaim Policy not set correctly
  
10.   Update the Access Mode on the claim to bind it to the PV.


Delete and recreate the claim-log-1.

- Volume Name: claim-log-1
- Storage Request: 50Mi
- PVol: pv-log
- Status: Bound

```
controlplane ~ ✖ vi claim-log-1.yaml 

k replace --force -f claim-log-1.yaml 
persistentvolumeclaim/claim-log-1 replaced
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

11. You requested for 50Mi, how much capacity is now available to the PVC?
- [ ] 1Gi
- [x] 100Mi
- [ ] 50Mi
- [ ] 0

```shell
k get pvc
NAME          STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
claim-log-1   Bound    pv-log   100Mi      RWX                           <unset>                 3s
```

12. Update the webapp pod to use the persistent volume claim as its storage.

Replace hostPath configured earlier with the newly created PersistentVolumeClaim.

- Name: webapp
- Image Name: kodekloud/event-simulator
- Volume: PersistentVolumeClaim=claim-log-1
- Volume Mount: /log

[Configure a Pod to Use a PersistentVolume for Storage - Create a Pod](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-pod)


```yaml
  - name: log-volume
    persistentVolumeClaim:
      claimName: claim-log-1
```

```shell
k edit po webapp 
k replace --force -f /tmp/kubectl-edit-2253563637.yaml
pod "webapp" deleted
pod/webapp replaced
```

13. What is the Reclaim Policy set on the Persistent Volume pv-log?
- [x] Retain
- [ ] Recycle
- [ ] Delete
- [ ] Scrub

```shell
k get pv
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pv-log   100Mi      RWX            Retain           Bound    default/claim-log-1                  <unset>                          5m57s
```

14. What would happen to the PV if the PVC was destroyed?

- [ ] The PV is made available again
- [ ] The PV os scrubbed
- [x] The PV is not deleted but not available
- [ ] The PV is deleted as well


15. Try deleting the PVC and notice what happens.

If the command hangs, you can use CTRL + C to get back to the bash prompt OR check the status of the pvc from another terminal
- [x] The PVC is stuck in 'terminating' state
- [ ] The PV is deleted.

```shell
k delete pvc claim-log-1 
persistentvolumeclaim "claim-log-1" deleted

k get pvc
NAME          STATUS        VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
claim-log-1   Terminating   pv-log   100Mi      RWX                           <unset>                 7m55s
```

16.  Why is the PVC stuck in Terminating state?
- [ ] The PVC is in the process of scrubbing
- [x] The PVC is being used by a POD
- [ ] The PVC is waiting for the PV to be deleted.

17. Let us now delete the webapp Pod.

Once deleted, wait for the pod to fully terminate.

- Name: webapp

```shell
k delete po webapp
pod "webapp" deleted
```

18.  What is the state of the PVC now?
- [x] Deleted
- [ ] pending
- [ ] Bound
- [ ] Available

19.  What is the state of the Persistent Volume now?
- [ ] Available
- [ ] Pending
- [x] Released
- [ ] Deleted
- [ ] Bound

```shell
k get pv
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM                 STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pv-log   100Mi      RWX            Retain           Released   default/claim-log-1                  <unset>   
```
