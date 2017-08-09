# kubernetes-dashboard

## yaml

地址: https://git.io/kube-dashboard

修改：

1. 镜像地址为`drolly/kubernetes-dashboard-amd64:v1.6.3`
2. 配置`apiserver-host`
3. 增加`nodeport`

```
$ vim kubernetes-dashboard.yaml
... ...
      containers:
      - name: kubernetes-dashboard
        image: drolly/kubernetes-dashboard-amd64:v1.6.3
        ports:
        - containerPort: 9090
          protocol: TCP
        args:
        - --apiserver-host=http://10.10.12.31:30099
... ...
---
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 9090
    nodePort: 32220
  selector:
    k8s-app: kubernetes-dashboard
```

完整文件：[v1.6.3](v1.6.3/kubernetes-dashboard.yaml)

## kubectl proxy

启动`proxy`：

```
$ kubectl proxy --address='0.0.0.0' --port=30099 --accept-hosts='^*$' &
```

## 启动`kubernetes-dashboard`

```
$ kubectl apply -f kubernetes-dashboard.yaml 
serviceaccount "kubernetes-dashboard" configured
clusterrolebinding "kubernetes-dashboard" configured
deployment "kubernetes-dashboard" configured
service "kubernetes-dashboard" configured
```

```
$ kubectl describe svc kubernetes-dashboard -n kube-system
Name:			kubernetes-dashboard
Namespace:		kube-system
Labels:			k8s-app=kubernetes-dashboard
Annotations:		kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"k8s-app":"kubernetes-dashboard"},"name":"kubernetes-dashboard","namespace":...
Selector:		k8s-app=kubernetes-dashboard
Type:			NodePort
IP:			10.100.90.240
Port:			<unset>	80/TCP
NodePort:		<unset>	32220/TCP
Endpoints:		10.36.0.1:9090
Session Affinity:	None
Events:			<none>
```

---

#### reference

- [Kubernetes集群Dashboard插件安装](http://tonybai.com/2017/01/19/install-dashboard-addon-for-k8s/)