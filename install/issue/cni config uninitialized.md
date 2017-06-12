
## network plugin is not ready: cni config uninitialized

```
[root@k8s01 ~]# kubectl get no
NAME      STATUS     AGE       VERSION
k8s01     NotReady   1m        v1.6.4
```

```
[root@k8s01 ~]# kubectl describe node k8s01
Conditions:
  Type			Status	LastHeartbeatTime			LastTransitionTime			Reason				Message
  ----			------	-----------------			------------------			------				-------
  OutOfDisk 		False 	Mon, 12 Jun 2017 03:37:01 -0400 	Sun, 11 Jun 2017 20:45:57 -0400 	KubeletHasSufficientDisk 	kubelet has sufficient disk space available
  MemoryPressure 	False 	Mon, 12 Jun 2017 03:37:01 -0400 	Sun, 11 Jun 2017 20:45:57 -0400 	KubeletHasSufficientMemory 	kubelet has sufficient memory available
  DiskPressure 		False 	Mon, 12 Jun 2017 03:37:01 -0400 	Sun, 11 Jun 2017 20:45:57 -0400 	KubeletHasNoDiskPressure 	kubelet has no disk pressure
  Ready 		False 	Mon, 12 Jun 2017 03:37:01 -0400 	Sun, 11 Jun 2017 20:45:57 -0400 	KubeletNotReady 		runtime network not ready: NetworkReady=false r
eason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitializedAddresses:		10.10.12.31,10.10.12.31,k8s01
```

未安装CNI。

## 解决方案

```
$ kubectl apply -f https://git.io/weave-kube-1.6
```

涉及镜像：

1. weaveworks/weave-kube:1.9.7
2. weaveworks/weave-npc:1.9.7


```
[root@k8s01 ~]# kubectl get no
NAME      STATUS    AGE       VERSION
k8s01     Ready     26m       v1.6.4

[root@k8s01 ~]# kubectl get pods --all-namespaces
NAMESPACE     NAME                            READY     STATUS    RESTARTS   AGE
kube-system   etcd-k8s01                      1/1       Running   0          28m
kube-system   kube-apiserver-k8s01            1/1       Running   0          30m
kube-system   kube-controller-manager-k8s01   1/1       Running   0          30m
kube-system   kube-dns-3913472980-2lrmp       3/3       Running   0          29m
kube-system   kube-proxy-0pr2g                1/1       Running   0          4m
kube-system   kube-proxy-8f1hq                1/1       Running   0          29m
kube-system   kube-scheduler-k8s01            1/1       Running   0          30m
kube-system   weave-net-kqsrs                 2/2       Running   0          5m
kube-system   weave-net-rtqf4                 2/2       Running   0          4m
```

---

##### reference

1. [kubeadm 1.6 is broken due to unconfigured CNI making kubelet NotReady #43815](https://github.com/kubernetes/kubernetes/issues/43815)
2. [Using kubeadm to Create a Cluster](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)
3. [Installing Addons](https://kubernetes.io/docs/concepts/cluster-administration/addons/)
4. [Integrating Kubernetes via the Addon](https://www.weave.works/docs/net/latest/kube-addon/)

