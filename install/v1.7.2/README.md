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


调整`cgroup-driver`，保持一致：
```
$ vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
... ...
Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=cgroupfs"
... ...

$ docker info |grep -i cgroup
WARNING: overlay: the backing xfs filesystem is formatted without d_type support, which leads to incorrect behavior.
         Reformat the filesystem with ftype=1 to enable d_type support.
         Running without d_type support will not be supported in future releases.
Cgroup Driver: cgroupfs

$ systemctl daemon-reload
$ systemctl restart docker && systemctl restart kubelet
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
[root@k8s01 ~]# docker images
REPOSITORY                                               TAG                 IMAGE ID            CREATED             SIZE
gcr.io/google_containers/kube-proxy-amd64                v1.7.2              e7ba9fbdf364        7 days ago          115MB
gcr.io/google_containers/kube-scheduler-amd64            v1.7.2              0d33fd775e00        7 days ago          77.2MB
gcr.io/google_containers/kube-apiserver-amd64            v1.7.2              d6fdafdff0af        7 days ago          186MB
gcr.io/google_containers/k8s-dns-kube-dns-amd64          1.14.4              a458d31f8b2c        7 days ago          49.4MB
gcr.io/google_containers/kube-controller-manager-amd64   v1.7.2              75bc21986d92        7 days ago          138MB
gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64     1.14.4              ef75f942b3c8        8 days ago          41.4MB
gcr.io/google_containers/k8s-dns-sidecar-amd64           1.14.4              c2e31fcbeef6        8 days ago          41.8MB
weaveworks/weave-npc                                     2.0.1               4f71bca714a3        5 weeks ago         54.7MB
weaveworks/weave-kube                                    2.0.1               d2099d50a03b        5 weeks ago         101MB
gcr.io/google_containers/pause-amd64                     3.0                 b42f5a67adc0        8 weeks ago         747kB
gcr.io/google_containers/etcd-amd64                      3.0.17              92f272c47f5a        2 months ago        169MB
```

### Node Images

```
[root@k8s02 ~]# docker images
REPOSITORY                                             TAG                 IMAGE ID            CREATED             SIZE
gcr.io/google_containers/kube-proxy-amd64              v1.7.2              e7ba9fbdf364        7 days ago          115MB
gcr.io/google_containers/k8s-dns-kube-dns-amd64        1.14.4              a458d31f8b2c        7 days ago          49.4MB
gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64   1.14.4              ef75f942b3c8        8 days ago          41.4MB
gcr.io/google_containers/k8s-dns-sidecar-amd64         1.14.4              c2e31fcbeef6        8 days ago          41.8MB
weaveworks/weave-npc                                   2.0.1               4f71bca714a3        5 weeks ago         54.7MB
weaveworks/weave-kube                                  2.0.1               d2099d50a03b        5 weeks ago         101MB
gcr.io/google_containers/pause-amd64                   3.0                 b42f5a67adc0        8 weeks ago         747kB
```


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

### 安装CNI

未安装时，会抛异常`cni config uninitialized`，解决方案 [issue/cni config uninitialized.md](../issue/cni%20config%20uninitialized.md)

```
$ kubectl apply -f https://git.io/weave-kube-1.6
```

无法自动下载时，可使用提供的yaml文件：

```
$ kubectl apply -f ./yaml/weave-daemonset-k8s-1.6.yaml
```

### 安装`kubernetes-dashboard`

[install kubernetes-dashboard](../../gcr.io/kubernetes-dashboard-amd64)

---

###### QA

- [The connection to the server localhost:8080 was refused - did you specify the right host or port?](../issue/did%20you%20specify%20the%20right%20host%20or%20port.md)
- [issue/cni config uninitialized.md](../issue/cni%20config%20uninitialized.md)

--- 

###### reference

- [使用kubeadm安装Kubernetes 1.7](http://blog.frognew.com/2017/07/kubeadm-install-kubernetes-1.7.html)
- [在 CentOS 7 下安装配置 shadowsocks](http://morning.work/page/2015-12/install-shadowsocks-on-centos-7.html)
- [1.6.0 kubelet fails with error "misconfiguration: kubelet cgroup driver: "cgroupfs" is different from docker cgroup driver: "systemd" #43805](https://github.com/kubernetes/kubernetes/issues/43805)

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