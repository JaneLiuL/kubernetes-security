## Overview

Apparmor 是一个基于access controler的 Linux安装模块。 它使用` profiles` 来决定什么文件跟 应用需要的权限



安装只需要`apt install  apparmor-profiles`



apparmor_status用于查看AppArmor配置文件的当前状态。

```
apparmor_status
```

例如启动nginx profile

```
apparmor_parser -q nginx_apparmor
```





## 保护Pod

要指定要用其运行 Pod 容器的 AppArmor 配置文件，请向 Pod 的metadata 添加annotation ：

格式应该为

`container.apparmor.security.beta.kubernetes.io/<容器名字>: localhost/<profile>`

例如`container.apparmor.security.beta.kubernetes.io/hello: localhost/nginx_profile-3`

```
apiVersion: v1
kind: Pod
metadata:
  name: hello-apparmor
  annotations:
    container.apparmor.security.beta.kubernetes.io/hello: localhost/k8s-apparmor-example-deny-write
spec:
  containers:
  - name: hello
```

