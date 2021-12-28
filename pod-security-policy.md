# Overview

*Pod 安全策略（Pod Security Policy）* 是集群级别的资源。  Pod 安全策略在 Kubernetes v1.21 版本中被弃用，将在 v1.25 中删除

[PodSecurityPolicy](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.23/#podsecuritypolicy-v1beta1-policy) 对象定义了一组 Pod 运行时必须遵循的条件及相关字段的默认值，只有 Pod 满足这些条件 才会被系统接受。 Pod 安全策略允许管理员控制如下方面：

| 控制的角度                     | 字段名称                                                     |
| ------------------------------ | ------------------------------------------------------------ |
| 运行特权容器                   | [`privileged`](https://kubernetes.io/zh/docs/concepts/policy/pod-security-policy/#privileged) |
| 使用宿主名字空间               | [`hostPID`、`hostIPC`](https://kubernetes.io/zh/docs/concepts/policy/pod-security-policy/#host-namespaces) |
| 使用宿主的网络和端口           | [`hostNetwork`, `hostPorts`](https://kubernetes.io/zh/docs/concepts/policy/pod-security-policy/#host-namespaces) |
| 控制卷类型的使用               | [`volumes`](https://kubernetes.io/zh/docs/concepts/policy/pod-security-policy/#volumes-and-file-systems) |
| 使用宿主文件系统               | [`allowedHostPaths`](https://kubernetes.io/zh/docs/concepts/policy/pod-security-policy/#volumes-and-file-systems) |
| 允许使用特定的 FlexVolume 驱动 | [`allowedFlexVolumes`](https://kubernetes.io/zh/docs/concepts/policy/pod-security-policy/#flexvolume-drivers) |
| 分配拥有 Pod 卷的 FSGroup 账号 | [`fsGroup`](https://kubernetes.io/zh/docs/concepts/policy/pod-security-policy/#volumes-and-file-systems) |
| 以只读方式访问根文件系统       | [`readOnlyRootFilesystem`](https://kubernetes.io/zh/docs/concepts/policy/pod-security-policy/#volumes-and-file-systems) |
| 设置容器的用户和组 ID          | [`runAsUser`, `runAsGroup`, `supplementalGroups`](https://kubernetes.io/zh/docs/concepts/policy/pod-security-policy/#users-and-groups) |
| 限制 root 账号特权级提升       | [`allowPrivilegeEscalation`, `defaultAllowPrivilegeEscalation`](https://kubernetes.io/zh/docs/concepts/policy/pod-security-policy/#privilege-escalation) |
| Linux 权能字（Capabilities）   | [`defaultAddCapabilities`, `requiredDropCapabilities`, `allowedCapabilities`](https://kubernetes.io/zh/docs/concepts/policy/pod-security-policy/#capabilities) |
| 设置容器的 SELinux 上下文      | [`seLinux`](https://kubernetes.io/zh/docs/concepts/policy/pod-security-policy/#selinux) |
| 指定容器可以挂载的 proc 类型   | [`allowedProcMountTypes`](https://kubernetes.io/zh/docs/concepts/policy/pod-security-policy/#allowedprocmounttypes) |
| 指定容器使用的 AppArmor 模版   | [annotations](https://kubernetes.io/zh/docs/concepts/policy/pod-security-policy/#apparmor) |
| 指定容器使用的 seccomp 模版    | [annotations](https://kubernetes.io/zh/docs/concepts/policy/pod-security-policy/#seccomp) |
| 指定容器使用的 sysctl 模版     | [`forbiddenSysctls`,`allowedUnsafeSysctls`](https://kubernetes.io/zh/docs/concepts/policy/pod-security-policy/#sysctl) |

*Pod 安全策略* 由设置和策略组成，它们能够控制 Pod 访问的安全特征。这些设置分为如下三类：

- *基于布尔值控制* ：这种类型的字段默认为最严格限制的值。
- *基于被允许的值集合控制* ：这种类型的字段会与这组值进行对比，以确认值被允许。
- *基于策略控制* ：设置项通过一种策略提供的机制来生成该值，这种机制能够确保指定的值落在被允许的这组值中

## 启用 Pod 安全策略

Pod 安全策略实现为一种可选的 [准入控制器](https://kubernetes.io/zh/docs/reference/access-authn-authz/admission-controllers/#podsecuritypolicy)。 [启用了准入控制器](https://kubernetes.io/zh/docs/reference/access-authn-authz/admission-controllers/#how-do-i-turn-on-an-admission-control-plug-in) 即可强制实施 Pod 安全策略，不过如果没有授权认可策略之前即启用 准入控制器 **将导致集群中无法创建任何 Pod**。

由于 Pod 安全策略 API（`policy/v1beta1/podsecuritypolicy`）是独立于准入控制器 来启用的，对于现有集群而言，建议在启用准入控制器之前先添加策略并对其授权。

启动是需要在apiserver里面通过启动admission plugin来启动，如下所示

```
# /etc/kubernetes/manifests/kube-apiserver.yaml
apiVersion: v1
kind: Pod
metadata:
...
spec:
  containers:
  - command:
    - kube-apiserver
...
    - --enable-admission-plugins=NodeRestriction,PodSecurityPolicy
# change
```



## 授权策略 

PodSecurityPolicy 资源被创建时，并不执行任何操作。为了使用该资源，需要对 发出请求的用户或者目标 Pod 的 [服务账号](https://kubernetes.io/zh/docs/tasks/configure-pod-container/configure-service-account/) 授权，通过允许其对策略执行 `use` 动词允许其使用该策略。

### 通过 RBAC 授权 

[RBAC](https://kubernetes.io/zh/docs/reference/access-authn-authz/rbac/) 是一种标准的 Kubernetes 鉴权模式，可以很容易地用来授权策略访问。

首先，某 `Role` 或 `ClusterRole` 需要获得使用 `use` 访问目标策略的权限。 访问授权的规则看起来像这样：

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: <Role 名称>
rules:
- apiGroups: ['policy']
  resources: ['podsecuritypolicies']
  verbs:     ['use']
  resourceNames:
  - <要授权的策略列表>
```

接下来将该 `Role`（或 `ClusterRole`）绑定到授权的用户：

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: <binding name>
roleRef:
  kind: ClusterRole
  name: <role name>
  apiGroup: rbac.authorization.k8s.io
subjects:
# 授权命名空间下的所有服务账号（推荐）：
- kind: Group
  apiGroup: rbac.authorization.k8s.io
  name: system:serviceaccounts:<authorized namespace>
# 授权特定的服务账号（不建议这样操作）：
- kind: ServiceAccount
  name: <authorized service account name>
  namespace: <authorized pod namespace>
# 授权特定的用户（不建议这样操作）：
- kind: User
  apiGroup: rbac.authorization.k8s.io
  name: <authorized user name>
```

如果使用的是 `RoleBinding`（而不是 `ClusterRoleBinding`），授权仅限于 与该 `RoleBinding` 处于同一名字空间中的 Pods。 可以考虑将这种授权模式和系统组结合，对名字空间中的所有 Pod 授予访问权限。

```yaml
# 授权某名字空间中所有服务账号
- kind: Group
  apiGroup: rbac.authorization.k8s.io
  name: system:serviceaccounts
# 或者与之等价，授权给某名字空间中所有被认证过的用户
- kind: Group
  apiGroup: rbac.authorization.k8s.io
  name: system:authenticated
```

