# postgres

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

kubectl create ns db

kubectl apply -f pvc.yaml

helm install -n db postgresql -f values.yaml bitnami/postgresql

```
