1. We have deployed two applications. Explore the setup.

Note: They are in a different namespace.

```shell
controlplane ~ ➜  k get svc -A
NAMESPACE     NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
app-space     default-http-backend   ClusterIP   10.98.89.196    <none>        80/TCP                   27s
app-space     video-service          ClusterIP   10.97.108.47    <none>        8080/TCP                 28s
app-space     wear-service           ClusterIP   10.101.173.77   <none>        8080/TCP                 28s
default       kubernetes             ClusterIP   10.96.0.1       <none>        443/TCP                  13m
kube-system   kube-dns               ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP   13m
```

2. Let us now deploy an Ingress Controller. First, create a namespace called ingress-nginx.

We will isolate all ingress related objects into its own namespace.

- Name: ingress-nginx

```shell
controlplane ~ ➜  k create ns ingress-nginx
namespace/ingress-nginx created
```

3. The NGINX Ingress Controller requires a ConfigMap object. Create a ConfigMap object with name ingress-nginx-controller in the ingress-nginx namespace.

No data needs to be configured in the ConfigMap.

- Name: ingress-nginx-controller

```shell
controlplane ~ ➜  k create cm ingress-nginx-controller -n ingress-nginx
configmap/ingress-nginx-controller created
```

4. The NGINX Ingress Controller requires two ServiceAccounts. Create both ServiceAccount with name ingress-nginx and ingress-nginx-admission in the ingress-nginx namespace.

Use the spec provided below.

- Name: ingress-nginx
- Name: ingress-nginx-admission

```shell
controlplane ~ ✖ k create sa ingress-nginx -n ingress-nginx
serviceaccount/ingress-nginx created

controlplane ~ ➜  k create sa ingress-nginx-admission -n ingress-nginx
serviceaccount/ingress-nginx-admission created
```

5. We have created the Roles, RoleBindings, ClusterRoles, and ClusterRoleBindings for the ServiceAccount. Check it out!!

```shell
controlplane ~ ➜  k get role,rolebinding,clusterrole,clusterrolebinding -A | grep ingress-nginx
```

6. Let us now deploy the Ingress Controller. Create the Kubernetes objects using the given file.

The Deployment and it's service configuration is given at /root/ingress-controller.yaml. There are several issues with it. Try to fix them.

Note: Do not edit the default image provided in the given file. The image validation check passes when other issues are resolved.

Deployed in the correct namespace.

- Replicas: 1
- Use the right image
- Namespace: ingress-nginx
- Service name: ingress-nginx-controller
- NodePort: 30080

```shell
controlplane ~ ✖ vi ingress-controller.yaml

controlplane ~ ➜  k apply -f ingress-controller.yaml
deployment.apps/ingress-nginx-controller configured
service/ingress-nginx-controller created
```

7. Create the ingress resource to make the applications available at /wear and /watch on the Ingress service.

Also, make use of rewrite-target annotation field: -

nginx.ingress.kubernetes.io/rewrite-target: /

Ingress resource comes under the namespace scoped, so don't forget to create the ingress in the app-space namespace.

Ingress Created

- Path: /wear
- Path: /watch
- Configure correct backend service for /wear
- Configure correct backend service for /watch
- Configure correct backend port for /wear service
- Configure correct backend port for /watch service

```shell
controlplane ~ ➜  k get svc -n app-space -o wide
NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE   SELECTOR
default-http-backend   ClusterIP   10.98.89.196    <none>        80/TCP     14m   app=default-backend
video-service          ClusterIP   10.97.108.47    <none>        8080/TCP   14m   app=webapp-video
wear-service           ClusterIP   10.101.173.77   <none>        8080/TCP   14m   app=webapp-wear

k create ingress -h

controlplane ~ ➜  kubectl create ingress ingress-wear-watch \
  --rule="/wear*=wear-service:8080" \
  --rule="/watch*=video-service:8080" \
  --annotation nginx.ingress.kubernetes.io/rewrite-target=/ \
  -n app-space --dry-run=client -o yaml

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

```shell
controlplane ~ ➜  kubectl create ingress ingress-wear-watch   --rule="/wear*=wear-service:8080"   --rule="/watch*=video-service:8080" --annotation nginx.ingress.kubernetes.io/rewrite-target=/ -n app-space
ingress.networking.k8s.io/ingress-wear-watch created
```

8. Access the application using the Ingress tab on top of your terminal.

Make sure you can access the right applications at /wear and /watch paths.
