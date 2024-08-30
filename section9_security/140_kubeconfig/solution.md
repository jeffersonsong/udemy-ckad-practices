1. Where is the default kubeconfig file located in the current environment?

Find the current home directory by looking at the HOME environment variable.
/root/.kube/config

```shell
ls ~/.kube/config
/root/.kube/config
```

2. How many clusters are defined in the default kubeconfig file?

1

```shell
k config view
```

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://controlplane:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
```

cat ~/.kube/config

3. How many Users are defined in the default kubeconfig file?

1

4. How many contexts are defined in the default kubeconfig file?

1

5. What is the user configured in the current context?

kubernetes-admin

6. What is the name of the cluster configured in the default kubeconfig file?

kubernetes

7. A new kubeconfig file named my-kube-config is created. It is placed in the /root directory. How many clusters are defined in that kubeconfig file?
4

```shell
k config -h
k config view --kubeconfig=my-kube-config
```
```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443
  name: development
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443
  name: kubernetes-on-aws
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443
  name: production
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443
  name: test-cluster-1
contexts:
- context:
    cluster: kubernetes-on-aws
    user: aws-user
  name: aws-user@kubernetes-on-aws
- context:
    cluster: test-cluster-1
    user: dev-user
  name: research
- context:
    cluster: development
    user: test-user
  name: test-user@development
- context:
    cluster: production
    user: test-user
  name: test-user@production
current-context: test-user@development
kind: Config
preferences: {}
users:
- name: aws-user
    server: https://controlplane:6443
  name: kubernetes-on-aws
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443
  name: production
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443
  name: test-cluster-1
contexts:
- context:
    cluster: kubernetes-on-aws
    user: aws-user
  name: aws-user@kubernetes-on-aws
- context:
    cluster: test-cluster-1
    user: dev-user
  name: research
- context:
    cluster: development
    user: test-user
  name: test-user@development
- context:
    cluster: production
    user: test-user
  name: test-user@production
current-context: test-user@development
kind: Config
preferences: {}
users:
- name: aws-user
  user:
    client-certificate: /etc/kubernetes/pki/users/aws-user/aws-user.crt
    client-key: /etc/kubernetes/pki/users/aws-user/aws-user.key
- name: dev-user
  user:
    client-certificate: /etc/kubernetes/pki/users/dev-user/developer-user.crt
    client-key: /etc/kubernetes/pki/users/dev-user/dev-user.key
- name: test-user
  user:
    client-certificate: /etc/kubernetes/pki/users/test-user/test-user.crt
    client-key: /etc/kubernetes/pki/users/test-user/test-user.key
```

8. How many contexts are configured in the my-kube-config file?

4

9. What user is configured in the research context?

dev-user


10. What is the name of the client-certificate file configured for the aws-user?
aws-user.crt


11. What is the current context set to in the my-kube-config file?
test-user@development

12. I would like to use the dev-user to access test-cluster-1. Set the current context to the right one so I can do that.


Once the right context is identified, use the kubectl config use-context command.

- Current context set

```shell
k config use-context research --kubeconfig=my-kube-config
Switched to context "research".
```

13. We don't want to have to specify the kubeconfig file option on each command.

Set the my-kube-config file as the default kubeconfig by overwriting the content of ~/.kube/config with the content of the my-kube-config file.

Default kubeconfig file configured

```shell
cp my-kube-config ~/.kube/config
k config view
```

14. With the current-context set to research, we are trying to access the cluster. However something seems to be wrong. Identify and fix the issue.


Try running the kubectl get pods command and look for the error. All users certificates are stored at /etc/kubernetes/pki/users.

Issue fixed

```shell
k get pod
error: unable to read client-cert /etc/kubernetes/pki/users/dev-user/developer-user.crt for dev-user due to open /etc/kubernetes/pki/users/dev-user/developer-user.crt: no such file or directory

ls /etc/kubernetes/pki/users/dev-user/
dev-user.crt  dev-user.csr  dev-user.key

vi ~/.kube/config
```

From
```yaml
users:
- name: dev-user
  user:
    client-certificate: /etc/kubernetes/pki/users/dev-user/developer-user.crt
```

To
```yaml
users:
- name: dev-user
  user:
    client-certificate: /etc/kubernetes/pki/users/dev-user/dev-user.crt
```

```shell
k get pod
No resources found in default namespace.
```