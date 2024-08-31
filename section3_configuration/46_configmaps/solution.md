1. How many PODs exist on the system?
in the current(default) namespace

1

```shell
k get po
NAME           READY   STATUS    RESTARTS   AGE
webapp-color   1/1     Running   0          25s
```

2. What is the environment variable name set on the container in the pod?

APP_COLOR

```shell
k describe po webapp-color 
    Environment:
      APP_COLOR:  pink
```

3. What is the value set on the environment variable APP_COLOR on the container in the pod?

pink

4. View the web application UI by clicking on the Webapp Color Tab above your terminal.

This is located on the right side.

5. Update the environment variable on the POD to display a green background.
Note: Delete and recreate the POD. Only make the necessary changes. Do not modify the name of the Pod.

- Pod Name: webapp-color
- Label Name: webapp-color
- Env: APP_COLOR=green

```shell
k edit po webapp-color 
```

```yaml
  containers:
  - env:
    - name: APP_COLOR
      value: green
```

k replace --force -f /tmp/kubectl-edit-1557678233.yaml

6. View the changes to the web application UI by clicking on the Webapp Color Tab above your terminal.

If you already have it open, simply refresh the browser.

7. How many ConfigMaps exists in the default namespace?

2

```shell
k get cm
NAME               DATA   AGE
kube-root-ca.crt   1      12m
db-config          3      10s
```

8. Identify the database host from the config map db-config.

SQL01.example.com

```shell
k describe cm db-config
Name:         db-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
DB_HOST:
----
SQL01.example.com
DB_NAME:
----
SQL01
DB_PORT:
----
3306

BinaryData
====

Events:  <none>
```

9. Create a new ConfigMap for the webapp-color POD. Use the spec given below.
- ConfigMap Name: webapp-config-map
- Data: APP_COLOR=darkblue
- Data: APP_OTHER=disregard

```shell
kubectl create configmap webapp-config-map --from-literal=APP_COLOR=darkblue --from-literal=APP_OTHER=disregard
configmap/webapp-config-map created
```

10. Update the environment variable on the POD to use only the APP_COLOR key from the newly created ConfigMap.
Note: Delete and recreate the POD. Only make the necessary changes. Do not modify the name of the Pod.

- Pod Name: webapp-color
- ConfigMap Name: webapp-config-map

[Configure a Pod to Use a ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)

```shell
k edit po webapp-color
```

```yaml
  - env:
    - name: APP_COLOR
      value: green
```

```yaml
  - env:
    - name: APP_COLOR
      valueFrom:
        configMapKeyRef:
          name: webapp-config-map
          key: APP_COLOR
```

```shell
k replace --force -f /tmp/kubectl-edit-1711318721.yaml
pod "webapp-color" deleted
pod/webapp-color replaced
```

11. View the changes to the web application UI by clicking on the Webapp Color Tab above your terminal.

If you already have it open, simply refresh the browser.
