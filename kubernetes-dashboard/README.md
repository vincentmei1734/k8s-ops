#  kubernetes dashboard
该kubernetes dashboard是一个模版，可以根据生产情况做调整，以上版本在我的环境可以正常运行。我的版本给ServiceAccount:kubernetes-dashboard绑定了cluster-admin权限，如果正式环境使用请慎重考虑安全问题。

### checkout版本并安装

```
git clone https://github.com/vincentmei1734/k8s-ops.git
KUBECTL apply -f kubernetes-dashboard/
```

### 查看运行状态
```
[root@master kubernetes-dashboard]# kubectl get  all  -n kube-system | grep dashboard
pod/kubernetes-dashboard-765cc7b476-28mtl        1/1     Running   0          18m
service/kubernetes-dashboard                           ClusterIP   10.106.100.187   <none>        443/TCP         18m
deployment.apps/kubernetes-dashboard   1/1     1            1           18m
replicaset.apps/kubernetes-dashboard-765cc7b476   1         1         1       18m
```

### 暴露服务

```
kubectl proxy --address=0.0.0.0 --port=8080 --disable-filter=true
```

### 浏览器访问【选择跳过访问】
```
http://x.x.x.x:8080/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/      #替换自己的master ip
```


### 参考资料
https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/