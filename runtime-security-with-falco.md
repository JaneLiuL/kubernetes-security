# Overview

Falco 是一个开源的专注于云原生运行时的安全工具。使用一下可以通过Falco保护以及监控系统：

1. 在运行时从内核解析Linux系统调用
2. 提供了一套强大的规则引擎，用于对Linux系统调用行为进行监控
3. 当违反规则时发出警报

Falco 检查些什么？

1. 使用特权容器的特权升级
2. 使用设置等工具更改名称空间
3. 读/写到众所周知的目录，如/etc、/usr/bin、/usr/sbin等
4. 创建符号链接
5. 所有权和模式变更
6. 意外的网络连接或套接字突变
7. 使用execve生成的进程
8. 执行shell二进制文件，如sh、bash、csh、zsh等
9. 执行SSH二进制文件，如SSH、scp、sftp等
10. 改变Linux coreutils可执行文件
11. 变异登录二进制文件
12. 改变shadowutil或passwd可执行文件，如shadowconfig, pwck, chpasswd, getpasswd, change, useradd等，以及其他



# 概念

## 规则（rules）

通过规则文件可以定义 Falco 的监控规则。

配置文件：

- 默认规则：`/etc/falco/falco_rules.yaml`
- 自定义规则：`/etc/falco/falco_rules.local.yaml`
- K8s审计规则：`/etc/falco/k8s_audit_rules.yaml`

### 规则Rules的Example

```yaml
############# rule ############
# rule: 删除或重命名shell历史文件
# condition中包含了modify_shell_history、truncate_shell_history、var_lib_docker_filepath等macro，及docker_binaries list的定义。
- rule: Delete or rename shell history
  desc: Detect shell history deletion
  condition: >
    (modify_shell_history or truncate_shell_history) and
       not var_lib_docker_filepath and
       not proc.name in (docker_binaries)
  output: >
    Shell history had been deleted or renamed (user=%user.name user_loginuid=%user.loginuid type=%evt.type command=%proc.cmdline fd.name=%fd.name name=%evt.arg.name path=%evt.arg.path oldpath=%evt.arg.oldpath %container.info)
  priority:
    WARNING
  tags: [process, mitre_defense_evasion]

############ macro ############
# 修改shell历史
- macro: modify_shell_history
  condition: >
    (modify and (
      evt.arg.name contains "bash_history" or
      evt.arg.name contains "zsh_history" or
      evt.arg.name contains "fish_read_history" or
      evt.arg.name endswith "fish_history" or
      evt.arg.oldpath contains "bash_history" or
      evt.arg.oldpath contains "zsh_history" or
      evt.arg.oldpath contains "fish_read_history" or
      evt.arg.oldpath endswith "fish_history" or
      evt.arg.path contains "bash_history" or
      evt.arg.path contains "zsh_history" or
      evt.arg.path contains "fish_read_history" or
      evt.arg.path endswith "fish_history"))

# 截断shell历史
- macro: truncate_shell_history
  condition: >
    (open_write and (
      fd.name contains "bash_history" or
      fd.name contains "zsh_history" or
      fd.name contains "fish_read_history" or
      fd.name endswith "fish_history") and evt.arg.flags contains "O_TRUNC")

- macro: var_lib_docker_filepath
  condition: (evt.arg.name startswith /var/lib/docker or fd.name startswith /var/lib/docker)      

- macro: modify
  condition: rename or remove  

############# list ############
- list: docker_binaries
  items: [docker, dockerd, exe, docker-compose, docker-entrypoi, docker-runc-cur, docker-current, dockerd-current]
```





## 警告（alerts）

Falco会针对监控规则进行检测，当检测到可疑行为时，会通过渠道打印日志、发送消息等渠道发送报警信息。

配置文件：`falco.yaml`的后半部分定义了告警发送渠道的配置。



# 安装

Falco是一个Linux安全工具，它使用系统调用来保护和监视系统。  

``` 
Falco可以用于Kubernetes运行时安全。 运行Falco最安全的方法是直接在主机系统上安装Falco，这样在Falco被破坏的情况下就可以与Kubernetes隔离。 然后Falco警报可以通过在Kubernetes中运行的只读代理来使用。 
```

Falco可以用于Kubernetes运行时安全。 运行Falco最安全的方法是直接在主机系统上安装Falco，这样在Falco被破坏的情况下就可以与Kubernetes隔离。 然后Falco警报可以通过在Kubernetes中运行的只读代理来使用。  



你也可以使用Helm在Kubernetes中直接运行Falco作为守护进程

 如果Falco是使用下面的包管理器构件安装的，你将会有以下内容:  

1.  Falco用户空间计划和观看通过systemd  
2. Falco驱动程序通过包管理器安装(可以是内核模块，也可以是eBPF，取决于主机)  
3. 正常和默认配置文件安装在/etc/falco  



具体可以参考官网安装：

https://falco.org/docs/getting-started/installation/#installing



# CKS 考试题目

Find a Pod running image httpd which modifies /etc/passwd .

Save the Falco logs for case  under /opt/course/2/falco.log in format time,container-id,container-name,user-name . No other information should be in any line. Collect the logs for at least 30 seconds.
Afterwards remove the threads (both 1 and 2) by scaling the replicas of the Deployments that control the offending Pods down to 0.



答案：

```
首先ssh进入到node里面
然后cd /etc/falco/目录下，
# grep log_syslog . -r
./falco.yaml:log_syslog: true
确定是true之后，然后我们可以去执行 # cat /var/log/syslog | grep falco
然后我们现在针对我们题目有一个Pod在尝试修改宿主机的/etc/passwd文件

然后我们就 grep falco  /var/log/syslog| grep passwd
然后就 如下所示他提示了容器ID container_id
# cat /var/log/syslog | grep falco | grep httpd | grep passwd
Sep 16 06:23:48 ubuntu2004 falco: 06:23:48.830962378: Error File below /etc opened for writing (user=root
user_loginuid=-1 command=sed -i $d /etc/passwd parent=sh pcmdline=sh -c echo hacker >> /etc/passwd; sed -i '$d'
/etc/passwd; true file=/etc/passwdngFmAl program=sed gparent=<NA> ggparent=<NA> gggparent=<NA>
container_id=b1339d5cc2de image=docker.io/library/httpd)

那么我们使用crictl ps -id 容器id去查pod ID
# crictl ps -id b1339d5cc2de
CONTAINER ID IMAGE NAME ... POD ID
b1339d5cc2dee f6b40f9f8ad71 httpd ... 595af943c3245
# crictl pods -id 595af943c3245
POD ID ... NAME NAMESPACE ...
595af943c3245 ... rating-service-68cbdf7b7-v2p6g team-purple ...
现在我们可以知道是哪个namespace的哪个pod

然后我们首先把这个pod缩容到0
kubectl -n  team-purple scale deployment rating-service --replicas 0

然后我们停止Falco服务，然后使用falco二进制重新启动服务
service falco stop

# falco
Thu Sep 16 06:33:11 2021: Falco version 0.29.1 (driver version 17f5df52a7d9ed6bb12d3b1768460def8439936d)
Thu Sep 16 06:33:11 2021: Falco initialized with configuration file /etc/falco/falco.yaml
Thu Sep 16 06:33:11 2021: Loading rules from file /etc/falco/falco_rules.yaml:
Thu Sep 16 06:33:11 2021: Loading rules from file /etc/falco/falco_rules.local.yaml:
Thu Sep 16 06:33:11 2021: Loading rules from file /etc/falco/k8s_audit_rules.yaml:
Thu Sep 16 06:33:12 2021: Starting internal webserver, listening on port 8765
06:33:17.382603204: Error Package management process launched in container (user=root user_loginuid=-1 command=apk
container_id=7a5ea6a080d1 container_name=nginx image=docker.io/library/nginx:1.19.2-alpine)

现在我们去/etc/falco目录下尝试查询"Package management process"这行日志的位置
cd /etc/falco
# grep -r "Package management process launched" .
./falco_rules.yaml: Package management process launched in container (user=%user.name user_loginuid=%user.loginuid

然后提示是falco_rules.yaml文件，编辑这个文件找到以下这段配置
```

配置如下

```
# Container is supposed to be immutable. Package management should be done in building the image.
- rule: Launch Package Management Process in Container
desc: Package management process ran inside container
condition: >
spawned_process
and container
and user.name != "_apt"
and package_mgmt_procs
and not package_mgmt_ancestor_procs
and not user_known_package_manager_in_container
output: >
Package management process launched in container (user=%user.name user_loginuid=%user.loginuid
command=%proc.cmdline container_id=%container.id container_name=%container.name
image=%container.image.repository:%container.image.tag)
priority: ERROR
tags: [process, mitre_persistence]
```

然后我们改成以下的配置

```
# Container is supposed to be immutable. Package management should be done in building the image.
- rule: Launch Package Management Process in Container
desc: Package management process ran inside container
condition: >
spawned_process
and container
and user.name != "_apt"
and package_mgmt_procs
and not package_mgmt_ancestor_procs
and not user_known_package_manager_in_container
output: >
Package management process launched in container evt.time%,%container.id,%container.name,%user.name
priority: ERROR
tags: [process, mitre_persistence]
```

然后我们就重启下falco服务

现在我们使用falco | grep "Package management"等待所有输出的日志保存到指定的文件 /opt/course/2/falco.log 即可



