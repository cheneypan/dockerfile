# image mirror

## 


## 镜像

### v1.6.4

```sh
#!/bin/sh

echo -----------------------------------------------------------------------------
echo ">>> images in path(/etc/kubernetes/manifests)"
echo gcr.io/google_containers/etcd-amd64:3.0.17
echo gcr.io/google_containers/kube-apiserver-amd64:v1.6.4
echo gcr.io/google_containers/kube-controller-manager-amd64:v1.6.4
echo gcr.io/google_containers/kube-scheduler-amd64:v1.6.4
echo ' '
echo ">>> other images"
echo gcr.io/google_containers/kube-proxy-amd64:v1.6.4
echo -----------------------------------------------------------------------------

images=(
etcd-amd64:3.0.17
kube-apiserver-amd64:v1.6.4
kube-controller-manager-amd64:v1.6.4
kube-scheduler-amd64:v1.6.4
kube-proxy-amd64:v1.6.4
pause-amd64:3.0
k8s-dns-sidecar-amd64:1.14.1
k8s-dns-sidecar-amd64:1.14.2
k8s-dns-kube-dns-amd64:1.14.1
k8s-dns-kube-dns-amd64:1.14.2
k8s-dns-dnsmasq-nanny-amd64:1.14.1
k8s-dns-dnsmasq-nanny-amd64:1.14.2
)

for image in ${images[*]}
do
  echo start to pull image [$image]
  docker pull drolly/$image
  docker tag docker.io/drolly/$image gcr.io/google_containers/$image
  docker rmi drolly/$image
done
```
