curl-Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v0.30.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://pecxp37q.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

```sh
sudo -E minikube stop
sudo -E minikube delete

sudo rm -fr /var/lib/minikube/certs/
sudo rm -fr /etc/kubernetes/
sudo rm -fr /home/chenmx/.kube/

```

sudo -E minikube start  --vm-driver=none 


https://storage.googleapis.com/kubernetes-release/release/v1.15.0/bin/linux/amd64/kubelet

https://storage.googleapis.com/kubernetes-release/release/v1.15.0/bin/linux/amd64/kubeadm

docker pull mirrorgooglecontainers/kube-apiserver:v1.15.0
docker pull mirrorgooglecontainers/kube-controller-manager:v1.15.0
docker pull mirrorgooglecontainers/kube-scheduler:v1.15.0
docker pull mirrorgooglecontainers/kube-proxy:v1.15.0
docker pull mirrorgooglecontainers/pause:3.1
docker pull mirrorgooglecontainers/etcd:3.3.10
docker pull coredns/coredns:1.3.1


docker pull mirrorgooglecontainers/kube-apiserver:v1.15.0
docker pull mirrorgooglecontainers/kube-controller-manager:v1.15.0
docker pull mirrorgooglecontainers/kube-scheduler:v1.15.0
docker pull mirrorgooglecontainers/kube-proxy:v1.15.0
docker pull mirrorgooglecontainers/pause:3.1
docker pull mirrorgooglecontainers/etcd:3.3.10
docker pull coredns/coredns:1.3.1


```sh
    sudo mv /home/chenmx/.kube /home/chenmx/.minikube $HOME
    sudo chown -R $USER $HOME/.kube $HOME/.minikube
```