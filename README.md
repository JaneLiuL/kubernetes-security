# Kubernetes-security

这是为了学习K8S 安全创建的Git repo， 为此也复习了CKS的考试大纲。

## cluster-hardening





## system-hardening

限制对Kubernetes API的访问

使用RBAC最大程度的减少资源暴露

- [Apparmor](./apparmor.md)

- [gVisor--TODO]()

- [Network Policy](./networkpolicy.md)

  

## supply-chain-security

减小image的大小

保护供应链：将允许的镜像仓库列入白名单，对镜像进行签名和验证

分析文件及镜像安全隐患（例如Kubernetes的yaml文件，Dockerfile）

扫描图像，找出已知的漏洞

- [yaml静态安全扫描Static analysis - Kubesec](./kubesec.md)
- [针对集群服务检查kube-bench](./kube-bench.md)
- [针对image 或者文件系统的漏洞扫描Trivy](./trivy-vulnerabilities-scan.md)
- [dockerfile最佳实践](./dockerfile.md)



## minimise-microservice-vulnerabilities

设置适当的OS级安全域，例如使用PSP, OPA，安全上下文

管理Kubernetes Secret

在多租户环境中使用容器运行时 (例如gvisor, kata容器)

使用mTLS实现Pod对Pod加密

- [ImagePolicyWebhook](./imagepolicywebhook.md)
- [Pod Security Policy](./pod-security-policy.md)
- [Secrets](./secret.md)



## monitoring-logging-runtime-security

在主机和容器级别执行系统调用进程和文件活动的行为分析，以检测恶意活动

检测物理基础架构，应用程序，网络，数据，用户和工作负载中的威胁

检测攻击的所有阶段，无论它发生在哪里，如何扩散

对环境中的不良行为者进行深入的分析调查和识别

确保容器在运行时不变

使用审计日志来监视访问

- [审计-Audit](./audit.md)
- [Linux系统调用行为监控Falco](./falco.md)
- [系统级别的故障排查Sysdig](./sysdig.md)