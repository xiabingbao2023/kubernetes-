# kubernetes安装后验证

**查看全部pod**

   `kubectl get pod -A`

**添加全部的补全功能**

  `source <(kubectl completion bash)`

**查看所有节点**

`kubectl get nodes -o wide`

**查看所有pod**

kubectl get pods -A

## 创建pod



创建一个nginx的pod，pod能成功过running

```
kubectl run web --image nginx
```



```
pod/web created
```



```
kubectl get pods -o wide
```



```
NAME   READY   STATUS    RESTARTS   AGE   IP           NODE          NOMINATED NODE   READINESS GATES
web    1/1     Running   0          63s   10.244.1.2   k8s-worker1   <none>           <none>
```



## 创建service



给nginx pod创建一个service, 通过curl能访问这个service的cluster ip地址。

```
kubectl expose pod web  --port=80 --name=web-service
```



```
service/web-service exposed
```



```
kubectl get service
```



```
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes    ClusterIP   10.96.0.1       <none>        443/TCP   3h53m
web-service   ClusterIP   10.98.102.238   <none>        80/TCP    4s
```



```
curl 10.98.102.238
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
vagrant@k8s-master:~$
```



## 环境清理



```
$ kubectl delete service web-service
$ kubectl delete pod web
```