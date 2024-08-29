1. What is not a function of admission controller?
- [ ] validate configuration
- [x] autehnticate user
- [ ] perform additional operations before the pod gets created
- [ ] help us implement better security measures

2. Which admission controller is not enabled by default?
- [ ] MutatingAdmissionWebhook
- [ ] ValidatingAdmissionWebhook
- [x] NamespaceAutoProvision
- [ ] NamespaceLifecycle

3. Which admission controller is enabled in this cluster which is normally disabled?
- [x] NodeRestriction
- [ ] DenyExecOnPrivileged
- [ ] DenyEscalatingExec
- [ ] NamespaceAutoProvision

```shell
controlplane ~ ➜  k cluster-info dump | grep admission
                            "--enable-admission-plugins=NodeRestriction",
```

4. Try to create nginx pod in the blue namespace. The blue namespace does not already exist. Dont create the blue namespace yet.

Run below command to deploy a pod with the nginx image in the blue namespace

```shell
kubectl run nginx --image nginx -n blue
```

```shell
controlplane ~ ➜  kubectl run nginx --image nginx -n blue
Error from server (NotFound): namespaces "blue" not found
```

5. The previous step failed because kubernetes have NamespaceExists admission controller enabled which rejects requests to namespaces that do not exist. So, to create a namespace that does not exist automatically, we could enable the NamespaceAutoProvision admission controller

Enable the NamespaceAutoProvision admission controller

Note: Once you update kube-apiserver yaml file, please wait for a few minutes for the kube-apiserver to restart completely.

NamespaceAutoProvision admission controller enabled on kube-apiserver?

```shell
controlplane ~ ✖ vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

```yaml
spec:
  containers:
  - command:
    - kube-apiserver
    - --enable-admission-plugins=NodeRestriction

    - --enable-admission-plugins=NodeRestriction,NamespaceAutoProvision
```

6. Now, let's run the nginx pod in blue namespace and check if it succeeds.
- Pod image: nginx
- nginx pod running in blue namespace?

```shell
controlplane ~ ➜  kubectl run nginx --image nginx -n blue
pod/nginx created
```

7. Note that the NamespaceExists and NamespaceAutoProvision admission controllers are deprecated and now replaced by NamespaceLifecycle admission controller.

The NamespaceLifecycle admission controller will make sure that requests
to a non-existent namespace is rejected and that the default namespaces such as
default, kube-system and kube-public cannot be deleted.

8. Disable DefaultStorageClass admission controller

This admission controller observes creation of PersistentVolumeClaim objects that do not request any specific storage class and automatically adds a default storage class to them. This way, users that do not request any special storage class do not need to care about them at all and they will get the default one.

Note: Once you update kube-apiserver yaml file then please wait few mins for the kube-apiserver to restart completely.


DefaultStorageClass added to disable-admission-plugins?

```shell
controlplane ~ ➜  vi /etc/kubernetes/manifests/kube-apiserver.yaml 
```

```yaml
    - --enable-admission-plugins=NodeRestriction,NamespaceAutoProvision
    - --disable-admission-plugins=DefaultStorageClass
```

9. Since the kube-apiserver is running as pod you can check the process to see enabled and disabled plugins.

```shell
ps -ef | grep kube-apiserver | grep admission-plugins
```

```shell
controlplane ~ ➜  ps -ef | grep kube-apiserver | grep admission-plugins
root       14491   14278  0 18:46 ?        00:00:03 kube-apiserver --advertise-address=192.39.26.6 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction,NamespaceAutoProvision --disable-admission-plugins=DefaultStorageClass --enable-bootstrap-token-auth=true --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key --etcd-servers=https://127.0.0.1:2379 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6443 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-account-signing-key-file=/etc/kubernetes/pki/sa.key --service-cluster-ip-range=10.96.0.0/12 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
```
