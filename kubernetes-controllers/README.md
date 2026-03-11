## Выполнение ДЗ № 2

1. **Создала [`namespace.yaml`](namespace.yaml), запустила:**

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
2. **Создала манифест [`deployment.yaml`](deployment.yaml)**

Запустила:
```bash
kubectl apply -f deployment.yaml
```
**Проверка:**
Покажем все поды:
```bash
kubectl get pods -n homework
NAME                                   READY   STATUS    RESTARTS   AGE
homework-deployment-75b8b77665-6w8nh   1/1     Running   0          28s
homework-deployment-75b8b77665-hlr9n   1/1     Running   0          28s
homework-deployment-75b8b77665-k827k   1/1     Running   0          28s
```
Вывести все deployment в ns homework:
```bash
kubectl get deploy -n homework
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
homework-deployment   3/3     3            3           40m
```
Статус процесса обновления deployment:
```bash
kubectl rollout status deployment homework-deployment -n homework
```
Ответ:
```
deployment "homework-deployment" successfully rolled out
```

