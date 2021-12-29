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

#### KV 

然后我们需要启动secret Engine

```
$ vault secrets enable -path=secret/ kv
Success! Enabled the kv secrets engine at: secret/

```

```
$ vault kv put secret/cvc pgusername=admin
Success! Data written to: secret/cvc
```





#### PKI

https://www.vaultproject.io/docs/secrets/pki

启动PKI secret Engine

```
vault secrets enable pki
```





## Auth Method

Reference: https://www.vaultproject.io/docs/auth/approle

Enable Auth Method

```
vault auth enable userpass
```



#### AppRole

先启动approle auth method

```
$ vault auth enable approle
Success! Enabled approle auth method at: approle/
```

Create a named role:

```
$ vault write auth/approle/role/my-role \
    secret_id_ttl=10m \
    token_num_uses=10 \
    token_ttl=20m \
    token_max_ttl=30m \
    secret_id_num_uses=40
```



为什么需要Approle 而不是直接用root token呢？ 是因为root token可以轻易的删库跑路

而Approle我们只需要给一组tech service/ component 一个Approle

然后这个Approle 只能在某个路径读写





```

```

