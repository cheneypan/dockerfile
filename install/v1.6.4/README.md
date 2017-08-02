# k8s v1.6.4

#### 更新yum

添加 yum 源
```
$ tee /etc/yum.repos.d/mritd.repo << EOF
[mritd]
name=Mritd Repository
baseurl=https://yum.mritd.me/centos/7/x86_64
enabled=1
gpgcheck=1
gpgkey=https://mritd.b0.upaiyun.com/keys/rpm.public.key
EOF
```

所有关于 yum 源地址变更等都将在 https://yum.mritd.me 页面公告，如出现不能使用请访问此页面查看相关原因；如果实在下载过慢可以将 yum.mritd.me 替换成 yumrepo.b0.upaiyun.com，此域名 yum 源在 CDN 上，由于流量有限，请使用 yumdownloader 工具下载到本地分发安装。

刷新cache 及 更新

```
$ yum makecache && yum update
```

#### 安装 docker kubelet kubeadm kubectl kubernetes-cni

```
$ yum install -y docker kubelet kubeadm kubectl kubernetes-cni

$ systemctl disable firewalld.service && systemctl stop firewalld.service

$ systemctl enable docker && systemctl start docker && systemctl enable kubelet && systemctl start kubelet
```

## 镜像

### Master Images

Image Name                                               | Version
---                                                      |---
gcr.io/google_containers/k8s-dns-sidecar-amd64           | 1.14.1
gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64     | 1.14.1
gcr.io/google_containers/k8s-dns-kube-dns-amd64          | 1.14.1
gcr.io/google_containers/kube-controller-manager-amd64   | v1.6.4
gcr.io/google_containers/kube-apiserver-amd64            | v1.6.4
gcr.io/google_containers/kube-scheduler-amd64            | v1.6.4
gcr.io/google_containers/etcd-amd64                      | 3.0.17
gcr.io/google_containers/kube-proxy-amd64                | v1.6.4
gcr.io/google_containers/pause-amd64                     | 3.0
docker.io/weaveworks/weave-npc                           | 1.9.7
docker.io/weaveworks/weave-kube                          | 1.9.7

### Node Images

Image Name                                               | Version
---                                                      |---
gcr.io/google_containers/kube-proxy-amd64                | v1.6.4
gcr.io/google_containers/pause-amd64                     | 3.0
docker.io/weaveworks/weave-npc                           | 1.9.7
docker.io/weaveworks/weave-kube                          | 1.9.7


### 下载镜像

通过`docker hub`中转。

pull.images-v1.6.4.sh

```
#!/bin/sh
echo -----------------------------------------------------------------------------
echo pull all images needed of "gcr.io/google_containers"
echo -----------------------------------------------------------------------------

images=(
k8s-dns-sidecar-amd64:1.14.1
k8s-dns-dnsmasq-nanny-amd64:1.14.1
k8s-dns-kube-dns-amd64:1.14.1
kube-controller-manager-amd64:v1.6.4
kube-apiserver-amd64:v1.6.4
kube-scheduler-amd64:v1.6.4
etcd-amd64:3.0.17
kube-proxy-amd64:v1.6.4
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


### 初始化

初始化集群：

```
kubeadm init --pod-network-cidr 10.244.0.0/16 --kubernetes-version=v1.6.4
```

### 安装CNI

未安装时，会抛异常`cni config uninitialized`，解决方案 [issue/cni config uninitialized.md](./issue/cni%20config%20uninitialized.md)

```
$ kubectl apply -f https://git.io/weave-kube-1.6
```

无法自动下载时，可使用提供的yaml文件：

```
$ kubectl apply -f ./yaml/weave-daemonset-k8s-1.6.yaml
```


