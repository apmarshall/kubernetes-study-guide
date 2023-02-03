# Kubernetes Study Guide

A collection of notes on Kubernetes to prepare for CKA

## Anatomy of an Object Manifest

### Standard Material
This basically appears in all Kubernetes objects.

```yml
apiVersion: v1 # the version of the Kubernetes API which contains the type of object you’re creating
kind: object-type # The type of object you are going to create (see above for examples)
metadata:
  name: object-name # always required, must be unique within namespace (or cluster for cluster-wide resources)
  namespace: namespace-to-create-in # optional, if not specified uses default or current context
  labels: # recommended, but optional
    app.kubernetes.io/name: name 
    app.kubernetes.io/instance: name-identifier 
    app.kubernetes.io/version: “1.2.3” 
    app.kubernetes.io/component: database # example
    app.kubernetes.io/part-of: wordpress # example
    app.kubernetes.io/managed-by: helm # example
    release: stable # more examples
    environment: dev
    tier: backend
    partition: customerA
    track: daily
  annotations:  # examples, optional
    time: 20230201T1201
    release-id: 12345
    branch: main
    hash: sha256…
    registry: “https://hub.docker.com/“  
    owner: me@example.com
spec:
```
Common apiVersion Values:
- `v1`: Pod, Node, Service, ConfigMap, PersistentVolumeClaim, Secret, Volume 
- `apps/v1`: DaemonSet, Deployment, ReplicaSet, StatefulSet
- `batch/v1`: Job

### Basic Pod Object
This is the building block that gets nested in many other K8s objects.

```yml
  metadata:
    labels: 
      key: value 
  spec: 
    containers:
    - name: container-name
      image: container-image:1.1.1
      command: [‘sh’, ‘-c’, ‘echo “Hello, Kubernetes!” && sleep 3600’] # examples, optional
      ports: # optional
      - containerPort: 80
  restartPolicy: OnFailure # optional, defaults to Always
```

### Specs for Common Pod Controllers
ie, Deployments, DaemonSets, Jobs

```yml
spec:
  replicas: 3 # Deployments, replicaSets, etc
  selector: # Used to match pods in things like deployments or services
    matchLabels: # equality-based
      key: value 
    matchExpressions: # set-based
      - {key: tier, operator: In, values: [backend]}
      - {key: environment, operator: NotIn, values: [prod]}    
  StrategyType: RollingUpdate | Recreate # Deployments, defaults to RollingUpdate  
  template:
      # Insert Pod Template here. Must contain labels matching selectors (may contain additional)
```

Selectors can be:
  - Equality-based: `=`, `==`, or `!=`
  - Set-based: `in`, `notin`, or `exists`
 
Selectors are logical `and` or `&&` statements, there is no `or` equivalent  

### Special Cases:

#### Nodes:

```yml
kind: Node
apiVersion: v1
metadata:
  name: node-name # must be unique
  labels:
    key: value
```

## Kubectl 

Synopsis:

```bash
kubectl <command> <target> <args> [-l key=value,key!=value | ‘key in (value1, value2),key notin(value)’] [--field-selector type.key=value] [--namespace=namespace-name] [--all-namespaces]
```

Key Commands:
  - `create` - imperative command to create an object
    - `<target>` = object name
    - common arguments:
      - `--image <image-name>`
  - `create -f <file-name>.yml` - imperative command to creat objects from a file
  - `diff -f <file-or-directory>` - declarative command to compare the contents of the target file/directory to the current cluster state. `diff -R -f <file-or-directory>` -> recursive comparison through subdirectories
  - `apply -f <file-or-directory>` - declarative command to apply the object definitions in the target file/directory to the cluster. `apply -R -f <file-or-directory>` -> recursive application through subdirectories
  - `cordon <node-name>` -> Node marked as unschedulable

## DNS

`<service-name>.<namespace>.svc.cluster.local`

## Status Messages

| Status | Node | Workload | Other |
| ----   | ---- | ----     | ----  |
| Ready | `True`: Healthy and Ready for Pods, `False`: Not healthy, cannot accept pods, `Unknown`: Node Controller cannot reach node | | |
| DiskPressure | `True`: Disk capacity is low | | |
| MemoryPressure | `True`: Memory is low | | |
| PIDPressure | `True`: Too many processes running | | |
| NetworkUnavailable | `True`: Network not correctly configured | | |


