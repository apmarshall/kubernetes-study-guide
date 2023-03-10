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
    initContainers: # example, optional
    - name: init-myservice
      image: initcontainer:image:1.2.3
      command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
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

Node objects also contain a `status` section, with fields for `conditions`, `addresses`, `capacity`, and `info.` These are set and maintained by the kubelet and node controller automatically. Some fields can be overwritten using the kubelet (ie, `hostName`). Manually registered (as opposed to self-registered) nodes need their capacity information set when added.

## Kubectl 

Synopsis:

```
kubectl <command> <args> <filters> [--namespace=namespace-name] [--all-namespaces]
```

Key Commands:
  - `create <object-name>` - imperative command to create an object
    - common arguments:
      - `--image <image-name>`
  - `create -f <file-name>.yml` - imperative command to creat objects from a file
  - `diff -f <file-or-directory>` - declarative command to compare the contents of the target file/directory to the current cluster state. `diff -R -f <file-or-directory>` -> recursive comparison through subdirectories
  - `apply -f <file-or-directory>` - declarative command to apply the object definitions in the target file/directory to the cluster. `apply -R -f <file-or-directory>` -> recursive application through subdirectories
  - `get <type>` - lists available resources by specified type (`pods`, `deployments`, `services`, etc)
  - `describe <type>(/<object>)` - gets the details of all objects of a type or a specified object of a type
  - `rollout status <type>/<object>` - checks the status of an update to the specified object
  - `rollout history <type>/<object>` - describes the rollout history of the specified object
  - `cordon <node-name>` -> Node marked as unschedulable
  
Filters:
  - `-l` = label-filters:
    - equality-based: `-l key=value,key!=value`
    - set-based: `-l 'key in (value1, value2),key notin (value)'`
  - `--field-selector type.key=value` -> filter on fields other than labels (field must exist, or command will error)

Popular Args/Options:
  - `-o yaml` - Generate the output (usually of `describe` or `create`) in yaml. Direct to a file with `> filename` at the end to get a draft of a resrouce

Other imperative commands:
  - `set image <type>/<object> <container-name>=<image>:<version>` - updates the image for the specified container in the specified object
  - `rollout undo <type>/<object>` - reverts to the previous version in an objects rollout history

Object Type Aliases:
  - rs = replicaSet
  - deploy = deployment

## DNS

`<service-name>.<namespace>.svc.cluster.local`

## Status Messages

| Condition/Status | Node | Pod/Pod Controller | Other |
| ----   | ---- | ----     | ----  |
| Ready | `True`: Healthy and Ready for Pods, `False`: Not healthy, cannot accept pods, `Unknown`: Node Controller cannot reach node | `True`: All containers in the Pod are ready && All conditions specified in `readinessGates` are met. Pod can be added to load balancing pools for matching services | |
| Complete | | `True`: All of a Deployments ReplicaSets are up-to-date, all replicas are available, no old replicas are running | |
| ContainersReady | | `True`: All containers in the pod are ready | |
| Scheduled | | `PodScheduled = True` -> Pod has been scheduled to a node | |
| Initialized | | `True`: All init containers have completed successfully | |
| Progressing | | `True`: A deployment has created/scaled a replicaSet OR new pods in a deployment have become ready; `False`: Deployment unable to complete its ReplicaSet | |
| DiskPressure | `True`: Disk capacity is low | | |
| MemoryPressure | `True`: Memory is low | | |
| PIDPressure | `True`: Too many processes running | | |
| NetworkUnavailable | `True`: Network not correctly configured | | |
| PodHasNetwork | | `True`: Pod sandbox has been successfully created and networking configured | |

## Security

- Need a method of authentication and authorization to the API server
- Decide if anonymous requests to API server will be allowed
- Decide if service account tokens will be allowed
- Need a means of certificate bootstrapping and distribution -> Nodes provisioned with public root cert for cluster, client credentials for kubelet = client cert
- Service accounts -> secure communication between pods and API server
- Either:
  - Turn on verification of the kubelet’s serving certificate
  - Use SSH tunneling between the API server and the kubelet
- Enable kubelet authentication/authorization
- Do not run connections between API server and nodes/pods/services over the open internet/untrusted networks (no authN/Z or verification possible, use something like SSH tunneling or the Konnectivity service)
