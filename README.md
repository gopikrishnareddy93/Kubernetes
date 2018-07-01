### Kubernetes setup using Kubeadm

# Introduction
- Kubernetes is a container orchestration system that manages containers at scale. Initially developed by Google based on its experience running containers in production, Kubernetes is open source and actively developed by a community around the world.

- Kubeadm automates the installation and configuration of Kubernetes components such as the API server, Controller Manager, and Kube DNS. It does not, however, create users or handle the installation of operating system level dependencies and their configuration. Using these tools makes creating additional clusters or recreating existing clusters much simpler and less error prone.

# Prepare the VM

- Change to root:
```sh
sudo su
```
- Turn off swap: To do this, you will first need to turn it off directly …
```sh
swapoff -a
```
- To disable swap permanently comment out the reference to swap in /etc/fstab. 
- Install updates
```sh
apt update && apt upgrade
```
# Install Docker
- Remove old docker installations
```sh
apt-get remove docker docker-engine docker.io
```
- Install docker prerequisites

```sh
apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

apt-key fingerprint 0EBFCD88

add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
   
apt-get update
```

- Pick docker version compatible with kubernetes from the list given by the below command
```sh
apt-cache madison docker-ce
```
- The validated docker versions for kubernetes v1.10.5 are the same as for v1.9: 1.11.2 to 1.13.1 and 17.03.x 

```sh
apt-get install docker-ce=17.03.2~ce-0~ubuntu-xenial
```

# Install kubeadm, kubectl and kubeproxy

```sh
apt-get update \
  && apt-get install -y apt-transport-https \
  && curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" \
  | tee -a /etc/apt/sources.list.d/kubernetes.list \
  && apt-get update 
  
  
apt-get update \
  && apt-get install -y \
  kubelet \
  kubeadm \
  kubernetes-cni
```

# Cluster initialization 
 > Replace Ip address in the below command

```sh
kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=174.138.15.158
```

- Make a note of join token generated after cluster initialization 

- To start using your cluster, you need to run the following as a regular user:

```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

# Networking

- This is a place, when things start to be more complicated. Kubernetes is built with extensibility in mind, this is why there are always a lot of options. If you are not sure which network addon to use, use flannel. Flannel is a very simple overlay network that satisfies the Kubernetes requirements. Many people have reported success with Flannel and Kubernetes.

`kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel-rbac.yml`

`kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml`

# Join your nodes to your Kubernetes cluster

- Repeat the above steps until cluster initialization
- Execute join command that is obtained after cluster initialization on worker node 

```sh
kubeadm join --token 702ff6.bc7aacff7aacab17 174.138.15.158:6443 --discovery-token-ca-cert-hash sha256:68bc22d2c631800fd358a6d7e3998e598deb2980ee613b3c2f1da8978960c8ab
```
# References:
  
  1. https://docs.docker.com/install/linux/docker-ce/ubuntu/
  2. https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.10.md#v1105  