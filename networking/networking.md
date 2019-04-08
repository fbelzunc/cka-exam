# Networking

## [Step 1] Deploy Jenkins OSS as a statefulset

```
apiVersion: "apps/v1"
kind: "StatefulSet"
metadata:
  name: jenkins
  labels:
    com.example.ci.type: jenkins
spec:
  selector:
    matchLabels:
      com.example.ci.type: jenkins
  serviceName: jenkins
  replicas: 1
  template:
    metadata:
      name: jenkins
      labels:
        com.example.ci.type: jenkins
    spec:
      containers:
      - name: jenkins
        image: jenkins/jenkins
        env:
        - name: ENVIRONMENT
          value: KUBERNETES
        - name: JENKINS_OPTS
          value: --prefix=/jenkins
        - name: JAVA_OPTS
          value: >-
            -XshowSettings:vm
            -XX:MaxRAMFraction=1
            -XX:+UnlockExperimentalVMOptions
            -XX:+UseCGroupMemoryLimitForHeap
            -XX:+PrintGCDetails
        ports:
        - containerPort: 8080
        - containerPort: 50000
        resources:
          limits:
            cpu: "1"
            memory: "1G"
        volumeMounts:
        - name: jenkins-home
          mountPath: /var/jenkins_home
      securityContext:
        fsGroup: 1000
      volumes:
      - name: jenkins-home
        emptyDir: {}
```
## [Step 2] Make Jenkins OSS accesible through a NodePort

```
apiVersion: v1
kind: Service
metadata:
  name: jenkins-nodeport
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    com.example.ci.type: jenkins
  type: NodePort
```

## [Step 3] Make the Jenkins OSS accesible through an ingress-nginx

* See https://github.com/kubernetes/ingress-nginx/blob/master/docs/deploy/index.md


```
export NGINX_BRANCH=https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy
```

```
kubectl apply -f $NGINX_BRANCH/mandatory.yaml
``` 

```
kubectl apply -f $NGINX_BRANCH/provider/aws/service-l4.yaml
```

```
kubectl apply -f $NGINX_BRANCH/provider/aws/patch-configmap-l4.yaml
```

```
kubectl get all --all-namespaces -l app=ingress-nginx -o wide
``` 

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: jenkins
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/app-root: "http://fbelzunc.support.bees.com/jenkins"
    nginx.ingress.kubernetes.io/proxy-body-size: 50m
    nginx.ingress.kubernetes.io/proxy-request-buffering: "off"
spec:
  # To enable SSL offloading at ingress level, uncomment the following 5 lines
  #tls:
  #- hosts:
  #  - cloudbees-core.example.com
  #  # Name of the secret containing the certificate to be used
  #  secretName: cje-example-com-tls
  rules:
  - http:
      paths:
      - path: /jenkins
        backend:
          serviceName: jenkins
          servicePort: 8080
    host: fbelzunc.support.bees.com
```

## [Step 3] Make the Jenkins OSS accesible through a load balancer


```
kind: Service
apiVersion: v1
metadata:
  name: jenkins-loadbalancer
spec:
  selector:
    com.example.ci.type: jenkins
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: LoadBalancer
```