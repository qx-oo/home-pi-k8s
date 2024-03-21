# Cilium

```
# install cilium
helm repo add cilium https://helm.cilium.io/
helm upgrade --install cilium cilium/cilium --version 1.15.0 \
--namespace kube-system \
-f values.yaml

# cli
wget https://github.com/cilium/cilium-cli/releases/download/v0.15.22/cilium-linux-arm64.tar.gz
tar zxvf cilium-linux-arm64.tar.gz
sudo cp cilium /usr/local/bin/cilium

# 使用metallb替代
# kubectl apply -f policy.yaml
# kubectl apply -f ip-pool.yaml
```

### 注意

因为集群没有使用kube-proxy, 需要配置k8sServiceHost, k8sServicePort
