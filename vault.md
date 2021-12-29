安装容器化部署vault 

https://www.vaultproject.io/docs/platform/k8s/helm

## 初始化

pod启动成功之后需要初始化

https://learn.hashicorp.com/tutorials/vault/getting-started-deploy

按下方初始化vault pod

```
export VAULT_ADDR='http://127.0.0.1:8200'
vault operator init

vault operator unseal
vault status
vault operator unseal
vault status
vault login s.xxx<root token>

```



## 启动Secret Engine

然后我们需要启动secret Engine

```
$ vault secrets enable -path=secret/ kv
Success! Enabled the kv secrets engine at: secret/

```

```
$ vault kv put secret/cvc pgusername=admin
Success! Data written to: secret/cvc
```





## PKI