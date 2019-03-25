# k8s-ops
K8S运维笔记

#ingress-nginx
该ingress-nginx是一个模版，可以根据生产情况做调整，以上版本在我的环境可以正常运行。
```
[root@master ingress-nginx]# kubectl get all -n ingress-nginx
NAME                                           READY   STATUS    RESTARTS   AGE
pod/default-http-backend-554c497679-w7wh4      1/1     Running   0          46m
pod/hello-world-78f948f44c-mfrn4               1/1     Running   0          46m
pod/hello-world-78f948f44c-nsc24               1/1     Running   0          46m
pod/hello-world-78f948f44c-q77q8               1/1     Running   0          46m
pod/nginx-ingress-controller-f879d9b4f-wzddb   1/1     Running   0          46m

NAME                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/default-http-backend   ClusterIP   10.100.90.255    <none>        80/TCP                       46m
service/ingress-nginx          NodePort    10.103.103.181   <none>        80:31594/TCP,443:32676/TCP   46m
service/myservice              ClusterIP   10.111.111.96    <none>        80/TCP                       46m

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/default-http-backend       1/1     1            1           46m
deployment.apps/hello-world                3/3     3            3           46m
deployment.apps/nginx-ingress-controller   1/1     1            1           46m

NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/default-http-backend-554c497679      1         1         1       46m
replicaset.apps/hello-world-78f948f44c               3         3         3       46m
replicaset.apps/nginx-ingress-controller-f879d9b4f   1         1         1       46m
```
查看ingress规则

```
[root@master ingress-nginx]# kubectl describe ingress ingress-my-service -n ingress-nginx
Name:             ingress-my-service
Namespace:        ingress-nginx
Address:          
Default backend:  default-http-backend:80 (<none>)
Rules:
  Host  Path  Backends
  ----  ----  --------
  *     
        /mytest   myservice:80 (<none>)
                  default-http-backend:80 (<none>)
Annotations:
  kubectl.kubernetes.io/last-applied-configuration:  {"apiVersion":"extensions/v1beta1","kind":"Ingress","metadata":{"annotations":{"nginx.ingress.kubernetes.io/force-ssl-redirect":"false","n
ginx.ingress.kubernetes.io/rewrite-target":"/","nginx.ingress.kubernetes.io/ssl-redirect":"false"},"name":"ingress-my-service","namespace":"ingress-nginx"},"spec":{"rules":[{"http":{"paths":[{"backend":{"serviceName":"myservice","servicePort":80},"path":"/mytest"},{"backend":{"serviceName":"default-http-backend","servicePort":80},"path":null}]}}]}}
  nginx.ingress.kubernetes.io/force-ssl-redirect:  false
  nginx.ingress.kubernetes.io/rewrite-target:      /
  nginx.ingress.kubernetes.io/ssl-redirect:        false
Events:
  Type    Reason  Age   From                      Message
  ----    ------  ----  ----                      -------
  Normal  CREATE  48m   nginx-ingress-controller  Ingress ingress-nginx/ingress-my-service
```
浏览器验证

http://10.10.69.232:31594/mytest

http://10.10.69.232:31594/