# Pods

* For quick pod creation check https://kubernetes.io/docs/reference/kubectl/conventions/#generators

## Create a simple pod with one container running

```
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: fbelzunc
spec:
  containers:
  - image: busybox
    name: busy
    command:
      - sleep
      - "3600"
```

```
kubectl run busybox --image=busybox --generator=run-pod/v1 -- sleep 3600
```

## Create a pod which contains two containers running

```
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - image: busybox
    name: busybox-1
    command:
      - sleep
      - "3600"
  - image: busybox
    name: busybox-2
    command:
      - sleep
      - "3600"
```

## Create a pod which get deployed into nodes with the label "disktype: ssd"

```
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  labels:
    disktype: ssd
spec:
  containers:
    - image: busybox
      name: busybox
      command:
        - sleep
        -  "3600"
  nodeSelector:
    disktype: ssd
```






