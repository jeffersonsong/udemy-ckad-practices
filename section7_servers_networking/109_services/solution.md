1. How many Services exist on the system?

In the current(default) namespace

1

```shell
k get svc --no-headers | wc -l
1
```

2. That is a default service created by Kubernetes at launch.
   

3. What is the type of the default kubernetes service?

ClusterIP

```shell
k get svc -o wide
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   SELECTOR
kubernetes   ClusterIP   10.43.0.1    <none>        443/TCP   24m   <none>
```

4. What is the targetPort configured on the kubernetes service?

6443

```shell
k describe svc kubernetes 
Name:              kubernetes
Namespace:         default
Labels:            component=apiserver
                   provider=kubernetes
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.43.0.1
IPs:               10.43.0.1
Port:              https  443/TCP
TargetPort:        6443/TCP
Endpoints:         192.0.41.9:6443
Session Affinity:  None
Events:            <none>
```

5. How many labels are configured on the kubernetes service?

2

```shell
k get svc kubernetes --show-labels
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   LABELS
kubernetes   ClusterIP   10.43.0.1    <none>        443/TCP   26m   component=apiserver,provider=kubernetes
```

6. How many Endpoints are attached on the kubernetes service?

1

7. How many Deployments exist on the system now?

In the current(default) namespace

1

```shell
k get deploy --no-headers | wc -l
1
```

8. What is the image used to create the pods in the deployment?

kodekloud/simple-webapp:red

```shell
k describe deployment simple-webapp-deployment 
```

```yaml
    Image:         kodekloud/simple-webapp:red
```

9. Are you able to accesss the Web App UI?

Try to access the Web Application UI using the tab simple-webapp-ui above the terminal.

No

10. Create a new service to access the web application using the service-definition-1.yaml file.

- Name: webapp-service
- Type: NodePort
- targetPort: 8080
- port: 8080
- nodePort: 30080
- selector:
  - name: simple-webapp


- Is service created?
- Is it type nodeport?
- Is the port set?
- Is the target port set?
- Is the node port set?
- Is the selector set?

```shell
vi service-definition-1.yaml

k create -f service-definition-1.yaml 
service/webapp-service created
```

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
  namespace: default
spec:
  ports:
  - nodePort: 30080
    port: 8080
    targetPort: 8080
  selector:
    name: simple-webapp
  type: NodePort

```

11. Access the web application using the tab simple-webapp-ui above the terminal window.
12. 