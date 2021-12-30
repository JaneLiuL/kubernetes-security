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

$ vault kv put secret/data/dev pgusername=admin
Success! Data written to: secret/data/dev
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

现在我们需要定义Policy，然后这个Approle只能在某个路径读写

##### Policy

查看policy

```
export VAULT_ADDR='http://127.0.0.1:8200'
export VAULT_TOKEN='s.xxx'

vault policy list
vault policy read <policy-name>
```

新建policy

```
 vault policy write my-policy - << EOF
path "secret/data/dev/*" {
  capabilities = ["read", "create", "update"]
}

path "secret/data/prod" {
  capabilities = ["read"]
}
EOF
```

测试policy

```bash
$ export VAULT_TOKEN="$(vault token create -field token -policy=my-policy)"
/ $ vault token lookup | grep policies
policies            [default my-policy]

# 然后发现 在secret/data/dev跟其他目录都没有权限
$ vault kv put secret/data/dev跟其他目录都没有权限 testkey="my-long-password"
Error writing data to secret/data/dev: Error making API request.

URL: PUT http://127.0.0.1:8200/v1/secret/data/dev
Code: 403. Errors:

* 1 error occurred:
	* permission denied

# 测试成功，他只能在secret/data/dev/ 下面目录新建目录下才有权限
/ $ vault kv put secret/data/dev/base-infra testkey="my-long-password"
Success! Data written to: secret/data/dev/base-infra
```





mapping policy with approle

```

```

