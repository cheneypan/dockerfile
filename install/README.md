# 利用kubeadm安装k8s

## 环境初始化

#### 关闭SELinux

```
$ vi /etc/selinux/config 
... ...
SELINUX=disabled
... ...
```

正常需重启才能生效，但`setenforce 0`可临时关闭selinux。

```
$ setenforce 0
```

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

#### Docker加速器

使用阿里云提供的[Docker Hub 镜像站点](https://cr.console.aliyun.com/?spm=5176.1971733.0.2.hX1LU8#/accelerator)，注册后获取专属加速器地址，替换下面的`registry-mirrors`值。

```
$ mkdir -p /etc/docker

$ tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://****.mirror.aliyuncs.com"]
}
EOF

$ systemctl daemon-reload && systemctl restart docker
```

## 安装集群

### images

提前下载好各版本依赖镜像，否则会卡死：

- [v1.6.4](./v1.6.4/README.md)

### 初始化

初始化集群：

```
kubeadm init --pod-network-cidr 10.244.0.0/16 --kubernetes-version=v1.6.4
```

### 安装CNI

未安装时，会抛异常`cni config uninitialized`，解决方案 [issue/cni config uninitialized.md](./issue/cni%config%uninitialized.md)

```
$ kubectl apply -f https://git.io/weave-kube-1.6
```

无法自动下载时，可使用提供的yaml文件：

```
$ kubectl apply -f ./yaml/weave-daemonset-k8s-1.6.yaml
```

