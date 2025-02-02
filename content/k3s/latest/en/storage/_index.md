---
title: "Volumes and Storage"
weight: 30
---

When deploying an application that needs to retain data, you’ll need to create persistent storage. Persistent storage allows you to store application data external from the pod running your application. This storage practice allows you to maintain application data, even if the application’s pod fails.

# Local Storage Provider
k3s comes with Rancher's Local Path Provisioner and this enables the ability to create persistent volume claims out of the box using local storage on the respective node. Below we cover a simple example. For more information please reference the official documentation [here](https://github.com/rancher/local-path-provisioner/blob/master/README.md#usage).

Create a hostPath backed persistent volume claim and a pod to utilize it:

### pvc.yaml

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-path-pvc
  namespace: default
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-path
  resources:
    requests:
      storage: 2Gi
```

### pod.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: volume-test
  namespace: default
spec:
  containers:
  - name: volume-test
    image: nginx:stable-alpine
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: volv
      mountPath: /data
    ports:
    - containerPort: 80
  volumes:
  - name: volv
    persistentVolumeClaim:
      claimName: local-path-pvc
```

Apply the yaml `kubectl create -f pvc.yaml` and `kubectl create -f pod.yaml`

Confirm the PV and PVC are created. `kubectl get pv` and `kubectl get pvc` The status should be Bound for each.

# Longhorn

[comment]: <> (pending change - longhorn may support arm64 and armhf in the future.)

> **Note:** At this time Longhorn only supports amd64.

k3s supports [Longhorn](https://github.com/longhorn/longhorn). Below we cover a simple example. For more information please reference the official documentation [here](https://github.com/longhorn/longhorn/blob/master/README.md).

Apply the longhorn.yaml to install Longhorn.

```
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/master/deploy/longhorn.yaml
```

Longhorn will be installed in the namespace `longhorn-system`.

Before we create a PVC, we will create a storage class for longhorn with this yaml.

```
kubectl create -f https://raw.githubusercontent.com/longhorn/longhorn/master/examples/storageclass.yaml
```

Now, apply the following yaml to create the PVC and pod with `kubectl create -f pvc.yaml` and `kubectl create -f pod.yaml`

### pvc.yaml

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: longhorn-volv-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: longhorn
  resources:
    requests:
      storage: 2Gi
```

### pod.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: volume-test
  namespace: default
spec:
  containers:
  - name: volume-test
    image: nginx:stable-alpine
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: volv
      mountPath: /data
    ports:
    - containerPort: 80
  volumes:
  - name: volv
    persistentVolumeClaim:
      claimName: longhorn-volv-pvc
```

Confirm the PV and PVC are created. `kubectl get pv` and `kubectl get pvc` The status should be Bound for each.
