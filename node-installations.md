Node Installation steps
-----------------------

1. Install Docker on all the nodes
```bash
sudo apt update
curl -fsSL https://get.docker.com -o install-docker.sh && sh install-docker.sh
sudo usermod -aG docker ubuntu
exit
```

2. Install CRI-dockerd, below steps are specific to ubuntu 20.04
```bash
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.4/cri-dockerd_0.3.4.3-0.ubuntu-focal_amd64.deb
sudo dpkg -i cri-dockerd_0.3.4.3-0.ubuntu-focal_amd64.deb
```

3. Next, install the following components _(kubelet, kubeadm & kubectl)_ on all the nodes in the cluster
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
exit
exit
```

4. Relogin & join the cluster using the following command as a root user on Master/Control Plane
```bash
sudo -i
kubeadm join 172.31.25.16:6443 --token sz14lp.jwkx2vy49w54fk79 \
--discovery-token-ca-cert-hash sha256:25fe0576979b9306d911139f22c47f02240ab63731619a745b0e396ddf9fbe46 \
--cri-socket "unix:///var/run/cri-dockerd.sock"
```

5. The above command received on Master/Control plane after initializing the cluster

OUTPUT:
```php
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

6. Copy the node joining command mentioned in the output and also add ```--cri-socket "unix:///var/run/cri-dockerd.sock"``` at the end. And execute this command as a root user in all the nodes except Master
```
kubeadm join 172.31.25.16:6443 --token sz14lp.jwkx2vy49w54fk79 \
        --discovery-token-ca-cert-hash sha256:25fe0576979b9306d911139f22c47f02240ab63731619a745b0e396ddf9fbe46 --cri-socket "unix:///var/run/cri-dockerd.sock"
```