1. CRD Object can be either namespaced or cluster scoped.

Is this statement true or false?

true

2. What is a custom resource?

It is an extension of the Kubernetes API that is not necessarily available in a default Kubernetes installation.

3. We have provided an incomplete CRDs manifest file called crd.yaml under the /root directory. Let’s complete it and create a custom resource definition from it.

Let’s create a custom resource definition called internals.datasets.kodekloud.com. Assign the group to datasets.kodekloud.com and the resource is accessible only from a specific namespace.

Make sure the version should be v1 and needed to enable the version so it’s being served via REST API.
So finally create a custom resource from a given manifest file called custom.yaml.

Note :- Make sure resources should be created successfully from the custom.yaml file.

- CRD created?
- Corrected the version?
- Enable the version?
- Deployed custom resource?

```shell
vi crd.yaml
```

```yaml
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: internals.datasets.kodekloud.com 
spec:
  group: datasets.kodekloud.com 
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                internalLoad:
                  type: string
                range:
                  type: integer
                percentage:
                  type: string
  scope: Namespaced
  names:
    plural: internals
    singular: internal
    kind: Internal
    shortNames:
    - int
```

```shell
k create -f crd.yaml 
customresourcedefinition.apiextensions.k8s.io/internals.datasets.kodekloud.com created

k api-resources | grep internals
internals                           int          datasets.kodekloud.com/v1         true         Internal

k create -f custom.yaml
internal.datasets.kodekloud.com/internal-space created
```

4. What are the properties given to the CRD’s called collectors.monitoring.controller?
image,name,replicas

```shell
k describe crd collectors.monitoring.controller
```

```yaml
            Properties:
              Image:
                Type:  string
              Name:
                Type:  string
              Replicas:
                Type:  integer
```

5. Create a custom resource called datacenter and the apiVersion should be traffic.controller/v1.

Set the dataField length to 2 and access permission should be true.

- Resource created?
- Name: datacenter
- dataField: 2
- access: true

```shell
k get crd globals.traffic.controller -o yaml
```

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  creationTimestamp: "2024-08-29T20:02:31Z"
  generation: 1
  name: globals.traffic.controller
  resourceVersion: "876"
  uid: f4cfc0ce-5247-45fa-b451-a2a147a978d0
spec:
  conversion:
    strategy: None
  group: traffic.controller
  names:
    kind: Global
    listKind: GlobalList
    plural: globals
    shortNames:
    - gb
    singular: global
  scope: Namespaced
  versions:
  - name: v1
    schema:
      openAPIV3Schema:
        properties:
          spec:
            properties:
              access:
                type: boolean
              dataField:
                type: integer
            type: object
        type: object
    served: true
    storage: true
status:
  acceptedNames:
    kind: Global
    listKind: GlobalList
    plural: globals
    shortNames:
    - gb
    singular: global
  conditions:
  - lastTransitionTime: "2024-08-29T20:02:31Z"
    message: no conflicts found
    reason: NoConflicts
    status: "True"
    type: NamesAccepted
  - lastTransitionTime: "2024-08-29T20:02:31Z"
    message: the initial names have been accepted
    reason: InitialNamesAccepted
    status: "True"
    type: Established
  storedVersions:
  - v1
```

```shell
vi datacenter.yaml 
```

```yaml
---
kind: Global
apiVersion: traffic.controller/v1
metadata:
  name: datacenter
  namespace: default
spec:
  dataField: 2
  access: true
```


7. What is the short name for globals.traffic.controller?

gb

```shell
k api-resources | grep globals
globals                             gb           traffic.controller/v1             true         Global

k create -f datacenter.yaml 
global.traffic.controller/datacenter created
```
