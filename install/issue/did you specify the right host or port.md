## The connection to the server localhost:8080 was refused - did you specify the right host or port?

```
[root@k8s-master ~]# kubectl get nodes
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

在`/etc/kubernetes/manifests/kube-apiserver.yaml`中：
`--insecure-port=0`

kube-apiserver的选项`--insecure-port=0`，也就是说kubeadm 1.6.0初始化的集群，kube-apiserver没有监听默认的http 8080端口。kube-apiserver只监听了https的6443端口。

为了使用kubectl访问apiserver，在~/.bash_profile中追加下面的环境变量：
```
export KUBECONFIG=/etc/kubernetes/admin.conf

source ~/.bash_profile

kubectl get nodes
```
