1. We have deployed a number of PODs. They are labelled with tier, env and bu. How many PODs exist in the dev environment (env)?

Use selectors to filter the output

7

```shell
k get po --selector="env=dev"
k get po --selector="env=dev" --no-headers | wc -l
7
```

2. How many PODs are in the finance business unit (bu)?

6

```shell
k get po --selector="bu=finance" --no-headers | wc -l
6
```

3. How many objects are in the prod environment including PODs, ReplicaSets and any other objects?

7

```shell
k get all --selector="env=prod" --no-headers | wc -l
7
```

4. Identify the POD which is part of the prod environment, the finance BU and of frontend tier?
app-1-zzxdf

```shell
k get po --selector="env=prod,bu=finance,tier=frontend" 
NAME          READY   STATUS    RESTARTS   AGE
app-1-zzxdf   1/1     Running   0          2m51s
```

5. A ReplicaSet definition file is given replicaset-definition-1.yaml. Attempt to create the replicaset; you will encounter an issue with the file. Try to fix it.

Once you fix the issue, create the replicaset from the definition file.

- ReplicaSet: replicaset-1
- Replicas: 2

```shell
k apply -f replicaset-definition-1.yaml 
The ReplicaSet "replicaset-1" is invalid: spec.template.metadata.labels: Invalid value: map[string]string{"tier":"nginx"}: `selector` does not match template `labels`

vi replicaset-definition-1.yaml
```

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
   name: replicaset-1
spec:
   replicas: 2
   selector:
      matchLabels:
        tier: front-end
   template:
     metadata:
       labels:
        tier: front-end
     spec:
       containers:
       - name: nginx
         image: nginx
```

```shell
k apply -f replicaset-definition-1.yaml 
replicaset.apps/replicaset-1 created
```