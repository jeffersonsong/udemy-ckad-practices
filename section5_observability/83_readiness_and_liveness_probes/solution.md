1. We have deployed a simple web application. Inspect the PODs and Services.

```shell
k get po,svc
NAME                  READY   STATUS    RESTARTS   AGE
pod/simple-webapp-1   1/1     Running   0          11s

NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/kubernetes       ClusterIP   10.96.0.1       <none>        443/TCP          2m56s
service/webapp-service   NodePort    10.99.132.193   <none>        8080:30080/TCP   11s
```

2. View the application by clicking on the 'Web Portal' link above your terminal.

It may take a minute or two for the web app to come up. Monitor the status of the POD until it is ready.

3. A test script is provided that sends multiple requests to access the web application.

Execute the script at /root/curl-test.sh. Run the command './curl-test.sh'

4. We just created a new POD to scale the application. View the pods. Run the script again and view the results.

The application takes 80 seconds to warm up.

5. Notice the errors in the output of the script.

```shell
./curl-test.sh 
Message from simple-webapp-1 : I am ready! OK

Failed
```

6. Update the newly created pod 'simple-webapp-2' with a readinessProbe using the given spec


Spec is given on the below. Do not modify any other properties of the pod.

- Pod Name: simple-webapp-2
- Image Name: kodekloud/webapp-delayed-start
- Readiness Probe: httpGet
- Http Probe: /ready
- Http Port: 8080

[Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

```yaml
spec:
  containers:
  - name: simple-webapp
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
```

7. Run the script again to test the web application. Notice that none of the requests fail now. Though all the requests hit the same POD.

Execute the script at /root/curl-test.sh.

```shell
./curl-test.sh 
Message from simple-webapp-1 : I am ready! OK

Message from simple-webapp-1 : I am ready! OK

Message from simple-webapp-1 : I am ready! OK
```

8. Wait for the new POD to be ready and run the test again to see traffic distributed between both the PODs.

Execute the script at /root/curl-test.sh.
Once the simple-webapp-2 pod is ready, the traffic will be distributed to it as well.

```shell
k get po -w
NAME              READY   STATUS    RESTARTS   AGE
simple-webapp-1   1/1     Running   0          12m
simple-webapp-2   0/1     Running   0          86s

./curl-test.sh 
Message from simple-webapp-1 : I am ready! OK

Message from simple-webapp-2 : I am ready! OK
```

9. What would happen if the application inside container on one of the PODs crashes?

Try it by accessing url /crash of the application in your browser or run the crash-app.sh script. Then check the status of POD. Run the curl-test to see if users are impacted.

The container inside the POD is restarted.

```shell
./crash-app.sh 
Message from simple-webapp-2 : Mayday! Mayday! Going to crash!
./curl-test.sh 
Message from simple-webapp-1 : I am ready! OK

Message from simple-webapp-1 : I am ready! OK
```

10. When the application crashes, the container is restarted. During this period the service directs users to the available POD, since the POD status is not READY.

11. What would happen if the application inside container on one of the PODs freezes?

Try it by accessing url /freeze of the application in your browser or run the freeze-app.sh script. Then check the status of POD. Run the curl-test to see if users are impacted.

New users are impacted

```shell
./freeze-app.sh 
nohup: appending output to 'nohup.out'

./curl-test.sh 
Failed

Failed
```

12. Update both the pods with a livenessProbe using the given spec


Delete and recreate the PODs.

- Pod Name: simple-webapp-1
- Image Name: kodekloud/webapp-delayed-start
- Liveness Probe: httpGet
- Http Probe: /live
- Http Port: 8080
- Period Seconds: 1
- Initial Delay: 80
- Pod Name: simple-webapp-2
- Image Name: kodekloud/webapp-delayed-start
- Liveness Probe: httpGet
- Http Probe: /live
- Http Port: 8080
- Initial Delay: 80
- Period Seconds: 1

```shell
k edit po simple-webapp-1
```

```yaml
spec:
  containers:
  - name: simple-webapp
    livenessProbe:
      httpGet:
        path: /live
        port: 8080
      periodSeconds: 1
      initialDelaySeconds: 80
```

```shell
k replace --force -f /tmp/kubectl-edit-1461031540.yaml 

k edit po simple-webapp-2
```

```yaml
spec:
  containers:
  - name: simple-webapp
    livenessProbe:
      httpGet:
        path: /live
        port: 8080
      periodSeconds: 1
      initialDelaySeconds: 80
```

```shell
k replace --force -f /tmp/kubectl-edit-835078268.yaml
```

1.  Freeze the application again and notice what happens. When a POD freezes, it would be restarted automatically.

Try it by accessing url /freeze of the application in your browser. Then check the status of POD. Run the curl-test to see if users are impacted.

```shell
./curl-test.sh 
Message from simple-webapp-1 : I am ready! OK

Failed
```
