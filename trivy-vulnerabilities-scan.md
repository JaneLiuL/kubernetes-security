

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

