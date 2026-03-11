## Выполнение ДЗ № 2

1. **Создала namespace.yaml, запустила:**

```bash
kubectl apply -f namespase.yaml
```
Проверка:
```bash
kubectl get ns
```
Ответ с сервера:
```bash
NAME              STATUS   AGE
default           Active   22m
homework          Active   2m24s
kube-node-lease   Active   22m
kube-public       Active   22m
kube-system       Active   22m
```


