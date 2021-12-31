

### 扫描image

```
trivy image [YOUR_IMAGE_NAME]
```



### 扫描文件系统的漏洞

```
trivy fs --security-checks vuln,config /dirs
例如
trivy fs --security-checks vuln,config myproject/
```

