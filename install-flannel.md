### reference

https://medium.com/karlmax-berlin/how-to-install-kubernetes-on-raspberry-pi-53b4ce300b58

---

```
sudo timedatectl set-timezone Asia/Shanghai
sudo locale-gen "en_US.UTF-8"
sudo dpkg-reconfigure locales
```

```
sudo apt update -y && sudo apt dist-upgrade -y

add cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1 to /boot/cmdline.txt

sudo apt install -y containerd containernetworking-plugins

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

systemctl stop apparmor
systemctl disable apparmor
systemctl stop ufw
systemctl disable ufw

systemctl restart containerd.service

echo 1 > /proc/sys/net/ipv4/ip_forward
sudo sysctl -w net.ipv4.ip_forward=1
# 永久生效
sudo vi /etc/sysctl.conf
modprobe br_netfilter
# master
sudo kubeadm init --v=5 \
--image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers \
--service-cidr=10.96.0.0/16 \
--pod-network-cidr=10.244.0.0/16 \
--kubernetes-version v1.28.3 \
--apiserver-advertise-address=192.168.31.101 \
--cri-socket unix:///run/containerd/containerd.sock

kubeadm token create --print-join-command

# node
kubeadm join 192.168.31.102:6443 --v=5 --token xxx --discovery-token-ca-cert-hash sha256:xxx \
--cri-socket unix:///run/containerd/containerd.sock
```

```
# install flannel
wget https://github.com/flannel-io/flannel/releases/download/v0.23.0/flanneld-arm64
sudo chmod +x flanneld-arm64
sudo cp flanneld-arm64 /usr/local/bin/flanneld
sudo mkdir -p /var/lib/k8s/flannel/networks
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

```
# dashboard deploy
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

# dashboard nodeport
kubectl --namespace kubernetes-dashboard patch svc kubernetes-dashboard -p '{"spec": {"type": "NodePort"}}'

# admin

apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
---
apiVersion: v1
kind: Secret
metadata:
  name: admin-user-token
  annotations:
    kubernetes.io/service-account.name: admin-user
type: kubernetes.io/service-account-token

# add token ttl to dashboard yml

args:
    - --token-ttl=86400

# create admin token

kubectl create token admin-user -n kube-system
```

# metllb

```
wget https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml

# 添加地址池
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: ip-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.31.201-192.168.31.220

# 添加广播
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: l2advertisement
  namespace: metallb-system
spec:
  ipAddressPools: 
    - ip-pool

# config arp
#kubectl get configmap kube-proxy -n kube-system -o yaml | \
#sed -e "s/strictARP: false/strictARP: true/" | \
#kubectl apply -f - -n kube-system

#kubectl get configmap -n kube-system kube-proxy -o yaml | sed -e "s/mode: \"\"/mode: ipvs/g" | kubectl apply -f - -n kube-system
```

# helm

```
wget https://get.helm.sh/helm-v3.13.2-linux-arm64.tar.gz
tar zxvf helm-v3.13.2-linux-arm64.tar.gz
sudo cp helm /usr/local/bin/helm
``

# zsh

```
PS1='%F{green}%n@%m %F{blue}%~%f$ '
```

