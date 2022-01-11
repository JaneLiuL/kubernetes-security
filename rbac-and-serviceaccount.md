题目

Create a service account named "seminar-sa" on the namespace "seminar"

Create a new role named "k8s-seminar" in the namespace "seminar" which only allows "create and update" operations only on resources of type "pods" and "deployments"

Create a new rolebinding name "k8s-seminar-bind" binding to the newly created role to the service account created previously named "seminar-sa".



答案

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: seminar-sa
  namespace: seminar
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: seminar
  name: k8s-seminar
rules:
- apiGroups: [""] # "" 标明 core API 组
  resources: ["pods"]
  verbs: ["get", "update"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "update"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: k8s-seminar-bind
  namespace: seminar
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: k8s-seminar
subjects:
- kind: ServiceAccount
  name: seminar-sa
```

也可以用命令代替，命令看起来更加整洁

```bash
kubectl create sa seminar-sa -n seminar
kubectl -n seminar create role k8s-seminar --verb=create,update --resource=pods,deployments
kubectl -n seminar create rolebinding k8s-seminar-bind --role=k8s-seminar  --serviceaccount=seminar:seminar-sa
```

