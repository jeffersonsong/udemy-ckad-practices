1. We have deployed a few PODs running workloads. Inspect them.

Wait for the pods to be ready before proceeding to the next question.

```shell
k get po
NAME       READY   STATUS    RESTARTS   AGE
elephant   1/1     Running   0          42s
lion       1/1     Running   0          42s
rabbit     1/1     Running   0          42s
```

2. Let us deploy metrics-server to monitor the PODs and Nodes. Pull the git repository for the deployment files.

Run: git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git

```shell
git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git
```

3. Deploy the metrics-server by creating all the components downloaded.

Run the kubectl create -f . command from within the downloaded repository.

```shell
cd kubernetes-metrics-server/

controlplane kubernetes-metrics-server on  master ➜  kubectl create -f .
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
serviceaccount/metrics-server created
deployment.apps/metrics-server created
service/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
```

4. It takes a few minutes for the metrics server to start gathering data.

Run the kubectl top node command and wait for a valid output.

```shell
controlplane kubernetes-metrics-server on  master ➜  k get svc,deploy -n kube-system
NAME                     TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns         ClusterIP   10.96.0.10     <none>        53/UDP,53/TCP,9153/TCP   9m17s
service/metrics-server   ClusterIP   10.96.133.50   <none>        443/TCP                  83s

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns          2/2     2            2           9m17s
deployment.apps/metrics-server   1/1     1            1           83s

controlplane kubernetes-metrics-server on  master ➜  k top node
NAME           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
controlplane   270m         0%     1126Mi          0%        
node01         23m          0%     303Mi           0%   
```

5. Identify the node that consumes the most CPU(cores).
   
controlplane

6. Identify the node that consumes the most Memory(bytes).

controlplane

7. Identify the POD that consumes the most Memory(bytes) in default namespace.
rabbit

```shell
controlplane kubernetes-metrics-server on  master ➜  k top pod
NAME       CPU(cores)   MEMORY(bytes)   
elephant   14m          32Mi            
lion       1m           18Mi            
rabbit     107m         252Mi      
```

8. Identify the POD that consumes the least CPU(cores) in default namespace.

lion
