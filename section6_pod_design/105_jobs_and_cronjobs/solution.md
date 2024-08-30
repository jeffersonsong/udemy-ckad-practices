1. A pod definition file named throw-dice-pod.yaml is given. The image throw-dice randomly returns a value between 1 and 6. 6 is considered success and all others are failure. Try deploying the POD and view the POD logs for the generated number.

File is located at /root/throw-dice-pod.yaml

- Pod Name: throw-dice-pod
- Image Name: kodekloud/throw-dice

```shell
k create -f throw-dice-pod.yaml 
pod/throw-dice-pod created
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: throw-dice-pod
spec:
  containers:
  -  image: kodekloud/throw-dice
     name: throw-dice
  restartPolicy: Never
```

2. Create a Job using this POD definition file or from the imperative command and look at how many attempts does it take to get a '6'.
Use the specification given on the below.

- Job Name: throw-dice-job
- Image Name: kodekloud/throw-dice

```shell
kubectl create job throw-dice-job --image=kodekloud/throw-dice --dry-run=client -o yaml

kubectl create job throw-dice-job --image=kodekloud/throw-dice
job.batch/throw-dice-job created
```

3. Monitor and wait for the job to succeed. Throughout this practice test remember to increase the 'BackOffLimit' to prevent the job from quitting before it succeeds.

Check out the documentation page about the BackOffLimit property.

- Job Name: throw-dice-job
- Image Name: kodekloud/throw-dice
- Job Succeeded: True

```shell
k get job throw-dice-job -w
```

4. How many attempts did it take to complete the job?

```shell
k describe job throw-dice-job

Pods Statuses:    0 Active (0 Ready) / 1 Succeeded / 5 Failed
Events:
  Type    Reason            Age    From            Message
  ----    ------            ----   ----            -------
  Normal  SuccessfulCreate  5m24s  job-controller  Created pod: throw-dice-job-cxgfg
  Normal  SuccessfulCreate  5m13s  job-controller  Created pod: throw-dice-job-m867m
  Normal  SuccessfulCreate  4m52s  job-controller  Created pod: throw-dice-job-75ww2
  Normal  SuccessfulCreate  4m11s  job-controller  Created pod: throw-dice-job-wgw64
  Normal  SuccessfulCreate  2m50s  job-controller  Created pod: throw-dice-job-wcfsx
  Normal  SuccessfulCreate  9s     job-controller  Created pod: throw-dice-job-fp46n
  Normal  Completed         5s     job-controller  Job completed
```

5. Update the job definition to run as many times as required to get 2 successful 6's.

Delete existing job and create a new one with the given spec. Monitor and wait for the job to succeed.

- Job Name: throw-dice-job
- Image Name: kodekloud/throw-dice
- Completions: 2
- Job Succeeded: True

```shell
k get job throw-dice-job -o yaml > throw-dice-job.yaml

vi throw-dice-job.yaml 
```

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: throw-dice-job
  namespace: default
spec:
  backoffLimit: 20
  completionMode: NonIndexed
  completions: 2
  manualSelector: false
  parallelism: 1
  podReplacementPolicy: TerminatingOrFailed
  suspend: false
  template:
    metadata:
      labels:
        job-name: throw-dice-job
    spec:
      containers:
      - image: kodekloud/throw-dice
        imagePullPolicy: Always
        name: throw-dice-job
        resources: {}
      restartPolicy: Never
```

```shell
k apply -f throw-dice-job.yaml 
job.batch/throw-dice-job created

k get job -w
```

6. How many attempts did it take to complete the job this time?

```shell
k describe job throw-dice-job 

Pods Statuses:    0 Active (0 Ready) / 2 Succeeded / 3 Failed
```

7. That took a while. Let us try to speed it up, by running upto 3 jobs in parallel.

Update the job definition to run 3 jobs in parallel.

- Job Name: throw-dice-job
- Image Name: kodekloud/throw-dice
- Completions: 3
- Parallelism: 3

```shell
k delete job throw-dice-job 
job.batch "throw-dice-job" deleted
```

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: throw-dice-job
  namespace: default
spec:
  backoffLimit: 20
  completionMode: NonIndexed
  completions: 3
  manualSelector: false
  parallelism: 3
  podReplacementPolicy: TerminatingOrFailed
  suspend: false
  template:
    metadata:
      labels:
        job-name: throw-dice-job
    spec:
      containers:
      - image: kodekloud/throw-dice
        imagePullPolicy: Always
        name: throw-dice-job
        resources: {}
      restartPolicy: Never
```

```shell
k apply -f throw-dice-job.yaml 
job.batch/throw-dice-job created

k get job -w
```

8. Notice how quickly that finished.

```shell
k describe job throw-dice-job 

Pods Statuses:    0 Active (0 Ready) / 3 Succeeded / 4 Failed
```

9. Let us now schedule that job to run at 21:30 hours every day.

Create a CronJob for this.

- CronJob Name: throw-dice-cron-job
- Image Name: kodekloud/throw-dice
- Schedule: 30 21 * * *

```shell
k create cj -h 

kubectl create cronjob throw-dice-cron-job --image=kodekloud/throw-dice --schedule="30 21 * * *"
cronjob.batch/throw-dice-cron-job created
```
