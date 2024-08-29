1. Identify the short names of the deployments, replicasets, cronjobs and customresourcedefinitions.

deploy,rs,cj,crd

```shell
controlplane ~ ➜  k api-resources | grep "deployments\|replicasets\|cronjobs\|customresourcedefinitions"
customresourcedefinitions           crd,crds     apiextensions.k8s.io/v1           false        CustomResourceDefinition
deployments                         deploy       apps/v1                           true         Deployment
replicasets                         rs           apps/v1                           true         ReplicaSet
cronjobs                            cj           batch/v1                          true         CronJob
```

1. What is the patch version in the given Kubernetes API version?

Kubernetes API version - 1.22.2

2

3. Identify which API group a resource called job is part of?

batch

```shell
controlplane ~ ➜  k api-resources | grep job
cronjobs                            cj           batch/v1                          true         CronJob
jobs                                             batch/v1                          true         Job
```

4. What is the preferred version for authorization.k8s.io api group?

v1

[Kubernetes API Concepts](https://kubernetes.io/docs/reference/using-api/api-concepts/)

```shell
controlplane ~ ➜  k proxy
Starting to serve on 127.0.0.1:8001
```

```shell
controlplane ~ ✖ curl http://localhost:8001/apis/authorization.k8s.io
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

5. Enable the v1alpha1 version for rbac.authorization.k8s.io API group on the controlplane node.
Note: If you made a mistake in the config file could result in the API server being unavailable and can break the cluster.

- runtime-config option added?
- kube-apiserver pod is running?

[API Overview - Enabling or disabling API groups](https://kubernetes.io/docs/reference/using-api/#enabling-or-disabling)

```shell
controlplane ~ ➜  vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

```yaml
    - --runtime-config=rbac.authorization.k8s.io/v1alpha1
```

```shell
controlplane ~ ✖ ps aux | grep kube-apiserver
root       15716  0.0  0.0 1343644 112936 ?      Ssl  19:50   0:00 kube-apiserver --advertise-address=192.40.232.9 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction --enable-bootstrap-token-auth=true --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key --etcd-servers=https://127.0.0.1:2379 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6443 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-account-signing-key-file=/etc/kubernetes/pki/sa.key --service-cluster-ip-range=10.96.0.0/12 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key --runtime-config=rbac.authorization.k8s.io/v1alpha1
```

6. Install the kubectl convert plugin on the controlplane node.

If unsure how to install then refer to the official k8s documentation page which is available at the top right panel.
- kubectl-convert configured correctly?

[Install and Set Up kubectl on Linux - Install kubectl convert plugin](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-convert-plugin)

```shell
controlplane ~ ➜  curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/am
d64/kubectl-convert"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   138  100   138    0     0   2341      0 --:--:-- --:--:-- --:--:--  2379
100 52.5M  100 52.5M    0     0  50.3M      0  0:00:01  0:00:01 --:--:--  167M

controlplane ~ ➜  curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl-convert.sha256"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   138  100   138    0     0   2384      0 --:--:-- --:--:-- --:--:--  2421
100    64  100    64    0     0    445      0 --:--:-- --:--:-- --:--:--   445

controlplane ~ ➜  echo "$(cat kubectl-convert.sha256) kubectl-convert" | sha256sum --check
kubectl-convert: OK

controlplane ~ ➜  sudo install -o root -g root -m 0755 kubectl-convert /usr/local/bin/kubectl-convert

controlplane ~ ➜  kubectl convert --help
```

7. Ingress manifest file is already given under the /root/ directory called ingress-old.yaml.

With help of the kubectl convert command, change the deprecated API version to the networking.k8s.io/v1 and create the resource.

- Ingress Created?
- Used the new API version?

```shell
controlplane ~ ➜  kubectl convert -f ingress-old.yaml --output-version networking.k8s.io/v1 > ingress-new.yaml
controlplane ~ ➜  k create -f ingress-new.yaml 
ingress.networking.k8s.io/ingress-space created
```