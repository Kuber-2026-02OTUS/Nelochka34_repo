## Выполнение ДЗ № 3

1. **Изменила readiness-пробу в манифесте [`deployment.yaml`](deployment.yaml)** 
Запустила: 
```bash
kubectl apply -f deployment.yaml 
```
Проверяю: 
```bash
kubectl get pods -n homework 
NAME                                   READY   STATUS    RESTARTS   AGE
homework-deployment-7d98b4b665-lmhcm   1/1     Running   0          74s
homework-deployment-7d98b4b665-n7txx   1/1     Running   0          74s
homework-deployment-7d98b4b665-pzw84   1/1     Running   0          61s
```
Включила forward и выполнила: 
```bash 
curl localhost:8000/index.html
```
Получила: 
```bash
<h1>Privet from Init Container!</h1>
```
2. **Создала манифест [`service.yaml`](service.yaml)**
