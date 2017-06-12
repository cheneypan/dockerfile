# 利用kubeadm安装k8s

## 环境初始化

```
# vi /etc/selinux/config 
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
```

```
# tee /etc/yum.repos.d/mritd.repo << EOF
[mritd]
name=Mritd Repository
baseurl=https://yum.mritd.me/centos/7/x86_64
enabled=1
gpgcheck=1
gpgkey=https://mritd.b0.upaiyun.com/keys/rpm.public.key
EOF
```

```
setenforce 0  && \
yum makecache && \
yum update    && \
yum install -y docker kubelet kubeadm kubectl kubernetes-cni  && \
systemctl disable firewalld.service && systemctl stop firewalld.service  && \
systemctl enable docker && systemctl start docker             && \
systemctl enable kubelet && systemctl start kubelet
```

Docker加速器

```
# mkdir -p /etc/docker

# tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://****.mirror.aliyuncs.com"]
}
EOF

# systemctl daemon-reload
# systemctl restart docker
```


## 安装

各版本依赖镜像：

- [v1.6.4](./v1.6.4/README.md)

初始化集群：

```
kubeadm init --pod-network-cidr 10.244.0.0/16 --kubernetes-version=v1.6.4
```
