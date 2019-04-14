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
kubectl create namespace emea-support

kubectl create serviceaccount fake-user -n emea-support

kubectl create rolebinding fake-user:edit --clusterrole=edit --serviceaccount=emea-support:fake-user -n emea-support
````

Create the `PodSecurityPolicy`

````
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: my-pod-security-policy
spec:
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

Allow to use the `PodSecurityPolicy` to the `fake-user`

```
kubectl create role unprivileged --verb=use --resource=podsecuritypolicy --resource-name=my-pod-security-policy -n emea-support

kubectl create rolebinding unprivileged:fake-user --role=unprivileged --serviceaccount=emea-support:fake-user -n emea-support 
```

To test a pod creation with priviled/unprivileged mode we can create an alias.

```
alias kubectl-user='kubectl --as=system:serviceaccount:emea-support:fake-user -n emea-support'
```

###  Create a networking policy such that only pods with the label access=granted can talk to it

* Create an nginx pod and attach this policy to it
* Create a busybox pod and attempt to talk to nginx - should be blocked
* Attach the label to busybox and try again - should be allowed

### Create a policy that does not allow emptyDir 

## Know to configure network policies.

## Create and manage TLS certificates for cluster components.

## Work with images securely.

## Define security contexts.

## Secure persistent key value store.

## Work with role-based access control.