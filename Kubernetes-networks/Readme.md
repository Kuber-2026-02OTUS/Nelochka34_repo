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
Применила: 
```bash
kubectl apply -f service.yaml
```
Проверила сервис: 
```bash
kubectl get svc -n homework
```
Получила ответ: 
```bash
NAME               TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
homework-service   ClusterIP   10.96.70.234   <none>        80/TCP    65s
```
Проверяю, что сервисы видят поды: 
```bash
kubectl get endpoints -n homework 
```
Ответ: 
```bash
Warning: v1 Endpoints is deprecated in v1.33+; use discovery.k8s.io/v1 EndpointSlice
NAME               ENDPOINTS                                            AGE
homework-service   10.244.0.21:8000,10.244.0.22:8000,10.244.0.23:8000   4m40s
```


