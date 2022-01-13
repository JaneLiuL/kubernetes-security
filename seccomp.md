## 查看OS 是否支持Seccomp

```bash
grep -i seccomp /boot/config-$(uname -r)
# CONFIG_HAVE_ARCH_SECCOMP_FILTER=y
# CONFIG_SECCOMP_FILTER=y
# CONFIG_SECCOMP=y
```



## 查看K8S 集群的 Seccomp 状态

```bash
kubectl run amicontained --image jess/amicontained -- amicontained
kubectl logs amicontained
# Container Runtime: docker
# Has Namespaces:
#         pid: true
#         user: false
# AppArmor Profile: docker-default (enforce)
# Capabilities:
#         BOUNDING -> chown dac_override fowner fsetid kill setgid setuid setpcap net_bind_service net_raw sys_chroot mknod audit_write setfcap
# Seccomp: disabled
# Blocked Syscalls (22):
#         MSGRCV SYSLOG SETPGID SETSID VHANGUP PIVOT_ROOT ACCT SETTIMEOFDAY UMOUNT2 SWAPON SWAPOFF REBOOT SETHOSTNAME SETDOMAINNAME INIT_MODULE DELETE_MODULE LOOKUP_DCOOKIE KEXEC_LOAD FANOTIFY_INIT OPEN_BY_HANDLE_AT FINIT_MODULE KEXEC_FILE_LOAD
# Looking for Docker.sock
```



## 对amicontained 启动Seccomp 

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: amicontained
spec:
 securityContext: # add the security context with seccomp profile
   seccompProfile:
     type: RuntimeDefault
 containers:
 - args:
   - amicontained
   image: jess/amicontained
   name: amicontained
 restartPolicy: Always
```

