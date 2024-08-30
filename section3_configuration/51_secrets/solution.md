1. How many Secrets exist on the system?

In the current(default) namespace.

1

```shell
k get secret
NAME              TYPE                                  DATA   AGE
dashboard-token   kubernetes.io/service-account-token   3      16s
```

2. How many secrets are defined in the dashboard-token secret?

```shell
k describe secret dashboard-token 
Name:         dashboard-token
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: dashboard-sa
              kubernetes.io/service-account.uid: d754d4c4-bc9b-4ff3-b2b1-ce3b8e246b2a

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     570 bytes
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IjRLZHNlelg0SGJOLVZDZndLbS1WVmpQWkR1M0JSTWFETkhVdm1CSndvUHcifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRhc2hib2FyZC10b2tlbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJkYXNoYm9hcmQtc2EiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJkNzU0ZDRjNC1iYzliLTRmZjMtYjJiMS1jZTNiOGUyNDZiMmEiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpkYXNoYm9hcmQtc2EifQ.X7caBUTxO71p4yF-J4F-3XtpUlQue3vOB6RAjgvT2lYTfY4lqX0KYAA0Vh8Yxuq5ZLpfLYOrrxoeh0CRX1A3x6lzUnakFL_jjncl4otWCp2ceIyO_EYLP1LXZgww9OFF2ooB7B9IhCyv3I5yUqWsBT_RegSAf3FeAKdI3OXPz8FD9LBdt8nlbiybYLkql1Vw_Af1p5buuOr7CzC5fTEYsZDAxbj-VQndzOLRKDiiCAtkO70sKbRrhhAvDYEvDughAnSg6aPskTYiYhs3xyVgYBHrclyKEw9SCRkWO8lfOvTAkRw8TTOEdK21sYaVnO_xA3VIVkNkO8drzlEp7u2qCg
```

3. What is the type of the dashboard-token secret?

kubernetes.io/service-account-token

4. Which of the following is not a secret data defined in dashboard-token secret?

type

5. We are going to deploy an application with the below architecture

We have already deployed the required pods and services. Check out the pods and services created. Check out the web application using the Webapp MySQL link above your terminal, next to the Quiz Portal Link.

```shell
k get po,svc
NAME             READY   STATUS    RESTARTS   AGE
pod/webapp-pod   1/1     Running   0          36s
pod/mysql        1/1     Running   0          36s

NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/kubernetes       ClusterIP   10.43.0.1       <none>        443/TCP          18m
service/webapp-service   NodePort    10.43.227.47    <none>        8080:30080/TCP   36s
service/sql01            ClusterIP   10.43.150.172   <none>        3306/TCP         36s
```

6. The reason the application is failed is because we have not created the secrets yet. Create a new secret named db-secret with the data given below.

You may follow any one of the methods discussed in lecture to create the secret.

- Secret Name: db-secret
- Secret 1: DB_Host=sql01
- Secret 2: DB_User=root
- Secret 3: DB_Password=password123

```shell
k create secret generic -h

kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123
secret/db-secret created
```

7. Configure webapp-pod to load environment variables from the newly created secret.

Delete and recreate the pod if required.

- Pod name: webapp-pod
- Image name: kodekloud/simple-webapp-mysql
- Env From: Secret=db-secret
  
[Configure all key-value pairs in a Secret as container environment variables](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#configure-all-key-value-pairs-in-a-secret-as-container-environment-variables)

```shell
k edit po webapp-pod 
```

```yaml
spec:
  containers:
  - image: kodekloud/simple-webapp-mysql
    imagePullPolicy: Always
    name: webapp
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-qqt2j
      readOnly: true
    envFrom:
    - secretRef:
        name: db-secret
```

```shell
replace --force -f /tmp/kubectl-edit-3758888637.yaml
pod "webapp-pod" deleted
pod/webapp-pod replaced
```

8. View the web application to verify it can successfully connect to the database
