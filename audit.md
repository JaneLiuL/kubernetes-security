## Overview

Kubernetes *审计（Auditing）* 功能提供了与安全相关的、按时间顺序排列的记录集， 记录每个用户、使用 Kubernetes API 的应用以及控制面自身引发的活动。



## 审计策略/规则

你可以使用 `--audit-policy-file` 标志将包含策略的文件传递给 `kube-apiserver`。 如果不设置该标志，则不记录事件。 

审计级别有以下几点

- `None` - 符合这条规则的日志将不会记录。
- `Metadata` - 记录请求的元数据（请求的用户、时间戳、资源、动词等等）， 但是不记录请求或者响应的消息体。
- `Request` - 记录事件的元数据和请求的消息体，但是不记录响应的消息体。 这不适用于非资源类型的请求。
- `RequestResponse` - 记录事件的元数据，请求和响应的消息体。这不适用于非资源类型的请求。

一个审计策略/规则的 例子：

```yaml
apiVersion: audit.k8s.io/v1 # This is required.
kind: Policy
# Don't generate audit events for all requests in RequestReceived stage.
omitStages:
  - "RequestReceived"
rules:
  #  Log pod changes at RequestResponse level
  - level: RequestResponse
    resources:
    - group: ""
      # Resource "pods" doesn't match requests to any subresource of pods,
      # which is consistent with the RBAC policy.
      resources: ["pods"]
  # Log "pods/log", "pods/status" at Metadata level
  - level: Metadata
    resources:
    - group: ""
      resources: ["pods/log", "pods/status"]

  # Don't log requests to a configmap called "controller-leader"
  - level: None
    resources:
    - group: ""
      resources: ["configmaps"]
      resourceNames: ["controller-leader"]

  # Don't log watch requests by the "system:kube-proxy" on endpoints or services
  - level: None
    users: ["system:kube-proxy"]
    verbs: ["watch"]
    resources:
    - group: "" # core API group
      resources: ["endpoints", "services"]

  # Don't log authenticated requests to certain non-resource URL paths.
  - level: None
    userGroups: ["system:authenticated"]
    nonResourceURLs:
    - "/api*" # Wildcard matching.
    - "/version"

  # Log the request body of configmap changes in kube-system.
  - level: Request
    resources:
    - group: "" # core API group
      resources: ["configmaps"]
    # This rule only applies to resources in the "kube-system" namespace.
    # The empty string "" can be used to select non-namespaced resources.
    namespaces: ["kube-system"]

  # Log configmap and secret changes in all other namespaces at the Metadata level.
  - level: Metadata
    resources:
    - group: "" # core API group
      resources: ["secrets", "configmaps"]

  # Log all other resources in core and extensions at the Request level.
  - level: Request
    resources:
    - group: "" # core API group
    - group: "extensions" # Version of group should NOT be included.

  # A catch-all rule to log all other requests at the Metadata level.
  - level: Metadata
    # Long-running requests like watches that fall under this rule will not
    # generate an audit event in RequestReceived.
    omitStages:
      - "RequestReceived"		
```

## 审计后端

审计后端实现将审计事件导出到外部存储。 

可以选择把审计写到文件系统，例如像正常写日志

也可以选择把审计像事件一样写道外部的HTTP API / Webhook

### 写到导出日志

- `--audit-log-path` 指定用来写入审计事件的日志文件路径。不指定此标志会禁用日志后端。`-` 意味着标准化
- `--audit-log-maxage` 定义保留旧审计日志文件的最大天数
- `--audit-log-maxbackup` 定义要保留的审计日志文件的最大数量
- `--audit-log-maxsize` 定义审计日志文件的最大大小（兆字节）

如果你的集群控制面以 Pod 的形式运行 kube-apiserver，记得要通过 `hostPath` 卷来访问策略文件和日志文件所在的目录，这样审计记录才会持久保存下来。例如：

```yaml
  --audit-policy-file=/etc/kubernetes/audit-policy.yaml
  --audit-log-path=/var/log/kubernetes/audit/audit.log
```



使用HostPath 挂载

```
  - hostPath:
      path: /etc/kubernetes/audit
      type: DirectoryOrCreate
    name: auditing

    - mountPath: /etc/kubernetes/audit
      name: auditing
```

