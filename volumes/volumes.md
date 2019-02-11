# Volumes

## Create a pod which mounts a local directory in the pod. The local directory should be automatically removed after the pod stop running.

```
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - image: busybox
    name: busybox
    command:
      - sleep
      - "3600"
    volumeMounts:
    - mountPath: /cache
      name: local-volume
  volumes:
    - name: local-volume
      emptyDir: {}
```

CHeck that the `/cache` folder was created locally

```
$ kubectl exec -it busybox -- ls
bin    cache  dev    etc    home   proc   root   sys    tmp    usr    var
```


