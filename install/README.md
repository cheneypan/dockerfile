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

#### Docker加速器

使用阿里云提供的[Docker Hub 镜像站点](https://cr.console.aliyun.com/?spm=5176.1971733.0.2.hX1LU8#/accelerator)，注册后获取专属加速器地址，替换下面的`registry-mirrors`值。

```
$ mkdir -p /etc/docker

$ tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://i9up8qq1.mirror.aliyuncs.com", "https://registry.docker-cn.com"]
}
EOF

$ systemctl daemon-reload && systemctl restart docker
```

## 安装集群

### images

提前下载好各版本依赖镜像，否则会卡死：

- [v1.7.2](./v1.7.2/README.md)
- [v1.6.4](./v1.6.4/README.md)
