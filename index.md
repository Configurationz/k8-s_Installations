
![Preview](./imgs/realk8(Custom).png)

kubernetes Installation steps for both Master & Nodes 
------------------------------------------------------

* Create few nodes (>1)
  * Install docker on each node
  * Install CRI-dockerd on each node
  * Install kubeadm on each node
  * Make one node Master/Control Plane
  * Add all other nodes to the cluster
  * Install any CNI implementation _[Flannel](https://github.com/flannel-io/flannel/releases/latest/download/)_

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
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.4/cri-dockerd_0.3.4.3-0.ubuntu-focal_amd64.deb
sudo dpkg -i cri-dockerd_0.3.4.3-0.ubuntu-focal_amd64.deb
```

3. Install 'go' by executing below commands on all the nodes
```bash
sudo -i
wget https://go.dev/dl/go1.21.1.linux-amd64.tar.gz && tar -C /usr/local -xzf go1.21.1.linux-amd64.tar.gz
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

4. Next, install the following components _**(kubelet, kubeadm & kubectl)**_ on all the nodes in the cluster
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
```
echo ‘deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /’ | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
exit
exit
```

5. Relogin & initialize the cluster using the following command as a root user [Making one node master/control plane]
```bash
sudo -i
kubeadm init –pod-network-cidr=10.244.0.0/16 --cri-socket=unix:///var/run/cri-dockerd.sock
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

9. Let's use flannel, Execute the following on master node
```
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

10.  Check if the status of all the nodes including master is ready
```
kubectl get nodes -w 
```

11.  Autocomplete kubectl commands in command line (kubectl cheat sheet) 
```bash
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
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
kubectl delete-f xyz.yaml
```

![Preview](./imgs/partyk8s.gif)
