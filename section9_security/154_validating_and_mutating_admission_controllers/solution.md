1. Which of the below combination is correct for Mutating and validating admission controllers ?

NamespaceAutoProvision- Mutating, NamespaceExists- Validating

2. What is the flow of invocation of admission controllers?

First Mutating then Validating

3. Create namespace webhook-demo where we will deploy webhook components
- webhook-demo namespace created?

```shell
k create ns webhook-demo
namespace/webhook-demo created
```

4.  Create TLS secret webhook-server-tls for secure webhook communication in webhook-demo namespace.

We have already created below cert and key for webhook server which should be used to create secret.

- Certificate : /root/keys/webhook-server-tls.crt
- Key : /root/keys/webhook-server-tls.key

- Is the webhook-server-tls of the tls secret type created correctly?

```shell
k create secret -h
k create secret tls -h | less
kubectl create secret tls webhook-server-tls --cert=/root/keys/webhook-server-tls.crt --key=/root/keys/webhook-server-tls.key -n webhook-demo 
secret/webhook-server-tls created
```

5. Create webhook deployment now.

We have already added sample deployment definition under /root/webhook-deployment.yaml so just create deployment with that definition.

- webhook-server deployed?

```shell
k create -f webhook-deployment.yaml 
deployment.apps/webhook-server created
```

6. Create webhook service now so that admission controller can communicate with webhook.

We have already added sample service definition under /root/webhook-service.yaml so just create service with that definition.

- webhook-server service created?

```shell
k create -f webhook-service.yaml 
service/webhook-server created
```

7. We have added MutatingWebhookConfiguration under /root/webhook-configuration.yaml.
If we apply this configuration which resource and actions it will affect?

Pod with CREATE operations


8. Now lets deploy MutatingWebhookConfiguration in /root/webhook-configuration.yaml
- MutatingWebhookConfiguration deployed?

```shell
k create -f webhook-configuration.yaml 
mutatingwebhookconfiguration.admissionregistration.k8s.io/demo-webhook created

k get mutatingwebhookconfiguration
NAME           WEBHOOKS   AGE
demo-webhook   1          16s
```

9. In previous steps we have deployed demo webhook which does below

- Denies all request for pod to run as root in container if no securityContext is provided.
- If no value is set for runAsNonRoot, a default of true is applied, and the user ID defaults to 1234
- Allow to run containers as root if runAsNonRoot set explicitly to false in the securityContext

In next steps we have added some pod definitions file for each scenario. Deploy those pods with existing definitions file and validate the behaviour of our webhook

10. Deploy a pod with no securityContext specified.

We have added pod definition file under /root/pod-with-defaults.yaml

pod-with-defaults pod created?

```shell
k create -f pod-with-defaults.yaml 
pod/pod-with-defaults created

k get pod
NAME                READY   STATUS      RESTARTS   AGE
pod-with-defaults   0/1     Completed   0          50s
simple-webapp-1     1/1     Running     0          10m
```

11. What are runAsNonRoot and runAsUser values for previously created pods securityContext?

We did not specify any securityContext values in pod definition so check out the changes done by mutation webhook in pod

runAsNonRoot: true, runAsUser: 1234

```shell
k get po pod-with-defaults -o yaml | less
```

```yaml
  securityContext:
    runAsNonRoot: true
    runAsUser: 1234
```

12. Deploy pod with a securityContext explicitly allowing it to run as root

We have added pod definition file under /root/pod-with-override.yaml

Validate securityContext after you deploy this pod

- pod-with-override pod created?

```shell
k create -f pod-with-override.yaml 
pod/pod-with-override created

k get pod pod-with-override -o yaml | less
```
```yaml
  securityContext:
    runAsNonRoot: false
```

13. Deploy a pod with a conflicting securityContext i.e. pod running with a user id of 0 (root)

We have added pod definition file under /root/pod-with-conflict.yaml

Mutating webhook should reject the request as its asking to run as root user without setting runAsNonRoot: false

- pod-with-conflict pod not created?

```shell
k create -f pod-with-conflict.yaml 
Error from server: error when creating "pod-with-conflict.yaml": admission webhook "webhook-server.webhook-demo.svc" denied the request: runAsNonRoot specified, but runAsUser set to 0 (the root user)
```
