## Installation

```bash
# ubuntu
apt install sysdig
# centus
yum -y install sysdig
```



## 使用



### 保存到文件

```bash
# Sysdig允许您将捕获的事件保存到磁盘，以便以后可以分析它们。语法如下:
sysdig -w myfile

# 如果想将保存到文件中的事件数限制为100，可以使用-n标志:
 sysdig –n 100 –w myfile.scap
```

### 过滤

```

```



使用sysdig 来至少30秒 去过滤检查容器ID 为e6b959eac699的调用并且保存到 log里面，格式为

[timestamp], [uid], [processName]

```bash
sysdig -M 31 -p "*%evt.time,%user.uid,%proc.name" container.id=e6b959eac699
```



TODO:

看下下面的链接

https://wohin.me/tan-suo-sysdig-falco-rong-qi-huan-jing-xia-de-yi-chang-xing-wei-jian-ce-gong-ju/