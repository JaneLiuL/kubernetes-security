

# Overview

kube-bench 是一个检查所部署的K8S是否符合CIS Benchmark的安全规则的工具。

# 安装

安装相对来说比较简单，直接在 https://github.com/aquasecurity/kube-bench/releases 目录下载所需要的rpm包安装即可。

运行 `kube-bench master` 可以查看master节点有什么不符合CIS benchmark的

```
# kube-bench master
...

1.3.1 Edit the Controller Manager pod specification file /etc/kubernetes/manifests/kube-controller-manager.yaml
on the master node and set the --terminated-pod-gc-threshold to an appropriate threshold,
for example:
--terminated-pod-gc-threshold=10

1.3.2 Edit the Controller Manager pod specification file /etc/kubernetes/manifests/kube-controller-manager.yaml
on the master node and set the below parameter.
--profiling=false
```



然后根据提示，加入我们想修复1.3.2 

那么我们去`/etc/kubernetes/manifests/kube-controller-manager.yaml`加上  `  - --profiling=false `即可

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: kube-controller-manager
    tier: control-plane
  name: kube-controller-manager
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-controller-manager
    ..
    - --leader-elect=true
    - --node-cidr-mask-size=24
    - --port=0
    - --profiling=false
```

然后再次运行这个命令检查是否生效

```
# kube-bench master |  grep 1.3.2
Command "master" is deprecated, this command will be retired soon. Please use the `run` command with `--targets=master` instead.
[PASS] 1.3.2 Ensure that the --profiling argument is set to false (Automated)
```



接下来我们检查 The ownership of directory /var/lib/etcd

我们先执行 ll /var/lib/etcd 发现用户跟组都是root

```
# ll /var/lib/etcd
total 0
drwx------ 4 root root 29 Nov 11 03:34 member
```

然后我们运行发现提示有问题，根据提示执行chown etcd:etcd /var/lib/etcd 再看看

```
kube-bench master | grep "/var/lib/etcd" -A5
Command "master" is deprecated, this command will be retired soon. Please use the `run` command with `--targets=master` instead.
For example, chown etcd:etcd /var/lib/etcd
```

现在就通过了



然后我们再去worker node里面去检查. The permissions of the kubelet configuration /var/lib/kubelet/config.yaml

我们执行发现提示有问题，根据提示执行chmod 644 /var/lib/kubelet/config.yaml再看看

```
kube-bench node | grep /var/lib/kubelet/config.yaml -B2
2.2.10 Run the following command (using the config file location identified in the Audit step)
chmod 644 /var/lib/kubelet/config.yaml
```

然后再检查发现已经pass了



最后我们在worker node里面检查. The --client-ca-file argument of the kubelet

我们执行发现提示有问题

```
# kube-bench node | grep client-ca-file
[PASS] 2.1.4 Ensure that the --client-ca-file argument is set as appropriate (Scored)
2.2.7 Run the following command to modify the file permissions of the --client-ca-file
2.2.8 Run the following command to modify the ownership of the --client-ca-file .
```

然后`vim /var/lib/kubelet/config.yaml` 改成一下即可

```
# /var/lib/kubelet/config.yaml
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
```

