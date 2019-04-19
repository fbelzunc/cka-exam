# Install kubernetes: worker node

## Add kubernetes repository

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo bash -c 'cat > /etc/apt/sources.list.d/kubernetes.list' << EOF
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
```

## Update the system

```
sudo apt-get update
```

## Install Docker

```
sudo apt-get install -y docker.io
```

## Install kubernetes-cni, kubelet, kebeadm and kubectl

```
sudo apt-get install -y kubernetes-cni=0.6.0-00
sudo apt-get install -qy kubelet=1.13.1-00 kubeadm=1.13.1-00 kubectl=1.13.1-00
```

## [Executed in master node] Capture a token 

```
sudo kubeadm token list
```

## [Executed in master node] Generate a discovery token CA cert

```
 openssl x509 -pubkey \
-in /etc/kubernetes/pki/ca.crt | openssl rsa \
-pubin -outform der 2>/dev/null | openssl dgst \
-sha256 -hex | sed 's/^.* //'
```

## Join the cluster

```
kubeadm join \
--token <MASTER_NODE_TOKEN> \
<MASTER_NODE_IP>:6443 \
--discovery-token-ca-cert-hash \
sha256:<DISCOVERY_TOKEN_CA_CERT>
```