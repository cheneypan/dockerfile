# k8s v1.7.2

## 安装docker

```
$ yum install -y yum-utils device-mapper-persistent-data lvm2
$ yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

$ yum list docker-ce.x86_64  --showduplicates |sort -r

[root@k8s01 ~]# yum list docker-ce --showduplicates | sort -r
 * updates: repo1.dal.innoscale.net
Loading mirror speeds from cached hostfile
Loaded plugins: fastestmirror
 * extras: mirror.pac-12.org
docker-ce.x86_64            17.06.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.2.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.0.ce-1.el7.centos             docker-ce-stable

$ yum makecache fast
$ yum install docker-ce-17.06.0.ce-1.el7.centos

$ systemctl start docker && systemctl enable docker
```

修改各节点docker的cgroup driver使其和kubelet一致，即修改或创建/etc/docker/daemon.json，加入下面的内容：

```
$ [root@k8s01 ~]# vim /etc/docker/daemon.json

{
  "registry-mirrors": ["https://i9up8qq1.mirror.aliyuncs.com", "https://registry.docker-cn.com"],
  "exec-opts": ["native.cgroupdriver=systemd"]
}

$ systemctl daemon-reload && systemctl restart docker
```

Docker从1.13版本开始调整了默认的防火墙规则，禁用了iptables filter表中FOWARD链，这样会引起Kubernetes集群中跨Node的Pod无法通信，在各个Docker节点执行下面的命令：

```
$ iptables -P FORWARD ACCEPT
```

## 安装kubeadm和kubelet

yum源

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```

```
$ yum makecache && yum update
```

查看kubeadm, kubelet, kubectl, kubernets-cni的最新版本（已安装Shadowsocks客户端 + proxychains4）：

```
[root@k8s01 ~]# proxychains4 yum list kubeadm  --showduplicates |sort -r
[proxychains] config file found: /etc/proxychains.conf
[proxychains] preloading /usr/local/lib/libproxychains4.so
... ...
kubeadm.x86_64                        1.7.2-0                        kubernetes 
kubeadm.x86_64                        1.7.2-0                        @kubernetes
kubeadm.x86_64                        1.7.1-0                        kubernetes 
kubeadm.x86_64                        1.7.0-0                        kubernetes 
kubeadm.x86_64                        1.6.7-0                        kubernetes 
kubeadm.x86_64                        1.6.6-0                        kubernetes 
kubeadm.x86_64                        1.6.5-0                        kubernetes 
kubeadm.x86_64                        1.6.4-0                        kubernetes 
kubeadm.x86_64                        1.6.3-0                        kubernetes 
kubeadm.x86_64                        1.6.2-0                        kubernetes 
kubeadm.x86_64                        1.6.1-0                        kubernetes 
kubeadm.x86_64                        1.6.0-0                        kubernetes 
Installed Packages
 * extras: repo.us.bigstepcloud.com
 * base: mirrors.sonic.net
Available Packages
```

```
$ yum install -y kubelet kubeadm kubectl kubernetes-cni

$ systemctl disable firewalld.service && systemctl stop firewalld.service

$ systemctl enable kubelet && systemctl start kubelet
```

### master

```
kubeadm init \
   --pod-network-cidr 10.244.0.0/16 \
   --kubernetes-version=v1.7.2 \
   --apiserver-advertise-address=10.10.12.31 \
   --skip-preflight-checks
```

## 镜像

### Master Images

```
REPOSITORY                                               TAG                 IMAGE ID            CREATED             SIZE
gcr.io/google_containers/kube-proxy-amd64                v1.7.2              e7ba9fbdf364        40 hours ago        115MB
gcr.io/google_containers/kube-scheduler-amd64            v1.7.2              0d33fd775e00        41 hours ago        77.2MB
gcr.io/google_containers/kube-apiserver-amd64            v1.7.2              d6fdafdff0af        41 hours ago        186MB
gcr.io/google_containers/k8s-dns-kube-dns-amd64          1.14.4              a458d31f8b2c        41 hours ago        49.4MB
gcr.io/google_containers/kube-controller-manager-amd64   v1.7.2              75bc21986d92        41 hours ago        138MB
gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64     1.14.4              ef75f942b3c8        42 hours ago        41.4MB
gcr.io/google_containers/k8s-dns-sidecar-amd64           1.14.4              c2e31fcbeef6        42 hours ago        41.8MB
gcr.io/google_containers/pause-amd64                     3.0                 b42f5a67adc0        7 weeks ago         747kB
gcr.io/google_containers/etcd-amd64                      3.0.17              92f272c47f5a        7 weeks ago         169MB
weaveworks/weave-npc                                     2.0.1               4f71bca714a3        4 weeks ago         54.7MB
weaveworks/weave-kube                                    2.0.1               d2099d50a03b        4 weeks ago         101MB
```

### Node Images

Image Name                                               | Version
---                                                      |---
gcr.io/google_containers/kube-proxy-amd64                | v1.7.2
gcr.io/google_containers/pause-amd64                     | 3.0
docker.io/weaveworks/weave-npc                           | 1.9.7
docker.io/weaveworks/weave-kube                          | 1.9.7


### 下载镜像

通过`docker hub`中转。

pull.images-v1.7.2.sh

```
#!/bin/sh
echo -----------------------------------------------------------------------------
echo pull all images needed of "gcr.io/google_containers"
echo -----------------------------------------------------------------------------

images=(
k8s-dns-sidecar-amd64:1.14.4
k8s-dns-dnsmasq-nanny-amd64:1.14.4
k8s-dns-kube-dns-amd64:1.14.4
kube-controller-manager-amd64:v1.7.2
kube-apiserver-amd64:v1.7.2
kube-scheduler-amd64:v1.7.2
etcd-amd64:3.0.17
kube-proxy-amd64:v1.7.2
pause-amd64:3.0
)

for image in ${images[*]}
do
  echo start to pull image [$image]
  docker pull drolly/$image
  docker tag docker.io/drolly/$image gcr.io/google_containers/$image
  docker rmi drolly/$image
  echo --------------------------------------------------------------------
done
```

---

###### QA

- [The connection to the server localhost:8080 was refused - did you specify the right host or port?](../issue/did%20you%20specify%20the%20right%20host%20or%20port.md)
- [issue/cni config uninitialized.md](../issue/cni%20config%20uninitialized.md)

--- 

###### reference

- [使用kubeadm安装Kubernetes 1.7](http://blog.frognew.com/2017/07/kubeadm-install-kubernetes-1.7.html)
- [在 CentOS 7 下安装配置 shadowsocks](http://morning.work/page/2015-12/install-shadowsocks-on-centos-7.html)

```
[root@k8s01 ~]# bash <(curl -s http://morning.work/examples/2015-12/install-shadowsocks.sh)
Installing Shadowsocks...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 1558k  100 1558k    0     0  20699      0  0:01:17  0:01:17 --:--:-- 19254
Requirement already up-to-date: pip in /usr/lib/python2.7/site-packages
Requirement already up-to-date: pip in /usr/lib/python2.7/site-packages
Requirement already satisfied: shadowsocks in /usr/lib/python2.7/site-packages
{
  "server": "0.0.0.0",
  "server_port": 8388,
  "password": "QKSGhac70HGBNrmhZq1D09oPd0LACudS",
  "method": "aes-256-cfb"
}
[Unit]
Description=Shadowsocks

[Service]
TimeoutStartSec=0
ExecStart=/usr/bin/ssserver -c /etc/shadowsocks.json

[Install]
WantedBy=multi-user.target
● shadowsocks.service - Shadowsocks
   Loaded: loaded (/etc/systemd/system/shadowsocks.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2017-08-01 00:48:21 EDT; 5s ago
 Main PID: 27689 (ssserver)
   CGroup: /system.slice/shadowsocks.service
           └─27689 /usr/bin/python /usr/bin/ssserver -c /etc/shadowsocks.json

Aug 01 00:48:21 k8s01 systemd[1]: Started Shadowsocks.
Aug 01 00:48:21 k8s01 systemd[1]: Starting Shadowsocks...
Aug 01 00:48:21 k8s01 ssserver[27689]: INFO: loading config from /etc/shadowsocks.json
Aug 01 00:48:21 k8s01 ssserver[27689]: 2017-08-01 00:48:21 INFO     loading libcrypto from libcrypto.so.10
Aug 01 00:48:21 k8s01 ssserver[27689]: 2017-08-01 00:48:21 INFO     starting server at 0.0.0.0:8388
================================

Congratulations! Shadowsocks has been installed on your system.
You shadowsocks connection info:
--------------------------------
server:      10.10.12.31
server_port: 8388
password:    QKSGhac70HGBNrmhZq1D09oPd0LACudS
method:      aes-256-cfb
--------------------------------
```