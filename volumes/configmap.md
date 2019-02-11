# ConfigMap

* From directories: `kubectl create configmap game-config --from-file=configure-pod-container/configmap/kubectl/
* From files: `kubectl create configmap game-config-2 --from-file=configure-pod-container/configmap/kubectl/game.properties`
* From literal values: `kubectl create configmap special-config --from-literal=special.how=very --from-literal=special.type=charm`

# Define a container environment variable with data from a single ConfigMap

``` 
kubectl create configmap special-config --from-literal=special.how=very --from-literal=special.type=charm
```

```
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - image: busybox
    name: busybox
    command: ["/bin/sh","-c","env"]
    env:
      - name: SPECIAL_HOW
        valueFrom:
          configMapKeyRef:
            name: special-config
            key: special.how
      - name: SPECIAL_TYPE
        valueFrom:
          configMapKeyRef:
            name: special-config
            key: special.type
  restartPolicy: Never
 ```

 ```
 kubectl logs busybox
 ```

# Define container environment variables with data from multiple ConfigMaps

``` 
kubectl create configmap special-config --from-literal=special.how=very 
kubectl create configmap special-config --from-literal=special.type=charm 
```

```
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - image: busybox
    name: busybox
    command: ["/bin/sh","-c","env"]
    env:
      - name: SPECIAL_HOW
        valueFrom:
          configMapKeyRef:
            name: special-config
            key: special.how
      - name: SPECIAL_TYPE
        valueFrom:
          configMapKeyRef:
            name: special-config
            key: special.type
  restartPolicy: Never
 ``` 


# Configure all key-value pairs in a ConfigMap as container environment 


``` 
kubectl create configmap special-config --from-literal=special.how=very 
kubectl create configmap special-config --from-literal=special.type=charm 
```

```
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - image: busybox
    name: busybox
    command: ["/bin/sh","-c","env"]
    envFrom:
      - configMapRef:
          name: special-config
  restartPolicy: Never
 ``` 


# Use ConfigMap-defined environment variables in Pod commands

```
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - image: busybox
    name: busybox
    command: ["/bin/sh","-c","env"]
    envFrom:
      - configMapRef:
          name: special-config
  restartPolicy: Never
  ```

# Add ConfigMap data to a Volume

```
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - image: busybox
    name: busybox
    command: ["/bin/sh","-c","ls -la /cache"]
    volumeMounts:
    - name: config-volume
      mountPath: /cache
  volumes:
    - name: config-volume
      configMap:
        name: special-config
  restartPolicy: Never
  ```

# Add ConfigMap data to a specific path in the Volume

```
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - image: busybox
    name: busybox
    command: ["/bin/sh","-c","ls -la /cache"]
    volumeMounts:
    - name: config-volume
      mountPath: /cache
  volumes:
    - name: config-volume
      configMap:
        name: special-config
        items:
        - key: special.type
          path: keys
  restartPolicy: Never
```


