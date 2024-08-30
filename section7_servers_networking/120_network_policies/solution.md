1. How many network policies do you see in the environment?

We have deployed few web applications, services and network policies. Inspect the environment.

```shell
k get netpol -A
NAMESPACE   NAME             POD-SELECTOR   AGE
default     payroll-policy   name=payroll   28s
```

2. What is the name of the Network Policy?

payroll-policy

3. Which pod is the Network Policy applied on?

payroll

```shell
k get pod --selector="name=payroll"
NAME      READY   STATUS    RESTARTS   AGE
payroll   1/1     Running   0          90s
```

4. What type of traffic is this Network Policy configured to handle?

Ingress

```shell
k describe netpol payroll-policy
Name:         payroll-policy
Namespace:    default
Created on:   2024-08-30 01:55:24 +0000 UTC
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     name=payroll
  Allowing ingress traffic:
    To Port: 8080/TCP
    From:
      PodSelector: name=internal
  Not affecting egress traffic
  Policy Types: Ingress
```

5. What is the impact of the rule configured on this Network Policy?

Traffic From Internal to Payroll POD is allowed.

6. What is the impact of the rule configured on this Network Policy?

Internal POD can access port 8080 on Payroll POD

7. Access the UI of these applications using the link given above the terminal.

8. Perform a connectivity test using the User Interface in these Applications to access the payroll-service at port 8080.

Only Internal application can access payroll service

9. Perform a connectivity test using the User Interface of the Internal Application to access the external-service at port 8080.

Successful

10. Create a network policy to allow traffic from the Internal application only to the payroll-service and db-service.

Use the spec given below. You might want to enable ingress traffic to the pod to test your rules in the UI.

Also, ensure that you allow egress traffic to DNS ports TCP and UDP (port 53) to enable DNS resolution from the internal pod.

- Policy Name: internal-policy
- Policy Type: Egress
- Egress Allow: payroll
- Payroll Port: 8080
- Egress Allow: mysql
- MySQL Port: 3306

```shell
vi internal-policy.yaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          name: payroll
    ports:
    - protocol: TCP
      port: 8080
  - to:
    - podSelector:
        matchLabels:
          name: mysql
    ports:
    - protocol: TCP
      port: 3306
```

```shell
k apply -f internal-policy.yaml 
networkpolicy.networking.k8s.io/internal-policy created
```
