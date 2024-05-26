# pod的一些简单操作

- 创建一个pod

  ```
  $ kubectl run web --image=nginx
  pod/web created
  ```

  

- 运行一个命令, kubectl run <pod name> –image=<image name> –command – <cmd> <arg1> … <argN>

​	`$ kubectl run client --image=busybox --command -- bin/sh -c "sleep 100000"`

- 创建多个容器的pod

一个pod是可以包含多个container的，如果要创建这样的pod，那么只能通过yaml文件实现，例如：

```
apiVersion: v1
kind: Pod
metadata:
   name: my-pod
spec:
   containers:
    - name: nginx
      image: nginx
    - name: client
      image: busybox
      command:
       - sh
       - -c
       - "sleep 1000000"
```



```
$ kubectl create -f my-pod.yml
$ kubectl get pod
NAME     READY   STATUS    RESTARTS   AGE
my-pod   2/2     Running   0          35s
```

- 查文档 kubernetes.io, 命令行帮助

```
kubectl explain pods | more
kubectl explain pod.spec | more
kubectl explain pod.spec.containers | more
```

- 检查文件是否有错误

  和正常情况一样处理客户端发送过来的请求，但是并不会把Object状态持久化存储到storage中

  ```
  $ kubectl apply -f nginx.yml --dry-run=server
  $ kubectl apply -f nginx.yml --dry-run=client
  ```

- 将运行的pod转换成为yml

```
$ kubectl run web --image=nginx --dry-run=client -o yaml
$ kubectl run web --image=nginx --dry-run=client -o yaml > nginx.yml
```

- 比较现在的yml与运行的pod的区别

显示当前要部署的manifest和集群中运行的有和不同，这样就知道如果apply会发生什么。

```
$ kubectl diff -f new-nginx.yml
```

- 获取pod列表

```
vagrant@k8s-master:~$ kubectl get pods
NAME     READY   STATUS    RESTARTS   AGE
client   1/1     Running   0          5m14s
web      1/1     Running   0          15m
vagrant@k8s-master:~$ kubectl get pods -o wide
NAME     READY   STATUS    RESTARTS   AGE     IP           NODE          NOMINATED NODE   READINESS GATES
client   1/1     Running   0          5m17s   10.244.2.4   k8s-worker2   <none>           <none>
web      1/1     Running   0          15m     10.244.1.2   k8s-worker1   <none>           <none>
```



通过 `-o yaml` 可以获取到具体一个pod的yaml定义文件

```
$ kubectl get pods client -o yaml
```

- 删除Pod

```
$ kubectl delete pod web
pod "web" deleted
```

- 获取pod详细信息

```
$ kubectl describe pod my-pod
```

- 进入容器执行命令

  - 对于只有单个容器的Pod， 执行date命令

  - ```
    vagrant@k8s-master:~$ kubectl get pods
    NAME     READY   STATUS    RESTARTS   AGE
    client   1/1     Running   0          38s
    my-pod   2/2     Running   0          6s
    vagrant@k8s-master:~$ kubectl exec client -- date
    Wed Jun  1 21:57:07 UTC 2022
    ```

    

  - 进入交互式shell

    ```
    vagrant@k8s-master:~$ kubectl exec client -- date
    Wed Jun  1 21:57:07 UTC 2022
    vagrant@k8s-master:~$ kubectl exec client -it -- sh
    / #
    / # ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
       inet 127.0.0.1/8 scope host lo
          valid_lft forever preferred_lft forever
       inet6 ::1/128 scope host
          valid_lft forever preferred_lft forever
    3: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1450 qdisc noqueue
       link/ether a6:56:08:ba:34:28 brd ff:ff:ff:ff:ff:ff
       inet 10.244.1.3/24 brd 10.244.1.255 scope global eth0
          valid_lft forever preferred_lft forever
       inet6 fe80::a456:8ff:feba:3428/64 scope link
          valid_lft forever preferred_lft forever
    / #
    ```

    

  - 对于具有多个容器的pod，需要通过 `-c` 指定要进入那个容器中

    ```
    vagrant@k8s-master:~$ kubectl get pods
    NAME     READY   STATUS    RESTARTS   AGE
    client   1/1     Running   0          3m16s
    my-pod   2/2     Running   0          2m44s
    vagrant@k8s-master:~$ kubectl exec my-pod -c
    client  nginx
    vagrant@k8s-master:~$ kubectl exec my-pod -c nginx -- date
    Wed Jun  1 21:59:58 UTC 2022
    ```

    ## API level log

    通过 `-v` 可以获取到每一个kubectl命令在API level的log，例如

    获取kubectl操作更详细的log, API level（ 通过 -v 指定）

    ```
    $ kubectl get pod <pod-name> -v 6   # 或者 7,8,9 不同的level，数值越大，得到的信息越详细
    ```

    

    `--watch` 持续监听kubectl操作，API level

    ```
    $ kubectl get pods <pod-name> --watch -v 6
    ```



# Init Containers

https://kubernetes.io/docs/concepts/workloads/pods/init-containers/

![init-container](https://learn-k8s-from-scratch.readthedocs.io/en/latest/_images/init-containers.png)

## why init containers

- 初始化
- 处理依赖，控制启动

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-init-containers
spec:
  initContainers:
  - name: init-service
    image: busybox
    command: ["sh", "-c", "echo waiting for sercice; sleep 4"]
  - name: init-database
    image: busybox
    command: ["sh", "-c", "echo waiting for database; sleep 4"]
  containers:
  - name: app-container
    image: nginx
```

# Static Pod

参考 https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/

## What is Static Pods?

- Managed by the kubelet on Node
- Static Pod manifests, `staticPodPath` in kubelet’s configuration, by default is `/etc/kubernetes/manifests`
- kubelet configuration file: `/var/lib/kubelet/config.yaml`
- pod can be ‘seen’ through API server, but can not be managed by API server

Control plane 的几个static pod

```
vagrant@k8s-master:~$ sudo ls /etc/kubernetes/manifests/
etcd.yaml  kube-apiserver.yaml      kube-controller-manager.yaml  kube-scheduler.yaml
vagrant@k8s-master:~$
```