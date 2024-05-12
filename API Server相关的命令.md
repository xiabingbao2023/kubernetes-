# API Server相关的命令

```
$ kubectl cluster-info
$ kubectl config view
$ kubectl config view --raw
$ kubectl config get-contexts
```

## API Object

API Object是通过API server可以操作的Kubernetes对象，它们代表了整个集群的状态，比如：

```
$ kubectl api-resources | more
$ kubectl api-resources --api-group=apps | more
$ kubectl api-versions | sort | more
```

### 如何操作API Object

两种模式

- Imperative Configuration (直接通过命令行去创建，操作)
- Declarative Configuration (通过YAML/JSON格式定义Manifest，把期望状态定义在文件中, 然后把文件传给API server)

```
apiVersion: v1
kind: Pod
metadata:
  name: web
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
```



```
$ kubectl apply -f nginx.yml  # Declarative Configuration
$ kubectl run web --image=nginx # Imperative Configuration 
```