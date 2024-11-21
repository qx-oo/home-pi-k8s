# nfs

```
# master 挂载
sudo mount /dev/sda1 /mnt/k8s/
```

```bash

sudo apt install nfs-kernel-server

sudo mkdir /mnt/k8s/

sudo chown -R nobody:nogroup /mnt/k8s/
sudo chmod -R 2775 /mnt/k8s/

vi /etc/exports

    /mnt/k8s 10.64.1.2(rw,sync,no_subtree_check,no_root_squash)
    /mnt/k8s 10.64.1.3(rw,sync,no_subtree_check,no_root_squash)
    /mnt/k8s 10.64.1.4(rw,sync,no_subtree_check,no_root_squash)

sudo systemctl restart nfs-kernel-server

sudo exportfs -a

sudo exportfs -s

# node install
sudo apt install nfs-common

# nfs git: https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner.git

kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 200Gi
  accessModes:
    - ReadWriteMany
  nfs:
    path: /mnt/k8s/nfspv
    server: 10.64.1.2
EOF

kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
    accessModes:
        - ReadWriteMany
    resources:
        requests:
        storage: 200Gi
EOF
```