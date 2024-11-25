# Dashborad

```
kubectl apply -f dashboard.yaml

kubectl apply -f dashboard-user.yaml

kubectl apply -f svc.yaml

kubectl create token admin-user -n kube-system


# 创建qxoo用户
kubectl create serviceaccount dashboard-qxoo -n kubernetes-dashboard

kubectl create clusterrolebinding dashboard-qxoo --clusterrole=cluster-admin --serviceaccount=kubernetes-dashboard:dashboard-qxoo

kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: dashboard-qxoo-token
  namespace: kubernetes-dashboard
  annotations:
    kubernetes.io/service-account.name: dashboard-qxoo
type: kubernetes.io/service-account-token
EOF


```


kubeconfig
```yaml
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority-data: <替换ca证书>
    server: https://<替换host>
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: dashboard-admin
  name: dashboard-context
current-context: dashboard-context
users:
- name: dashboard-admin
  user:
    token: <替换token>
```