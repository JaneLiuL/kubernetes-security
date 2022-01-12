## Overview

imagepolicywebhook ` 是admission controller中的一个插件， 

准入控制器是一段代码，它在对象持久性之前截取到Kubernetes API服务器的请求，但在请求经过身份验证和授权之后执行。



检查`imagepolicywebhook `是否被启用.

```bash
kube-apiserver -h | grep "enable-admission-plugins"
```

我们编辑`vi /etc/kubernetes/manifests/kube-apiserver.yaml`

```yaml
spec:
  containers:
  - command:
    - kube-apiserver    	    
    - --enable-admission-plugins=NodeRestriction,ImagePolicyWebhook
```

加上`ImagePolicyWebhook`， 建议是先把这个文件mv 到其他目录，加好再mv 回来



/etc/kubernetes/admission-controller.conf

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
- name: ImagePolicyWebhook
  path: imagepolicy.conf
```



imagepolicy.conf

```json
{
"imagePolicy": {
  	"kubeConfigFile": "/path/imagepolicy.kubeconfig",
  	"allowTTL": 50,
  	"denyTTL": 50,
  	"retryBackoff": 500,
  	"defaultAllow": false, # 改了这里
	}
}
```



imagepolicy.kubeconfig

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: /etc/kubernetes/admission-control/imagepolicywebhook.crt
    server: https://xxx.k8s.webhook:8090/scan
  name: k8s-webhook
contexts:
- context:
    cluster: kubernetes
    user: api-server
  name: k8s-webhook
current-context: k8s-webhook
kind: Config
preferences: {}
users:
- name: api-server
  user:
    client-certificate-data: /etc/kubernetes/admission-control/api-server-client.crt
    client-key-data: /etc/kubernetes/admission-control.api-server-client.key

```



我们继续编辑`vi /etc/kubernetes/manifests/kube-apiserver.yaml`

```yaml
spec:
  containers:
  - command:
    - kube-apiserver    	    
    - --admission-control-config-file=/etc/kubernetes/admission-controller.conf
```

加上--admission-control-config-file





## 验证

```
tail -n 10 /var/log/imagepolicy/roadrunner.log
```

