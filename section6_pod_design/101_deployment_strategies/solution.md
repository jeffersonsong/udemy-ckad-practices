1. A deployment has been created in the default namespace. What is the deployment strategy used for this deployment?

Rolling Update

```shell
k get deploy
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
frontend   5/5     5            5           47s

k describe deploy frontend | less
```

2. The deployment called frontend app is exposed on the NodePort via a service.

Identify the name of this service.

frontend-service

```shell
k describe deploy frontend | less
Pod Template:
  Labels:  app=frontend
           version=v1

k get svc -o wide
NAME               TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE     SELECTOR
frontend-service   NodePort    10.103.66.51   <none>        8080:30080/TCP   2m25s   app=frontend
kubernetes         ClusterIP   10.96.0.1      <none>        443/TCP          4m9s    <none>
```

3. What is the selector used by this service?

Inspect the service called frontend-service.

app: frontend

4. Check out the web application using the Webapp link above your terminal.

5. A new deployment called frontend-v2 has been created in the default namespace using the image kodekloud/webapp-color:v2. This deployment will be used to test a newer version of the same app.

Configure the deployment in such a way that the service called frontend-service routes less than 20% of traffic to the new deployment.
Do not increase the replicas of the frontend deployment.

- Task Completed?

```shell
k get deploy -o wide
NAME          READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS     IMAGES                      SELECTOR
frontend      5/5     5            5           4m42s   webapp-color   kodekloud/webapp-color:v1   app=frontend
frontend-v2   2/2     2            2           67s     webapp-color   kodekloud/webapp-color:v2   app=frontend

k scale deploy frontend-v2 --replicas=1
deployment.apps/frontend-v2 scaled

k get deploy -o wide
NAME          READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS     IMAGES                      SELECTOR
frontend      5/5     5            5           5m25s   webapp-color   kodekloud/webapp-color:v1   app=frontend
frontend-v2   1/1     1            1           110s    webapp-color   kodekloud/webapp-color:v2   app=frontend
```

6. Check out the new version of the web application using the Webapp link above your terminal.

You may need to refresh it multiple times to see the new version.

7. We have now established that the new version v2 of the application is working as expected.

We can now safely redirect all users to the v2 version.

8. Scale down the v1 version of the apps to 0 replicas and scale up the new(v2) version to 5 replicas.

- v1 scaled down to 0 pods?
- v2 scaled up to 5 pods?

```shell
k scale deploy frontend --replicas=0
deployment.apps/frontend scaled

k scale deploy frontend-v2 --replicas=5
deployment.apps/frontend-v2 scaled
```

9. Now delete the deployment called frontend completely.

- Task completed?

```shell
k delete deploy frontend
deployment.apps "frontend" deleted

k get deploy
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
frontend-v2   5/5     5            5           3m43s
```

10. You can now reload the Webapp tab to validate that all user traffic now uses the v2 version of the app.

