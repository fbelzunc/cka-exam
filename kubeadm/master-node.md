# Install kubernetes: master node

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

## Init the cluster

```
kubeadm init \ 
--kubernetes-version 1.13.1 \ 
--pod-network-cidr 192.168.0.0/16
```

## Install Calico

```
kubectl create -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
kubectl create -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
```

## Allow non root user to administer the cluster 

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## kubectl auto-completation

```
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
```
