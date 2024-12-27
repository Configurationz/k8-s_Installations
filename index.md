
kubernetes Installation in ubuntu 20.04 for both Master & Nodes 
---------------------------------------------------------------

### Pre-requisites:
  * Create few nodes (>1)
  * Install docker on each node
  * Install CRI-dockerd on each node
  * Install kubeadm on each node
  * Make one node Master/Control Plane
  * Add all other nodes to the cluster
  * Install any CNI implementation _[Flannel](https://github.com/flannel-io/flannel/releases/latest/download/)_

### Configuration:

1. Install Docker on all the nodes
```bash
sudo apt update
curl -fsSL https://get.docker.com -o install-docker.sh && sh install-docker.sh
sudo usermod -aG docker ubuntu
exit
```

2. Install CRI-dockerd, below steps are specific to ubuntu 20.04
_[Refer Here](https://github.com/Mirantis/cri-dockerd/releases)_
```
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.16/cri-dockerd_0.3.16.3-0.ubuntu-focal_amd64.deb
sudo dpkg -i cri-dockerd_0.3.16.3-0.ubuntu-focal_amd64.deb
```

3. Install _[go](https://go.dev/doc/install)_ by executing below commands on all the nodes _[go Installations]( https://github.com/Mirantis/cri-dockerd)_
```bash
sudo -i
wget https://go.dev/dl/go1.23.4.linux-amd64.tar.gz && tar -C /usr/local -xzf go1.23.4.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
git clone https://github.com/Mirantis/cri-dockerd.git
cd cri-dockerd
mkdir bin
go build -o bin/cri-dockerd
mkdir -p /usr/local/bin
install -o root -g root -m 0755 bin/cri-dockerd /usr/local/bin/cri-dockerd
cp -a packaging/systemd/* /etc/systemd/system
sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
systemctl daemon-reload
systemctl enable cri-docker.service
systemctl enable --now cri-docker.socket
```

4. Next, install the following components _[kubeadm, kubelet and kubectl](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl)_ on all the nodes in the cluster being a root user
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
```
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
exit
exit
```

5. Relogin & initialize the cluster using the following command as a root user making one node _[Master/control-plane](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)_ to create a cluster, login into a Master node and execute the following
```bash
sudo -i
kubeadm init --pod-network-cidr=10.244.0.0/16 --cri-socket=unix:///var/run/cri-dockerd.sock
```

```
OUTPUT:

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.31.25.16:6443 --token sz14lp.jwkx2vy49w54fk79 \
        --discovery-token-ca-cert-hash sha256:25fe0576979b9306d911139f22c47f02240ab63731619a745b0e396ddf9fbe46
```

6. On the master node, to run kubectl as a normal user, execute the following commands as non-root user
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

7. Check for all the available nodes
```
kubectl get nodes
```

8. kuberentes needs CNI Plugin so that pod-network is enabled. Untill this is done, the DNS doesn't work, services donot work so the status of the nodes shows 'NotReady'. Install any CNI implementation _(Flannel)_

9. For k8s networking, the reference specification is CNI. There are many vendors who implement this _[Addons Installations](https://kubernetes.io/docs/concepts/cluster-administration/addons/)_. We are going to use _[flannel](https://github.com/flannel-io/flannel#deploying-flannel-manually)_  Let's execute the following on master node being a normal user
```
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

10.  Check if the status of all the nodes including master is ready
```
kubectl get nodes -w 
```

11.  Autocomplete kubectl commands in command line using _[kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)_
```bash
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

12. After the successful installation of flannel on Master/control-plane, execute the following command on all other nodes as a root user allowing them to join the cluster
```
kubeadm join 172.31.25.16:6443 --token sz14lp.jwkx2vy49w54fk79 \
        --discovery-token-ca-cert-hash sha256:25fe0576979b9306d911139f22c47f02240ab63731619a745b0e396ddf9fbe46 --cri-socket "unix:///var/run/cri-dockerd.sock"
``` 

* Some useful commands –

```
kubectl get no
```
```
kubectl get nodes
```
```
kubectl get nodes -o wide
```
```
kubectl api-resources
```
```
kubectl api-resources | grep pod
```
```
kubectl get pods -A w
```
```
kubectl apply -f xyz.yaml
```
```
kubectl get pods -o wide
```
```
kubectl get pods <pod-name> -o yaml
```
```
kubectl describe pods <pod-name>
```
```
kubectl delete -f xyz.yaml
```
```
kubectl delete pods <pod-name>
```
```
kubectl get po -n kube-system -w
```
```
kubectl logs <pod-name>
```

### References ~

_**[Kubernetes Networking Model](https://sookocheff.com/post/kubernetes/understanding-kubernetes-networking-model/){:target="_blank"}**_

_**[Imperative Commands](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#run){:target="_blank"}**_

<p align="center">
      <img src="imgs/output.gif" align="left">
      <img src="imgs/transparent-cw.gif">
      <img src="imgs/new-k8s-anti-cw.gif" align="right">
</p>

<!-- <p style="text-align: center"><img src="./imgs/k8s-cw.gif"></p> -->