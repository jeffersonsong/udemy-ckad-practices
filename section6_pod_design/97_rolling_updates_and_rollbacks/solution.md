1. We have deployed a simple web application. Inspect the PODs and the Services

Wait for the application to fully deploy and view the application using the link called Webapp Portal above your terminal.

```shell
k get po,svc
NAME                           READY   STATUS    RESTARTS   AGE
pod/frontend-f8d68f98f-nzrhz   1/1     Running   0          16s
pod/frontend-f8d68f98f-98mcc   1/1     Running   0          16s
pod/frontend-f8d68f98f-fp6qj   1/1     Running   0          16s
pod/frontend-f8d68f98f-zjkvr   1/1     Running   0          16s

NAME                     TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/kubernetes       ClusterIP   10.43.0.1      <none>        443/TCP          15m
service/webapp-service   NodePort    10.43.134.34   <none>        8080:30080/TCP   16s
```

2. What is the current color of the web application?

Access the Webapp Portal.

blue

3. Run the script named curl-test.sh to send multiple requests to test the web application. Take a note of the output.

Execute the script at /root/curl-test.sh.

```shell
./curl-test.sh 
```

4. Inspect the deployment and identify the number of PODs deployed by it

4

```shell
k get deploy
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
frontend   4/4     4            4           92s
```

5. What container image is used to deploy the applications?

kodekloud/webapp-color:v1

```shell
k describe deploy frontend | grep Image
    Image:         kodekloud/webapp-color:v1
```

6. Inspect the deployment and identify the current strategy

RollingUpdate

```shell
k describe deploy frontend | grep StrategyType
StrategyType:           RollingUpdate
```

7. If you were to upgrade the application now what would happen?
PODs are upgraded few at a time.

8. Let us try that. Upgrade the application by setting the image on the deployment to kodekloud/webapp-color:v2

Do not delete and re-create the deployment. Only set the new image name for the existing deployment.

- Deployment Name: frontend
- Deployment Image: kodekloud/webapp-color:v2

```shell
k describe deploy frontend 
Name:                   frontend
Namespace:              default
CreationTimestamp:      Fri, 30 Aug 2024 02:31:53 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               name=webapp
Replicas:               4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        20
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  name=webapp
  Containers:
   simple-webapp:
    Image:         kodekloud/webapp-color:v1
    Port:          8080/TCP
    Host Port:     0/TCP
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   frontend-f8d68f98f (4/4 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  3m48s  deployment-controller  Scaled up replica set frontend-f8d68f98f to 4

k set image -h

kubectl set image deployment/frontend simple-webapp=kodekloud/webapp-color:v2
deployment.apps/frontend image updated
```

9. Run the script curl-test.sh again. Notice the requests now hit both the old and newer versions. However none of them fail.

Execute the script at /root/curl-test.sh.

10. Up to how many PODs can be down for upgrade at a time

Consider the current strategy settings and number of PODs - 4

1

```shell
k describe deploy frontend

Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  6m11s  deployment-controller  Scaled up replica set frontend-f8d68f98f to 4
  Normal  ScalingReplicaSet  60s    deployment-controller  Scaled up replica set frontend-69b69fcc6d to 1
  Normal  ScalingReplicaSet  60s    deployment-controller  Scaled down replica set frontend-f8d68f98f to 3 from 4
  Normal  ScalingReplicaSet  60s    deployment-controller  Scaled up replica set frontend-69b69fcc6d to 2 from 1
  Normal  ScalingReplicaSet  37s    deployment-controller  Scaled down replica set frontend-f8d68f98f to 1 from 3
  Normal  ScalingReplicaSet  37s    deployment-controller  Scaled up replica set frontend-69b69fcc6d to 4 from 2
  Normal  ScalingReplicaSet  14s    deployment-controller  Scaled down replica set frontend-f8d68f98f to 0 from 1
```

11. Change the deployment strategy to Recreate

Delete and re-create the deployment if necessary. Only update the strategy type for the existing deployment.

- Deployment Name: frontend
- Deployment Image: kodekloud/webapp-color:v2
- Strategy: Recreate

```shell
k edit deploy frontend
deployment.apps/frontend edited
```
```yaml
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate

  strategy:
    type: Recreate
```

12. Upgrade the application by setting the image on the deployment to kodekloud/webapp-color:v3

Do not delete and re-create the deployment. Only set the new image name for the existing deployment.

- Deployment Name: frontend
- Deployment Image: kodekloud/webapp-color:v3

```shell
kubectl set image deployment/frontend simple-webapp=kodekloud/webapp-color:v3
deployment.apps/frontend image updated
```

13. Run the script curl-test.sh again. Notice the failures. Wait for the new application to be ready. Notice that the requests now do not hit both the versions

Execute the script at /root/curl-test.sh.
