# nextcloud

```
create database nextcloud;

grant all privileges on database nextcloud to nextcloud;

GRANT CREATE ON SCHEMA public TO nextcloud;

kubectl apply -f base.yaml

# disable Probe, after enable

helm upgrade --install -n nextcloud -f values.yaml nextcloud nextcloud/nextcloud

kubectl apply -f svc.yaml
```
