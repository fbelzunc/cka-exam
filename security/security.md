# Security

## Know how to configure authentication and authorization.

## Understand Kubernetes security primitives.

Reference: https://kubernetes.io/docs/concepts/policy/pod-security-policy/

### Create a policy that simply prevents the creation of privileged pods.

The first thing to do is to add `PodSecurityPolicy` to the `admission-plugins`.

```
sudo vim /etc/kubernetes/manifests/kube-apiserver.yaml

# Add PodSecurityPolicy to the --enable-admission-plugins
# kube-apiserver will be automatically restarted
--enable-admission-plugins=NodeRestriction,PodSecurityPolicy
``` 

Then, we create a serviceaccount for a user providing edit permission in the namespace

```
kubectl create namespace fbelzunc
namespace/fbelzunc created

kubectl create serviceaccount fake-user -n emea-support
serviceaccount/fake-user created

kubectl create rolebinding -n fbelzunc fake-editor --clusterrole=edit --serviceaccount=fbelzunc:fake-user
rolebinding.rbac.authorization.k8s.io/fake-editor created
````

To perform the tests we will create two alias:

```
alias kubectl-admin='kubectl -n fbelzunc'
alias kubectl-user='kubectl --as=system:serviceaccount:fbelzunc:fake-user -n fbelzunc'
```

Now, we can check that `fake-user` only have permission to operate in the `fbelzunc` namespace

```
kubectl-user -n fbelzunc get pods --all-namespaces
Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:fbelzunc:fake-user" cannot list resource "pods" in API group "" at the cluster scope

kubectl-user -n fbelzunc get pods
No resources found.

kubectl-user run busybox --image=busybox --generator=run-pod/v1 -- sleep 3600
Error from server (Forbidden): pods "busybox" is forbidden: unable to validate against any pod security policy: []
```

Create the `PodSecurityPolicy`

````
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: my-pod-security-policy
spec:
  privileged: false
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  runAsUser:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  volumes:
  - '*'
  ```

```
kubectl create -f pod-security-policy.yaml
```

Allow to use the `PodSecurityPolicy` to the `fake-user`

```
kubectl create role -n fbelzunc fbelzunc:unprivileged --verb=use --resource=podsecuritypolicy --resource-name=my-pod-security-policy
role.rbac.authorization.k8s.io/fbelzunc:unprivileged created

kubectl create rolebinding fake-user:fbelzunc:unprivileged -n fbelzunc --role=fbelzunc:unprivileged --serviceaccount=fbelzunc:fake-user
rolebinding.rbac.authorization.k8s.io/fake-user:fbelzunc:unprivileged created
```

Now, we test that we can create the a simple pod

```
kubectl-user run -n fbelzunc busybox --image=busybox --generator=run-pod/v1 -- sleep 3600
pod/busybox created
```

But, we can't create a privileged pod

```
kubectl create -n fbelzunc -f- <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - name: busybox
    image: busybox
    securityContext:
      privileged: true
EOF
Error from server (Forbidden): error when creating "STDIN": pods "busybox" is forbidden: unable to validate against any pod security policy: [spec.containers[0].securityContext.privileged: Invalid value: true: Privileged containers are not allowed]
```

## Know to configure network policies.

References: 
* https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/
* https://kubernetes.io/docs/concepts/services-networking/network-policies/

### Create a networking policy such that only pods with the label `access: true` can querry the nginx service

For this lab we need to use the image `busybox:1.28` as `busybox` latest images currently has a bug which makes `nslookup` not to work correctly.

* Create a pod

```
kubectl run nginx --image=nginx --replicas=2 --generator=run-pod/v1
```

* Expose the service

```
kubectl expose pod nginx --port=80 --name=nginx
service/nginx exposed
```

* Test the service from another pod

```
kubectl run busybox --image=busybox:1.28 --generator=run-pod/v1 -- sleep 3600
pod/busybox created

kubectl exec -it busybox -- nslookup nginx
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      nginx
Address 1: 10.107.11.239 nginx.default.svc.cluster.local
```

* Limit the access to the nginx service

```
kubectl get pods -o wide --show-labels
NAME      READY   STATUS    RESTARTS   AGE     IP            NODE               NOMINATED NODE   READINESS GATES   LABELS
busybox   1/1     Running   0          2m40s   192.168.1.9   ip-172-31-71-140   <none>           <none>            run=busybox
nginx     1/1     Running   0          7m26s   192.168.1.8   ip-172-31-71-140   <none>           <none>            run=nginx
```

```
kubectl exec -it busybox -- ping -c 1 192.168.1.8
PING 192.168.1.8 (192.168.1.8): 56 data bytes
64 bytes from 192.168.1.8: seq=0 ttl=63 time=0.165 ms

--- 192.168.1.8 ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 0.165/0.165/0.165 ms
```

```
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: access-nginx
spec:
  podSelector:
    matchLabels:
      run: nginx
  ingress:
  - from:
    - podSelector:
        matchLabels:
          access: "true"
```

```
kubectl apply -f networkpolicy.md
networkpolicy.networking.k8s.io/access-nginx created
```

```
kubectl exec -it busybox -- ping -c 1 192.168.1.8
PING 192.168.1.8 (192.168.1.8): 56 data bytes

--- 192.168.1.8 ping statistics ---
1 packets transmitted, 0 packets received, 100% packet loss
command terminated with exit code 1

kubectl exec -it busybox -- wget --spider --timeout=1 nginx
Connecting to nginx (10.107.11.239:80)
wget: download timed out
command terminated with exit code 1
```

## Create and manage TLS certificates for cluster components.

Reference: https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/

## Work with images securely.

## Define security contexts.

Reference: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/

### Define a pod in which all containers will be running with: `runAsUser: 1000`, `runAsGroup: 3000` and `fsGroup: 2000`

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: busybox
  name: busybox
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - image: busybox
    name: busybox
    command: ["sh","-c", "sleep 3600"]
    volumeMounts:
      - name: my-local-volume
        mountPath: /data
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
  - name: my-local-volume
    emptyDir: {}
status: {}
```

```
kubectl exec -it busybox -- id
uid=1000 gid=0(root) groups=2000
```

## Secure persistent key value store.

## Work with role-based access control.

Reference: https://kubernetes.io/docs/reference/access-authn-authz/rbac/

### Create a Role named “pod-reader” that allows user to perform “get”, “watch” and “list” on pods in a specific namespace

```
# Create the serviceaccount fake-user
kubectl -n fbelzunc create serviceaccount fake-user

# Create the role pod-reader
kubectl -n fbelzunc create role pod-reader --verb=get --verb=list --verb=watch --resource=pods

# Create the rolebinding pod-reader:fake-user
kubectl -n fbelzunc create rolebinding pod-reader:fake-user --serviceaccount=fbelzunc:fake-user --role=pod-reader
```

* Test that we can't create a pod with `kubectl-user`

```
kubectl-user -n fbelzunc run busybox --image=busybox --generator=run-pod/v1 -- sleep 3600
Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:fbelzunc:fake-user" cannot create resource "pods" in API group "" in the namespace "fbelzunc"
```

* Test that we can read a pod with `kubectl-user` 

```
# Create a pod with admin user
kubectl -n fbelzunc run busybox --image=busybox --generator=run-pod/v1 -- sleep 3600

# Read the pods inside the fbelzunc namespace with the kubectl-user
kubectl-user get pod -n fbelzunc
NAME      READY   STATUS    RESTARTS   AGE
busybox   1/1     Running   0          2m52s
```

### Create a Role named “pod-reader” that allows user to perform “get”, “watch” and “list” on pods in all namespaces

AFAIK Th ebelow does not make sense at all because serviceaccounts are always bound to a namespace

```
# Create the serviceaccount fake-user
kubectl create serviceaccount fake-user

# Create the corresponded clusterrole
kubectl create clusterrole pod-reader --verb=get,list,watch --resource=pods

# Create a rolebinding
kubectl create rolebinding fake-user:pod-reader --clusterrole=pod-reader --serviceaccount=default:fake-user
```

```
alias kubectl-user='kubectl --as=system:serviceaccount:default:fake-user'
```

### Across the entire cluster, grant the permissions in the cluster-admin ClusterRole to a user named “root”:

### Grant a role to all service accounts in a namespace


### Grant super-user access to all service accounts cluster-wide (strongly discouraged)

