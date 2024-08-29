1. Inspect the environment and identify the authorization modes configured on the cluster.

Check the kube-apiserver settings.

- [ ] Node
- [ ] ABAC
- [x] Node,RBAC
- [ ] RBAC

```shell
controlplane ~ ➜  k cluster-info dump | grep authorization-mode
                            "--authorization-mode=Node,RBAC",
```

2. How many roles exist in the default namespace?
0

```shell
controlplane ~ ➜  k get role
No resources found in default namespace.
```

3. How many roles exist in all namespaces together?
12

```shell
controlplane ~ ➜  k get role -A --no-headers | wc -l
```

4. What are the resources the kube-proxy role in the kube-system namespace is given access to?
configMaps

```shell
controlplane ~ ➜  k describe role kube-proxy -n kube-system
Name:         kube-proxy
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources   Non-Resource URLs  Resource Names  Verbs
  ---------   -----------------  --------------  -----
  configmaps  []                 [kube-proxy]    [get]
```

5. What actions can the kube-proxy role perform on configmaps?

Get

6. Which of the following statements are true?

kube-proxy role can get details of config,ap object by the name kube-proxy only.

7. Which account is the kube-proxy role assigned to?
Group: system:bootstrappers:kubeadm:default-node-token

```shell
controlplane ~ ➜  k get rolebinding -n kube-system
NAME                                                ROLE                                                  AGE
kube-proxy                                          Role/kube-proxy                                       7m19s
kubeadm:kubelet-config                              Role/kubeadm:kubelet-config                           7m20s
kubeadm:nodes-kubeadm-config                        Role/kubeadm:nodes-kubeadm-config                     7m20s
system::extension-apiserver-authentication-reader   Role/extension-apiserver-authentication-reader        7m20s
system::leader-locking-kube-controller-manager      Role/system::leader-locking-kube-controller-manager   7m20s
system::leader-locking-kube-scheduler               Role/system::leader-locking-kube-scheduler            7m20s
system:controller:bootstrap-signer                  Role/system:controller:bootstrap-signer               7m20s
system:controller:cloud-provider                    Role/system:controller:cloud-provider                 7m20s
system:controller:token-cleaner                     Role/system:controller:token-cleaner                  7m20s

controlplane ~ ➜  k describe rolebinding kube-proxy -n kube-system
Name:         kube-proxy
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  kube-proxy
Subjects:
  Kind   Name                                             Namespace
  ----   ----                                             ---------
  Group  system:bootstrappers:kubeadm:default-node-token  
```


8. A user dev-user is created. User's details have been added to the kubeconfig file. Inspect the permissions granted to the user. Check if the user can list pods in the default namespace.

Use the --as dev-user option with kubectl to run commands as the dev-user.

dev-user does not have permissions to list pods

```shell
controlplane ~ ➜  k auth can-i --as dev-user get pods
no
```

9. Create the necessary roles and role bindings required for the dev-user to create, list and delete pods in the default namespace.


Use the given spec:


Role: developer

Role Resources: pods

Role Actions: list

Role Actions: create

Role Actions: delete

RoleBinding: dev-user-binding

RoleBinding: Bound to dev-user

```shell
controlplane ~ ✖ k create role -h | less

controlplane ~ ➜  kubectl create role developer --verb=list,create,delete,get --resource=pods
role.rbac.authorization.k8s.io/developer created

controlplane ~ ➜  k create rolebinding -h | less

controlplane ~ ➜  kubectl create rolebinding dev-user-binding --role=developer --user=dev-user
rolebinding.rbac.authorization.k8s.io/dev-user-binding created
```

10. A set of new roles and role-bindings are created in the blue namespace for the dev-user. However, the dev-user is unable to get details of the dark-blue-app pod in the blue namespace. Investigate and fix the issue.


We have created the required roles and rolebindings, but something seems to be wrong.

Issue Fixed

```shell
controlplane ~ ➜  k get role,rolebinding -n blue
NAME                                       CREATED AT
role.rbac.authorization.k8s.io/developer   2024-08-29T17:27:14Z

NAME                                                     ROLE             AGE
rolebinding.rbac.authorization.k8s.io/dev-user-binding   Role/developer   12m

controlplane ~ ✖ k auth can-i --as dev-user get pod/dark-blue-app -n blue
no

controlplane ~ ✖ k describe role developer -n blue
Name:         developer
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  pods       []                 [blue-app]      [get watch create delete]

controlplane ~ ➜  k edit role developer -n blue
role.rbac.authorization.k8s.io/developer edited

controlplane ~ ➜  k auth can-i --as dev-user get pod/dark-blue-app -n blue
yes
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: "2024-08-29T17:27:14Z"
  name: developer
  namespace: blue
  resourceVersion: "589"
  uid: c802b247-727b-4fe8-85e7-8a92e9d040ed
rules:
- apiGroups:
  - ""
  resourceNames:
  - blue-app
  - dark-blue-app
  resources:
  - pods
  verbs:
  - get
  - watch
  - create
  - delete
```

11. Add a new rule in the existing role developer to grant the dev-user permissions to create deployments in the blue namespace.

Remember to add api group "apps".

```shell
controlplane ~ ➜  k edit role developer -n blue
role.rbac.authorization.k8s.io/developer edited

controlplane ~ ✖ k auth can-i --as dev-user create deployment -n blue
yes
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: "2024-08-29T17:27:14Z"
  name: developer
  namespace: blue
  resourceVersion: "1949"
  uid: c802b247-727b-4fe8-85e7-8a92e9d040ed
rules:
- apiGroups:
  - ""
  resourceNames:
  - blue-app
  - dark-blue-app
  resources:
  - pods
  verbs:
  - get
  - watch
  - create
  - delete
- apiGroups:
  - "apps"
  resources:
  - deployments
  verbs:
  - create
```
