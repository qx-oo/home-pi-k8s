# Home Raspberry pi k8s deploy

### TODO

- [ ] transmission
- [ ] plex

### Reference

[docker images](https://docs.linuxserver.io/)

### crictl

```bash

vi ~/.crictl.yaml

    runtime-endpoint: unix:///run/containerd/containerd.sock

sudo crictl -c ~/.crictl.yaml images

```

### containerd config proxy
    
```bash
sudo vi /usr/lib/systemd/system/containerd.service

[Service]
Environment="HTTP_PROXY=socks5://10.64.0.4:7890" 
Environment="HTTPS_PROXY=socks5://10.64.0.4:7890"
Environment="NO_PROXY=localhost,127.0.0.0/8,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16,registry.aliyuncs.com"

sudo systemctl daemon-reload
sudo systemctl restart containerd.service
```

### pi5开启pcie接电启动

```
sudo vi /boot/firmware/config.txt
    
        dtparam=pciex1

sudo rpi-eeprom-config --edit

        BOOT_ORDER=0xf416
        PCIE_PROBE=1

sudo reboot
```