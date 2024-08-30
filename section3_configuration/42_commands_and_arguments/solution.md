1. How many PODs exist on the system?

In the current(default) namespace

1

```shell
k get po --no-headers | wc -l
1
```

k get po
NAME             READY   STATUS    RESTARTS   AGE
ubuntu-sleeper   1/1     Running   0          41s

2. What is the command used to run the pod ubuntu-sleeper?

sleep 4800

```shell
k describe po ubuntu-sleeper

    Command:
      sleep
      4800
```

3. Create a pod with the ubuntu image to run a container to sleep for 5000 seconds. Modify the file ubuntu-sleeper-2.yaml.

Note: Only make the necessary changes. Do not modify the name.

- Pod Name: ubuntu-sleeper-2
- Command: sleep 5000

```shell
k apply -f ubuntu-sleeper-2.yaml 
pod/ubuntu-sleeper-2 created
```

4. Create a pod using the file named ubuntu-sleeper-3.yaml. There is something wrong with it. Try to fix it!

Note: Only make the necessary changes. Do not modify the name.

- Pod Name: ubuntu-sleeper-3
- Command: sleep 1200

```shell
k apply -f ubuntu-sleeper-3.yaml 
Error from server (BadRequest): error when creating "ubuntu-sleeper-3.yaml": Pod in version "v1" cannot be handled as a Pod: json: cannot unmarshal number into Go struct field Container.spec.containers.command of type string

k apply -f ubuntu-sleeper-3.yaml 
pod/ubuntu-sleeper-3 created
```

5. Update pod ubuntu-sleeper-3 to sleep for 2000 seconds.
Note: Only make the necessary changes. Do not modify the name of the pod. Delete and recreate the pod if necessary.

- Pod Name: ubuntu-sleeper-3
- Command: sleep 2000

```shell
k edit po ubuntu-sleeper-3 

k replace --force -f /tmp/kubectl-edit-3603704817.yaml
```

6. Inspect the file Dockerfile given at /root/webapp-color directory. What command is run at container startup?

python app.py

7. Inspect the file Dockerfile2 given at /root/webapp-color directory. What command is run at container startup?

python app.py --color red

8. Inspect the two files under directory webapp-color-2. What command is run at container startup?

Assume the image was created from the Dockerfile in this directory.

--color green

9. Inspect the two files under directory webapp-color-3. What command is run at container startup?

Assume the image was created from the Dockerfile in this directory.

python app.py --color pink

10. Create a pod with the given specifications. By default it displays a blue background. Set the given command line arguments to change it to green.

- Pod Name: webapp-green
- Image: kodekloud/webapp-color
- Command line arguments: --color=green

```shell
k run webapp-green --image=kodekloud/webapp-color --dry-run=client -o yaml -- --color=green

k run webapp-green --image=kodekloud/webapp-color -- --color=green 
pod/webapp-green created
```