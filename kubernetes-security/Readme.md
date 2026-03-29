## Выполнение ДЗ № 5
**1. Задание: "в namespace homework создать service account monitoring и дать ему доступ к эндпоинту /metrics кластера"**

Создала ServiceAccount: [`serviceAccount.yaml`](serviceAccount.yaml), 
запустила: 
```bash 
kubectl apply -f serviceAccount.yaml -n homework 
```
Создала доступ к метрикам [`сlusterRole.yaml`](clusterRole.yaml):
```bash
kubectl apply -f clusterRole.yaml -n homework 
```
Привязала права к ServiceAccount: [`ClusterRoleBinding.yaml`](ClusterRoleBinding.yaml):
```bash
kubectl apply -f ClusterRoleBinding.yaml -n homework 
```
Проверяю, что ресурсы созданы: 
1. Service Account
```bash 
kubectl get sa -n homework 
```
Ответ:
```bash
NAME         AGE
default      17d
monitoring   6m24s
```
2. ClusterRole: 
```bash
kubectl get clusterrole -n homework | grep monitoring
```
Ответ:
```bash
monitoring-metrics                                                     2026-03-29T12:15:21Z
system:monitoring                                                      2026-03-11T18:33:51Z
```
3. ClusterRoleBinding: 
```bash
kubectl get clusterrolebinding -n homework | grep monitoring
```
Ответ: 
```bash
monitoring-metrics-binding                                      ClusterRole/monitoring-metrics                                                     28m
```
Можно проверить командой: 
```bash
kubectl auth can-i get /metrics --as=system:serviceaccount:homework:monitoring
```
Ответ: 
```bash 
yes
```

**2. Задание: "изменить манифест deployment из прошлых ДЗ так, чтобы поды запускались под service account monitoring"**

Внесла изменения в [`deployment.yaml`](deployment.yaml)

Применила: 
```bash
kubectl apply -f deployment.yaml -n homework 
```
Проверка: 
```bash
kubectl get pods -n homework 
NAME                                   READY   STATUS    RESTARTS   AGE
homework-deployment-66784c9bb4-lr9pn   1/1     Running   0          4m31s
```
Кроме того, можно увидеть в описании пода: 
```bash
kubectl describe pods homework-deployment-66784c9bb4-lr9pn -n homework | grep monitoring
Service Account:  monitoring
```
Под теперь использует Service Account, все обращения к API будут идти от имени monitoring. 


**3. Задание: "В namespace homework создать service account с именем cd и дать ему роль admin в рамках namespace homework"**

Создала service accaunt - cd [`sa-cd.yaml`](sa-cd.yaml) и запустила: 
```bash
kubectl apply -f sa-cd.yaml -n homework 
```

В Kubernetes уже есть role - admin. 

Создаю и запускаю [`crbinding.yaml`](crbinding.yaml)
``bash
kubectl apply -f crbinding.yaml -n homework 
```
Проверка: 
```bash
kubectl auth can-i create pods --as=system:serviceaccount:homework:cd -n homework
```
Ответ: 
```bash 
yes
```
