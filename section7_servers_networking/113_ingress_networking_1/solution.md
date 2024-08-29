1. We have deployed Ingress Controller, resources and applications. Explore the setup.

Note: They are in different namespaces.

```shell
k get all -A
```

2. Which namespace is the Ingress Controller deployed in?

ingress-nginx

3. What is the name of the Ingress Controller Deployment?

ingress-nginx-controller

4. Which namespace are the applications deployed in?
app-space

```shell
k get svc -A
```

5. How many applications are deployed in the app-space namespace?

Count the number of deployments in this namespace.

3

6. Which namespace is the Ingress Resource deployed in?
app-space

```shell
k get ingress -A
NAMESPACE   NAME                 CLASS    HOSTS   ADDRESS          PORTS   AGE
app-space   ingress-wear-watch   <none>   *       10.105.147.181   80      3m28s
```

7. What is the name of the Ingress Resource?
   
ingress-wear-watch

8. What is the Host configured on the Ingress Resource?

The host entry defines the domain name that users use to reach the application like www.google.com

All hosts(*)

```shell
k describe ingress ingress-wear-watch -n app-space

Name:             ingress-wear-watch
Labels:           <none>
Namespace:        app-space
Address:          10.105.147.181
Ingress Class:    <none>
Default backend:  <default>
Rules:
  Host        Path  Backends
  ----        ----  --------
  *           
              /wear    wear-service:8080 (10.244.0.4:8080)
              /watch   video-service:8080 (10.244.0.5:8080)
Annotations:  nginx.ingress.kubernetes.io/rewrite-target: /
              nginx.ingress.kubernetes.io/ssl-redirect: false
Events:
  Type    Reason  Age                    From                      Message
  ----    ------  ----                   ----                      -------
  Normal  Sync    4m15s (x2 over 4m15s)  nginx-ingress-controller  Scheduled for sync
```

9. What backend is the /wear path on the Ingress configured with?

wear-service

10. At what path is the video streaming application made available on the Ingress?

/watch

11. If the requirement does not match any of the configured paths in the Ingress, to which service are the requests forwarded?

default-backend-service

12. Now view the Ingress Service using the tab at the top of the terminal. Which page do you see?

Click on the tab named Ingress.

404 Error Page

13. View the applications by appending /wear and /watch to the URL you opened in the previous step.

14. You are requested to change the URLs at which the applications are made available.

Make the video application available at /stream.
- Ingress: ingress-wear-watch
- Path: /stream
- Backend Service: video-service
- Backend Service Port: 8080

```shell
k edit ingress ingress-wear-watch -n app-space
ingress.networking.k8s.io/ingress-wear-watch edited
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
  creationTimestamp: "2024-08-29T23:22:09Z"
  generation: 1
  name: ingress-wear-watch
  namespace: app-space
  resourceVersion: "1701"
  uid: eade9753-1cbc-4a9f-9218-837b97aae0ad
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
  loadBalancer:
    ingress:
    - ip: 10.105.147.181
```

```yaml
      - backend:
          service:
            name: video-service
            port:
              number: 8080
        path: /stream
        pathType: Prefix
```

15. View the Video application using the /stream URL in your browser.

Click on the Ingress tab above your terminal, if its not open already, and append /stream to the URL in the browser.

16. A user is trying to view the /eat URL on the Ingress Service. Which page would he see?

If not open already, click on the Ingress tab above your terminal, and append /eat to the URL in the browser.

404 Error Page


17. Due to increased demand, your business decides to take on a new venture. You acquired a food delivery company. Their applications have been migrated over to your cluster.

Inspect the new deployments in the app-space namespace.

```shell
k get svc -n app-space
NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
default-backend-service   ClusterIP   10.98.164.161   <none>        80/TCP     10m
food-service              ClusterIP   10.109.50.90    <none>        8080/TCP   17s
video-service             ClusterIP   10.102.21.62    <none>        8080/TCP   10m
wear-service              ClusterIP   10.96.61.112    <none>        8080/TCP   10m
```

18. You are requested to add a new path to your ingress to make the food delivery application available to your customers.


Make the new application available at /eat.

- Ingress: ingress-wear-watch
- Path: /eat
- Backend Service: food-service
- Backend Service Port: 8080

```shell
k edit ingress ingress-wear-watch -n app-space
ingress.networking.k8s.io/ingress-wear-watch edited
```
```yaml
      - backend:
          service:
            name: food-service
            port:
              number: 8080
        path: /eat
        pathType: Prefix
```

19. View the Food delivery application using the /eat URL in your browser.

Click on the Ingress tab above your terminal, if its not open already, and append /eat to the URL in the browser.

20. A new payment service has been introduced. Since it is critical, the new application is deployed in its own namespace.

Identify the namespace in which the new application is deployed.

```shell
k get svc -A
NAMESPACE        NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
critical-space   pay-service                          ClusterIP   10.108.254.139   <none>        8282/TCP                     17s
```

21. What is the name of the deployment of the new application?

webapp-pay

```shell
k get deploy -n critical-space
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
webapp-pay   1/1     1            1           74s
```

22. You are requested to make the new application available at /pay.

Identify and implement the best approach to making this application available on the ingress controller and test to make sure its working. Look into annotations: rewrite-target as well.

- Ingress Created
- Path: /pay
- Configure correct backend service
- Configure correct backend port

```shell
k create ingress -h | less

controlplane ~ ✖ kubectl create ingress ingress-pay-service -n critical-space --rule="/pay=pay-service:8282"
ingress.networking.k8s.io/ingress-pay-service created
````

23. View the Payment application using the /pay URL in your browser.

Click on the Ingress tab above your terminal, if its not open already, and append /pay to the URL in the browser.
