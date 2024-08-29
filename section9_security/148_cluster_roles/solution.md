1. For the first few questions of this lab, you would have to inspect the existing ClusterRoles and ClusterRoleBindings that have been created in this cluster.

```shell
k get clusterrole,clusterrolebinding
```

2. How many ClusterRoles do you see defined in the cluster?
72

```shell
k get clusterrole --no-headers | wc -l
72
```

3. How many ClusterRoleBindings exist on the cluster?
57

```shell
k get clusterrolebinding --no-headers | wc -l
57
```

4. What namespace is the cluster-admin clusterrole part of?
Cluster Roles are cluster wide and not part of any namespace.

```shell
k describe clusterrole cluster-admin
Name:         cluster-admin
Labels:       kubernetes.io/bootstrapping=rbac-defaults
Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  *.*        []                 []              [*]
             [*]                []              [*]
```

5. What user/groups are the cluster-admin role bound to?

The ClusterRoleBinding for the role is with the same name.
system:masters

```shell
k get clusterrolebinding | grep cluster-admin
cluster-admin                                                   ClusterRole/cluster-admin                                                   15m
helm-kube-system-traefik                                        ClusterRole/cluster-admin                                                   14m
helm-kube-system-traefik-crd                                    ClusterRole/cluster-admin                                                   14m

k describe clusterrolebinding cluster-admin
Name:         cluster-admin
Labels:       kubernetes.io/bootstrapping=rbac-defaults
Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
Role:
  Kind:  ClusterRole
  Name:  cluster-admin
Subjects:
  Kind   Name            Namespace
  ----   ----            ---------
  Group  system:masters  
```

6. What level of permission does the cluster-admin role grant?

Inspect the cluster-admin role's privileges.

Perform any action on any resource in the cluster

```shell
k describe clusterrole cluster-admin
Name:         cluster-admin
Labels:       kubernetes.io/bootstrapping=rbac-defaults
Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  *.*        []                 []              [*]
             [*]                []              [*]
```

7. A new user michelle joined the team. She will be focusing on the nodes in the cluster. Create the required ClusterRoles and ClusterRoleBindings so she gets access to the nodes.

```shell
k create clusterrole -h

kubectl create clusterrole node-reader --verb=get,list,watch --resource=nodes
clusterrole.rbac.authorization.k8s.io/node-reader created

k create clusterrolebinding -h

kubectl create clusterrolebinding michelle-node-reader --clusterrole=node-reader --user=michelle
clusterrolebinding.rbac.authorization.k8s.io/michelle-node-reader created
```

8. michelle's responsibilities are growing and now she will be responsible for storage as well. Create the required ClusterRoles and ClusterRoleBindings to allow her access to Storage.

Get the API groups and resource names from command kubectl api-resources. Use the given spec:
- ClusterRole: storage-admin
- Resource: persistentvolumes
- Resource: storageclasses
- ClusterRoleBinding: michelle-storage-admin
- ClusterRoleBinding Subject: michelle
- ClusterRoleBinding Role: storage-admin

```shell
k create clusterrole -h

kubectl create clusterrole storage-admin --verb=get,list,watch --resource=persistentvolumes,storageclasses
clusterrole.rbac.authorization.k8s.io/storage-admin created

k create clusterrolebinding -h

kubectl create clusterrolebinding michelle-storage-admin --clusterrole=storage-admin --user=michelle
clusterrolebinding.rbac.authorization.k8s.io/michelle-storage-admin created
```
