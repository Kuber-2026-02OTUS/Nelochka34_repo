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
```
Ответ:
```bash
NAME                                   READY   STATUS    RESTARTS   AGE
homework-deployment-75b8b77665-6w8nh   1/1     Running   0          28s
homework-deployment-75b8b77665-hlr9n   1/1     Running   0          28s
homework-deployment-75b8b77665-k827k   1/1     Running   0          28s
```
Вывести все deployment в ns homework:
```bash
kubectl get deploy -n homework
```
Ответ:
```bash
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

3. **Задание со * **
- сначала удалю прошлый deployment (удалятся все поды):
```bash
kubectl delete deployments.apps homework-deployment -n homework
```
проверка:
```bash
kubectl get pods -n homework
No resources found in homework namespace.
```
- применяю новый манифест [`deployment_nod_select`](deployment_nod_select.yaml):
```bash
kubectl apply -f deployment_nod_select.yaml
deployment.apps/homework-deployment created
```
Проверяю поды:
```bash
kubectl get pods -n homework
NAME                                   READY   STATUS    RESTARTS   AGE
homework-deployment-5945dcfb5b-b9zc5   0/1     Pending   0          2m27s
homework-deployment-5945dcfb5b-wbn5c   0/1     Pending   0          2m27s
homework-deployment-5945dcfb5b-wwdl8   0/1     Pending   0          2m27s
```
```text
Все верно, kubernetes теперь запускает pod-ы только на нодах, у которых есть label (homework=true).
```
Проверить labels нод:
```bash
kubectl get nodes --show-labels
```
Такой метки нет.

```text
Как добавить метку:
```
```bash
kubectl label nodes minikube homework=true
```
после добавления метки можем заново применить:
```bash
kubectl apply -f deployment_nod_select.yaml
```
Проверяем:
```bash
kubectl get pods -n homework
```
Ответ: 
```bash
NAME                                   READY   STATUS    RESTARTS   AGE
homework-deployment-5945dcfb5b-b9zc5   1/1     Running   0          11m
homework-deployment-5945dcfb5b-wbn5c   1/1     Running   0          11m
homework-deployment-5945dcfb5b-wwdl8   1/1     Running   0          11m
```
