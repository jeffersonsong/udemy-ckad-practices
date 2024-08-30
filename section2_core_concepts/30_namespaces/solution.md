1. How many Namespaces exist on the system?

10

```shell
k get ns
NAME              STATUS   AGE
kube-system       Active   14m
kube-public       Active   14m
kube-node-lease   Active   14m
default           Active   14m
finance           Active   26s
marketing         Active   26s
dev               Active   26s
prod              Active   26s
manufacturing     Active   26s
research          Active   26s

k get ns --no-headers | wc -l
10
```

2. How many pods exist in the research namespace?

2

```shell
k get po -n research
NAME    READY   STATUS      RESTARTS      AGE
dna-1   0/1     Completed   3 (33s ago)   57s
dna-2   0/1     Completed   3 (35s ago)   57s
```

3. Create a POD in the finance namespace.

Use the spec given below.

- Name: redis
- Image name: redis

```shell
k run redis --image=redis -n finance
pod/redis created
```

4. Which namespace has the blue pod in it?

marketing

```shell
k get pod -A | grep blue
marketing       blue                                      1/1     Running            0             2m1s
```

5. Access the Blue web application using the link above your terminal!!

From the UI you can ping other services.

6. What DNS name should the Blue application use to access the database db-service in its own namespace - marketing?

You can try it in the web application UI. Use port 6379.

db-service

7. What DNS name should the Blue application use to access the database db-service in the dev namespace?

You can try it in the web application UI. Use port 6379.

db-service.dev.svc.cluster.local
