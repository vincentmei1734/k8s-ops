# Prometheus Operator
这篇文章的主要目的是简单了解prometheus operator并快速在自己环境部署一套prometheus operator。

### 简单介绍
[brancz](https://github.com/coreos/prometheus-operator)是prometheus operator项目的核心代码提交人。这个项目主要是解决prometheus在Kubernetes监控方案的落地。prometheus operator会自动创建、配置、管理[Prometheus](https://prometheus.io/)监控实例,另外会自动生成监控对象基于kubernetes的label匹配。总之，这个工具就是解决了kubernetes集群上node，pod，namespace,cluster整体的监控落地。
[camilb](https://github.com/camilb/prometheus-kubernetes)提供了prometheus operator一键部署，功能列举如下：
- 支持AWS,GCP，Azure自动化安装配置，支持自建kubernetes集群；
- 为prometheus以statefulsets方式部署
- 默认配置了servicemonitor
- 预置了告警
- 预置了Grafana dashboard和模版dashboard
- 部署不到一分钟，支持一键卸载
这篇文章主要是使用这个一键部署脚本实现Kubernets集群监控，下面我们开始操作了。

### 运行环境：
- kubernetes 1.13.0版本
- docker 18.09.2版本
- centos 7系统
- Prometheus Operator v0.23.1

### 快速部署
下面github是从camilb大神fork出来，并针对国内环境和部分bug做了修改。使用下面版本即可。在使用deploy脚本的时候注意下下面2点；
- kubectl默认是没有使用http的api，如果使用了https，请参照下面修改deploy脚本，最好放在39行；
```
kubelet(){
/usr/bin/kubectl --server=https://x.x.x.x:6443 --certificate-authority=/etc/kubernetes/pki/ca.crt --client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --client-key=/etc/ kubernetes/pki/apiserver-kubelet-client.key "$@"
}
```
- deploy和teardown脚本在执行完后会使用git checkout -- . 会还原仓库

```
git clone https://github.com/vincentmei1734/k8s-ops.git
cd prometheus-operator/
./deploy
```

### 检查部署
```
[root@master prometheus-kubernetes]# kubectl get all -n monitoring
NAME                                       READY   STATUS    RESTARTS   AGE
pod/alertmanager-main-0                    2/2     Running   0          3d20h
pod/alertmanager-main-1                    2/2     Running   0          3d20h
pod/alertmanager-main-2                    2/2     Running   0          3d20h
pod/grafana-85d9c9c4cb-4rckm               1/1     Running   0          3d20h
pod/kube-state-metrics-5899ccbdb7-svknk    2/2     Running   0          3d20h
pod/node-exporter-4zm2m                    1/1     Running   0          3d20h
pod/node-exporter-c5xvh                    1/1     Running   0          3d20h
pod/node-exporter-hd8wf                    1/1     Running   0          3d20h
pod/prometheus-k8s-0                       3/3     Running   1          3d20h
pod/prometheus-k8s-1                       3/3     Running   1          3d20h
pod/prometheus-operator-5b7f6c44f5-s9rwm   1/1     Running   0          3d20h

NAME                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
service/alertmanager-main       ClusterIP   10.103.146.106   <none>        9093/TCP            3d20h
service/alertmanager-operated   ClusterIP   None             <none>        9093/TCP,6783/TCP   3d20h
service/grafana                 ClusterIP   10.109.213.217   <none>        3000/TCP            3d20h
service/kube-state-metrics      ClusterIP   10.101.144.85    <none>        8080/TCP            3d20h
service/node-exporter           ClusterIP   None             <none>        9100/TCP            3d20h
service/prometheus-k8s          ClusterIP   10.107.169.7     <none>        9090/TCP            3d20h
service/prometheus-operated     ClusterIP   None             <none>        9090/TCP            3d20h
service/prometheus-operator     ClusterIP   10.111.245.9     <none>        8080/TCP            3d20h

NAME                           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/node-exporter   3         3         3       3            3           <none>          3d20h

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/grafana               1/1     1            1           3d20h
deployment.apps/kube-state-metrics    1/1     1            1           3d20h
deployment.apps/prometheus-operator   1/1     1            1           3d20h

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/grafana-85d9c9c4cb               1         1         1       3d20h
replicaset.apps/kube-state-metrics-5899ccbdb7    1         1         1       3d20h
replicaset.apps/kube-state-metrics-dfcbbb8f5     0         0         0       3d20h
replicaset.apps/prometheus-operator-5b7f6c44f5   1         1         1       3d20h

NAME                                 READY   AGE
statefulset.apps/alertmanager-main   3/3     3d20h
statefulset.apps/prometheus-k8s      2/2     3d20h


[root@master prometheus-kubernetes]# kubectl get servicemonitor -n monitoring
NAME                      AGE
alertmanager              3d
kube-apiserver            3d
kube-controller-manager   3d
kube-scheduler            3d
kube-state-metrics        3d
kubelet                   3d
node-exporter             3d
prometheus                3d
prometheus-operator       3d
```
### 页面访问
我在node上单独部署了nginx,node具有和集群在同一集群网络，可以使用service域名做upstream。可借鉴我的，也可以使用ingress、node port、负载均衡等方式访问。代码如下：
```
    upstream grafana{
		server grafana.monitoring.svc.cluster.local:3000;
    }
    upstream alertmanager{
		server alertmanager-main.monitoring.svc.cluster.local:9093;
    }
    upstream prometheus{
		server prometheus-k8s.monitoring.svc.cluster.local:9090;
    }

    server {
	listen 80;
	server_name grafana.vincent.com;
	location / {
		proxy_pass http://grafana;
	    }
    }
    server {
	listen 80;
	server_name alertmanager.vincent.com;
	location / {
		proxy_pass http://alertmanager;
	    }
    }
    server {
	listen 80;
	server_name prometheus.vincent.com;
	location / {
		proxy_pass http://prometheus;
	    }
    }
```
在主机上做hosts解析【node_ip换为nginx的外网ip即可】
```
node_ip grafana.vincent.com   
node_ip alertmanager.vincent.com
node_ip prometheus.vincent.com
```

访问prometheus


访问alertmanager


访问grafana



### 鸣谢！
感谢[brancz](https://github.com/coreos/prometheus-operator)大神是prometheus operator的主要代码贡献者。

感谢[camilb](https://github.com/camilb/prometheus-kubernetes)大神在prometheus operator的基础上提供了一键简单快速的部署配置Prometheus在Kubernetes集群，并提前预制了Grafana dashboard模版，预制了告警。