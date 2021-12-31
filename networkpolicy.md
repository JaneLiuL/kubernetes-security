在development命名空间内，创建名为pod-access的NetworkPolicy作用于名为products-service的Pod，只允许命名空间为test的Pod或者在任何命名空间内有environment:staging标签的Pod来访问。

如果Pod或者Namespace没有标签，需要先打上标签

```
kubectl label ns test name=testing
kubectl label pod products-service environment=staging
```

创建NetworkPolicy

```
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector:
    matchLabels:
      environment: staging
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: testing
    - podSelector:
        matchLabels:
          environment: staging  
```





There is a metadata service available at http://192.168.100.21:32000 on which Nodes can reach sensitive data, like cloud credentials for initialisation. By default, all Pods in the cluster also have access to this endpoint. The DevSecOps team has asked you to restrict access to thismetadata server.

In Namespace metadata-access :
Create a NetworkPolicy named metadata-deny which prevents egress to 192.168.100.21 for all Pods but still allows access to everything else
Create a NetworkPolicy named metadata-allow which allows Pods having label role: metadata-accessor to access endpoint 192.168.100.21

```
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: metadata-deny
spec:
  podSelector: {}
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0/32
        except:
        - 192.168.100.21/32
  policyTypes:
  - Egress
```





```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: metadata-deny
  namespace: metadata-access
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
      cidr: 0.0.0.0/0
      except:
      - 192.168.100.21/32
```



