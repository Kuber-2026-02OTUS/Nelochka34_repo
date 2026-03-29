## Выполнение ДЗ № 5
1. Задание: "в namespace homework создать service account monitoring и дать ему доступ к эндпоинту /metrics кластера"

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
