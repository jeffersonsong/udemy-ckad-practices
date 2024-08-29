1. Which command is used to search for a wordpress helm chart package from the Artifact Hub?

helm search hub wordpress

```shell
controlplane ~ ➜  helm search -h
controlplane ~ ➜  helm search hub wordpress
```

2. Add a bitnami helm chart repository in the controlplane node.

- name - bitnami
- chart repo name - https://charts.bitnami.com/bitnami

Chart added successfully?

```shell
controlplane ~ ➜  helm repo add -h

controlplane ~ ➜  helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories
```

3. Which command is used to search for the joomla package from the added repository?

```shell
controlplane ~ ➜  helm search repo joomla
NAME            CHART VERSION   APP VERSION     DESCRIPTION                                       
bitnami/joomla  20.0.4          5.1.2           DEPRECATED Joomla! is an award winning open sou...
```

4. What is the app version of joomla in the bitnami helm repository?

5.1.2

5. Which chart version can you see for the joomla package in the bitnami helm repo?

20.0.4

6. How many helm repositories are added in the controlplane node?

3

```shell
controlplane ~ ➜  helm repo list
NAME            URL                                                 
bitnami         https://charts.bitnami.com/bitnami                  
puppet          https://puppetlabs.github.io/puppetserver-helm-chart
hashicorp       https://helm.releases.hashicorp.com   
```

7. Install drupal helm chart from the bitnami repository.

- Release name should be bravo.
- Chart name should be bitnami/drupal.
- Note: Ignore the state of the application now.

- Chart name is correct?
- Release name is correct?

```shell
controlplane ~ ➜  helm install -h | less

controlplane ~ ➜  helm install bravo bitnami/drupal
```

8. Which command is used to list packages installed using helm?

```shell
controlplane ~ ➜  helm list
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
bravo   default         1               2024-08-29 20:48:31.929059416 +0000 UTC deployed        drupal-20.0.1   11.0.1     
```

9. Uninstall the drupal helm package which we installed earlier.
- Uninstalled successfully?

```shell
controlplane ~ ➜  helm -h | less

controlplane ~ ➜  helm uninstall -h | less

controlplane ~ ➜  helm uninstall bravo
release "bravo" uninstalled
```

10. Download the bitnami apache package under the /root directory.
- Note: Do not install the package. Just download it.

```shell
controlplane ~ ➜  helm -h | less
pull        download a chart from a repository and (optionally) unpack it in local directory

controlplane ~ ➜  helm pull -h | less

controlplane ~ ✖ helm search repo apache
NAME                            CHART VERSION   APP VERSION     DESCRIPTION                                       
bitnami/apache                  11.2.16         2.4.62          Apache HTTP Server is an open-source HTTP serve...

controlplane ~ ➜  helm pull bitnami/apache --untar
```

11. Inspect the file values.yaml and make changes so that 2 replicas of the webserver are running and the http is exposed on nodeport 30080.

Note: You can read the Bitnami documentation for more.
https://github.com/bitnami/charts/tree/master/bitnami/apache/#installing-the-chart


```shell
controlplane ~ ➜ vi values.yaml
```

```yaml
## @param replicaCount Number of replicas of the Apache deployment
##
replicaCount: 2

## Apache service parameters
##
service:
  ## Node ports to expose
  ## @param service.nodePorts.http Node port for HTTP
  ## @param service.nodePorts.https Node port for HTTPS
  ##
  nodePorts:
    http: "30080"
    https: ""
```

12. Install the apache from the downloaded helm package.
- Release name: mywebapp
Note: Do make changes accordingly so that 2 replicas of the webserver are running and the http is exposed on nodeport 30080.

Make sure that the pods are in the running state.

- Release name is correct?
- Installed successfully?
- Deployment created?
- Two pods running?
- Correct NodePort?

```shell
controlplane ~/apache is 📦 v11.2.16 via ⎈ v3.15.3 ➜  helm install mywebapp .
```

13. You can access the Apache default page by clicking on mywebapp link from the top of the terminal.
