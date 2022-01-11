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

