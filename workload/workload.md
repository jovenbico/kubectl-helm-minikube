## Creating a Deployment

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

* Create a Deployment to rollout a ReplicaSet.
```BASH
kubectl apply -f deployment/nginx-deployment.yaml
```

* Declare the new state of the Pods
```BASH
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
```
* Rollback to an earlier Deployment revision
```BASH
# First, check the revisions of this Deployment
kubectl rollout history deployment/nginx-deployment
# To see the details of each revision, run 
kubectl rollout history deployment/nginx-deployment --revision=2

# Rolling Back
kubectl rollout undo deployment/nginx-deployment
```
* Scale up the Deployment to facilitate more load.
```BASH
kubectl scale deployment/nginx-deployment --replicas=10
```

* Pause the rollout of a Deployment
* Use the status of the Deployment
* Clean up older ReplicaSets 

#

## Replicaset, when to use
A ReplicaSet ensures that a specified number of pod replicas are running at any given time. However, a Deployment is a higher-level concept that manages ReplicaSets and provides declarative updates to Pods along with a lot of other useful features.

```BASH
kubectl apply -f replicaset/frontend.yaml
```
```YAML
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
```

* Non-Template Pod acquisitions
```BASH
kubectl apply -f replicaset/pod-rs.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    tier: frontend
spec:
  containers:
  - name: hello1
    image: gcr.io/google-samples/hello-app:2.0

---

apiVersion: v1
kind: Pod
metadata:
  name: pod2
  labels:
    tier: frontend
spec:
  containers:
  - name: hello2
    image: gcr.io/google-samples/hello-app:1.0
```

#

## StatefulSets
StatefulSet is the workload API object used to manage stateful applications.

Using StatefulSets
* Stable, unique network identifiers.
* Stable, persistent storage.
* Ordered, graceful deployment and scaling.
* Ordered, automated rolling updates.

Limitations
* The storage for a given Pod must either be provisioned by a `PersistentVolume Provisioner`
* Deleting and/or scaling a StatefulSet down will `NOT` delete the volumes associated with the StatefulSet.
* StatefulSets currently require a `Headless Service` to be responsible for the network identity of the Pods.
* StatefulSets do not provide any guarantees on the termination of pods when a StatefulSet is deleted.
* When using `Rolling Updates` with the default Pod Management Policy (`OrderedReady`),

The example below demonstrates the components of a StatefulSet.

```BASH
kubectl apply -f statefulsets/pv.yaml
kubectl apply -f statefulsets/statefulsets.yaml
```

```YAML
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None # Headless Service
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  minReadySeconds: 10 # by default is 0
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: registry.k8s.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class" # provide stable storage using PersistentVolumes
      resources:
        requests:
          storage: 1Gi
```
#

## DaemonSet
A `DaemonSet` ensures that all (or some) Nodes run a copy of a Pod. 

```BASH
kubectl apply -f daemonsets/fluentd.yaml
```
```YAML
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # these tolerations are to have the daemonset runnable on control plane nodes
      # remove them if your control plane nodes should not run pods
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```