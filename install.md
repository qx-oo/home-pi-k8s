### reference

https://medium.com/karlmax-berlin/how-to-install-kubernetes-on-raspberry-pi-53b4ce300b58

---

# helm

```
wget https://get.helm.sh/helm-v3.13.2-linux-arm64.tar.gz
tar zxvf helm-v3.13.2-linux-arm64.tar.gz
sudo cp linux-arm64/helm /usr/local/bin/helm
```

# kubernetes

```
sudo timedatectl set-timezone Asia/Shanghai
sudo locale-gen "en_US.UTF-8"
sudo dpkg-reconfigure locales
```

```
sudo apt update -y && sudo apt dist-upgrade -y

添加 `usb_max_current_enable=1` /boot/firmware/config.txt 为新行

// 查看最大电流
sudo lsusb -v | grep MaxPower

追加 ` cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1` to /boot/firmware/cmdline.txt 末尾

sudo apt install -y containerd containernetworking-plugins

sudo mkdir /etc/containerd/

cat <<EOF | sudo tee /etc/containerd/config.toml
version = 2
[plugins]
  [plugins."io.containerd.grpc.v1.cri"]
    sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.8"
    [plugins."io.containerd.grpc.v1.cri".containerd]
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
          runtime_type = "io.containerd.runc.v2"
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            SystemdCgroup = true
EOF

sudo apt install -y apt-transport-https ca-certificates curl
sudo apt install -y linux-modules-extra-raspi

curl -fsSL http://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -
sudo add-apt-repository "deb http://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial main"

sudo apt-get install kubelet kubeadm kubectl

sudo apt-mark hold kubelet kubeadm kubectl

# echo 1 > /proc/sys/net/ipv4/ip_forward
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv6.conf.all.disable_ipv6=1
net.ipv6.conf.default.disable_ipv6=1
net.ipv6.conf.lo.disable_ipv6=1
net.ipv6.conf.eth0.disable_ipv6=1
net.ipv4.ip_forward=1
EOF

sudo sysctl --system

sudo systemctl stop apparmor
sudo systemctl disable apparmor
sudo systemctl stop ufw
sudo systemctl disable ufw

sudo systemctl restart containerd.service

sudo su -c "echo 1 > /proc/sys/net/ipv4/ip_forward"
sudo sysctl -w net.ipv4.ip_forward=1
# 永久生效
sudo vi /etc/sysctl.conf
'''
net.ipv4.ip_forward=1
'''
modprobe br_netfilter

# master
sudo kubeadm init --v=5 \
--image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers \
--skip-phases=addon/kube-proxy \
--service-cidr=10.96.0.0/16 \
--pod-network-cidr=10.244.0.0/16 \
--kubernetes-version v1.28.3 \
--apiserver-advertise-address=10.64.1.2 \
--cri-socket unix:///run/containerd/containerd.sock

rm -rf $HOME/.kube
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubeadm token create --print-join-command

# node
sudo kubeadm join 10.64.1.2:6443 --v=5 --token xxx --discovery-token-ca-cert-hash sha256:xxx \
--cri-socket unix:///run/containerd/containerd.sock
```

```
# cert manager (不进行安装)
helm repo add jetstack https://charts.jetstack.io
helm install  cert-manager jetstack/cert-manager --namespace cert-manager   --create-namespace  --version v1.14.0
# disable cert-manager validation
kubectl label namespace kube-system cert-manager.io/disable-validation=true
```

```
# install cilium
# install nfs
# install metallb
# install dashboard
```


