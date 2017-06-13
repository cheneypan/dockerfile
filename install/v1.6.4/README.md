# k8s v1.6.4

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

通过`docker hub`跳转。

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


