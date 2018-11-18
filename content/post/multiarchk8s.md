+++
date = "2017-05-13T15:49:08+08:00"
tags = ["raspberrypi","kubernetes"]
title = "Setting up a Multi-Architecture Kubernetes Cluster"

+++

# Introduction
This post is based on the great work the Hypriot guys did (https://blog.hypriot.com/post/setup-kubernetes-raspberry-pi-cluster/).

I extend their architecture to different architectures and address some of the issues with this.

Deployment:

| Servername |      IP      | Architecture |      OS    |  Description   |
|------------|--------------|:------------:|:----------:|:--------------:|
|   master   | 192.168.1.1  |     x86      |   Ubuntu   | HP MicroServer |
|   node0    | 192.168.1.10 |    ARMV7     |  HypriotOS | Raspberry Pi 3 |
|   node1    | 192.168.1.11 |    ARMV6     |  HypriotOS | Raspberry Pi 2 |

***

# Setup X86 Node (master)

## Install kubeadm
```
sudo su -
apt-get update && apt-get install -y kubeadm
```
## Bootstrap the kubernetes cluster
```
kubeadm init --pod-network-cidr 10.244.0.0/16
```
**IMPORTANT:** Please note down the kubeadm join command that was output from the step above. This is the exact command you will run on each of the slave nodes later on.

Validate your Deployment by running as a normal user:
```
sudo cp /etc/kubernetes/admin.conf $HOME/
sudo chown $(id -u):$(id -g) $HOME/admin.conf
export KUBECONFIG=$HOME/admin.conf

kubectl get nodes
```

Remove taints from the master node
```
kubectl taint nodes --all node-role.kubernetes.io/master-
```

## Setup the pod network driver
Install flannel and validate you got the pods up and running correctly:
```
kubectl create -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel-rbac.yml
kubectl create -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl --namespace=kube-system get pods
```

Test single node by deploying the hello minikube sample:
```
kubectl run hello-minikube --image=gcr.io/google_containers/echoserver:1.4 --port=8080
kubectl expose deployment hello-minikube --type=NodePort
kubectl get pod
kubectl get svc hello-minikube
curl 10.111.29.69:8080
```
You should expect the following output:

![curl output](/blog/img/curlminikube.png "curl output")

## Setup the cluster load balancer
Install Traefik as the load balancer for our cluster. Please note that we have to use the rbac yaml file aswe are running  kubernetes 1.6+.
```
kubectl apply -f https://raw.githubusercontent.com/containous/traefik/master/examples/k8s/traefik-with-rbac.yaml
```

## Install the Kubernetes Dashboard

```
$ kubectl create -f https://git.io/kube-dashboard
```
Test that you get the kubernetes dashboard page by acessing http://localhost:8001/ui on your browser.

You should see something like:

![k8s dashboard](/blog/img/k8sdash.png "k8s dashboard")

***

# Setup ARM Nodes (slaves)
Follow the process below for each one od the slave nodes (node0 and node1).

 **NOTICE:** *Please start by flashing both Raspberry pi's with the HypriotOS and configured them with the above hostnames and IP addresses.*

## Install kubeadm

```
sudo su -
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
apt-get update && apt-get install -y kubeadm
```

Remove KUBELET_NETWORK_ARGS from the kubeadm conf file
```
nano /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```
make sure the variable is no longer present in the current session.

## Pull and tag the kube-proxy container for arm

This step is an hack required at the moment because the deployment of kube-proxy does not natively support mutli-architeture and thus, the nodes assume to be amd64 and try to get  the amd64 containter (like their master) instead of the required arm version.
```
docker pull gcr.io/google_containers/kube-proxy-arm:v1.6.2 
docker tag gcr.io/google_containers/kube-proxy-arm:v1.6.2 gcr.io/google_containers/kube-proxy-amd64:v1.6.2
```

check that the right images are in the repository:
```
docker images
```

## Join the cluster
This command is the exact same one that was printed on the command line when we run kubeadm init on the master.

An example would be:
```
kubeadm join --token sometoken  192.168.1.1:6443
```
***

# Validate Cluster Deployment
Validate that both nodes were added correctly by running  on the master server:

```
kubectl get nodes
```

Validate all nodes have STATUS Ready:

![All Nodes Ready](/blog/img/getnodes.png "All Nodes Ready")

***

# Reset Environment
On the master server
```
kubectl drain node1 --delete-local-data --force --ignore-daemonsets
kubectl delete node node1

kubectl drain node0 --delete-local-data --force --ignore-daemonsets
kubectl delete node node0
```

On each of the slave nodes
```
sudo su -
kubeadm reset
rm -rf .kube/
rm -rf /var/lib/cni
rm -rf /run/flannel
rm -rf /etc/cni
```

Back on the master node
```
sudo su -
kubeadm reset
rm -rf .kube/
rm -rf /var/lib/cni
rm -rf /run/flannel
rm -rf /etc/cni
```

***

# Useful DEBUG commands
```
sudo journalctl -xn -u kubelet.service
kubectl get events
/etc/systemd/system/kubelet.service.d/10-kubeadm.conf

kubectl --namespace=kube-system get pods
kubectl get pods --show-labels
kubectl get events
kubectl describe node bilhimserver

kubectl --namespace=kube-system logs kube-flannel-ds-cpv9b kube-flannel
```

***

# Resources
Hypriot raspberry pi kubernetes cluster setup - https://blog.hypriot.com/post/setup-kubernetes-raspberry-pi-cluster/

Kubernetes Dashboard Deployment - https://github.com/kubernetes/dashboard#kubernetes-dashboard

Flannel - https://github.com/coreos/flannel

Traefik - https://docs.traefik.io/user-guide/kubernetes/
