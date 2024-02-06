# nfs

```
sudo apt install nfs-kernel-server

sudo mkdir /mnt/k8s/

sudo chown -R nobody:nogroup /mnt/k8s/
sudo chmod -R 2775 /mnt/k8s/

vi /etc/exports

    /mnt/k8s 192.168.31.0/24(rw,sync,no_subtree_check,no_root_squash)

sudo systemctl restart nfs-kernel-server

sudo exportfs -a

sudo exportfs -s

sudo apt install nfs-common

# nfs git: https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner.git

kubectl create ns nfs
kubectl apply -f nfs-rbac.yaml
kubectl apply -f nfs.yaml
kubectl apply -f class.yaml

sudo mount /dev/sda1 /mnt/k8s/
```
