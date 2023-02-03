# Kubernetes Study Guide

A collection of notes on Kubernetes to prepare for CKA

## Anatomy of an Object Manifest

### Standard Material
This basically appears in all Kubernetes objects.

```
apiVersion: // the version of the Kubernetes API which contains the type of object you’re creating
kind: // The type of object you are going to create (see above for examples)
metadata:
  name: object-name //always required, must be unique within namespace (or cluster for cluster-wide resources)
  namespace: namespace-to-create-in // optional, if not specified uses default or current context
  labels: // recommended, but optional
    app.kubernetes.io/name: name 
    app.kubernetes.io/instance: name-identifier 
    app.kubernetes.io/version: “1.2.3” 
    app.kubernetes.io/component: database // example
    app.kubernetes.io/part-of: wordpress // example
    app.kubernetes.io/managed-by: helm // example
    release: stable // more examples
    environment: dev
    tier: backend
    partition: customerA
    track: daily
  annotations:  // examples, optional
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

```
  metadata:
    labels: 
      key: value 
  spec: 
    containers:
    - name: container-name
      image: container-image:1.1.1
      command: [‘sh’, ‘-c’, ‘echo “Hello, Kubernetes!” && sleep 3600’] // examples, optional
      ports: // optional
      - containerPort: 80
  restartPolicy: OnFailure // optional, defaults to Always
```

### Specs for Common Pod Controllers
ie, Deployments, DaemonSets, Jobs

```
spec:
  replicas: 3 // Deployments, replicaSets, etc
  selector:
    matchLabels:  // examples. Used to match pods in things like deployments or services
      key: value
    matchExpressions:
      - {key: tier, operator: In, values: [backend]}
      - {key: environment, operator: NotIn, values: [prod]}    
  StrategyType: RollingUpdate/Recreate // Deployments, defaults to RollingUpdate  
  template:
      // Insert Pod Template here. Must contain labels matching selectors (may contain additional)
```


## Key Kubectl Commands

## DNS

## Status Messages
