1. How many Service Accounts exist in the default namespace?

2

```shell
k get sa
NAME      SECRETS   AGE
default   0         38m
dev       0         21s
```

2. What is the secret token used by the default service account?

none

```shell
k describe sa default
Name:                default
Namespace:           default
Tokens:              <none>
```

3. We just deployed the Dashboard application. Inspect the deployment. What is the image used by the deployment?

gcr.io/kodekloud/customimage/my-kubernetes-dashboard

```shell
k get deploy
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
web-dashboard   1/1     1            1           11s
```

k describe deploy web-dashboard | grep "Image:"
    Image:      gcr.io/kodekloud/customimage/my-kubernetes-dashboard

4. Wait for the deployment to be ready. Access the custom-dashboard by clicking on the link to dashboard portal.

5. What is the state of the dashboard? Have the pod details loaded successfully?

Failed

6. What type of account does the Dashboard application use to query the Kubernetes API?

Service Account

7. Which account does the Dashboard application use to query the Kubernetes API?

Default

8. Inspect the Dashboard Application POD and identify the Service Account mounted on it.

default

```shell
k describe po web-dashboard-6cbbc88b59-lwrpl | less
Service Account:  default
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-qn54c (ro)
```

9. At what location is the ServiceAccount credentials available within the pod?

/var/run/secrets

10. The application needs a ServiceAccount with the Right permissions to be created to authenticate to Kubernetes. The default ServiceAccount has limited access. Create a new ServiceAccount named dashboard-sa.
- Service Account Name: dashboard-sa

```shell
k create sa dashboard-sa
serviceaccount/dashboard-sa created
```

11. We just added additional permissions for the newly created dashboard-sa account using RBAC.

If you are interested checkout the files used to configure RBAC at /var/rbac. We will discuss RBAC in a separate section.

12. Enter the access token in the UI of the dashboard application. Click Load Dashboard button to load Dashboard

Create an authorization token for the newly created service account, copy the generated token and paste it into the token field of the UI.

To do this, run kubectl create token dashboard-sa for the dashboard-sa service account, copy the token and paste it in the UI.

```shell
kubectl create token dashboard-sa
```

13. You shouldn't have to copy and paste the token each time. The Dashboard application is programmed to read token from the secret mount location. However currently, the default service account is mounted. Update the deployment to use the newly created ServiceAccount


Edit the deployment to change ServiceAccount from default to dashboard-sa.

- Deployment name: web-dashboard
- Service Account: dashboard-sa
- Deployment Ready

[Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

```yaml
spec:
  template:
    spec:
      serviceAccountName: dashboard-sa
      containers:
```

14. Refresh the Dashboard application UI and you should now see the PODs listed automatically.

This time you shouldn't have to put in the token manually.
