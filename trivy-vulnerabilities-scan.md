

### 扫描image

```
trivy image [YOUR_IMAGE_NAME]
trivy image xxx | egrep -i "HIGH|CRITICAL"
```



### 扫描文件系统的漏洞

```
trivy fs --security-checks vuln,config /dirs
例如
trivy fs --security-checks vuln,config myproject/
```



题目： 请把qa namespace下Image安全漏洞级别为CRITICAL 的Pod删除

```bash
kubectl  get pod -n qa  -o yaml | grep "image:"

trivy image xxx | egrep -i "CRITICAL"

kubectl delete pod -n qa xx
```

