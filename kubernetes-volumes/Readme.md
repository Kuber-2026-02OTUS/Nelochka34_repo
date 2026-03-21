## Выполнение ДЗ № 4

1. **Задание: необходимо создать манифест pvc.yaml, описывающий PersistentVolumeClaim, запрашивающий хранилище с storageClass по-умолчанию**


Манифест [`pvc.yaml`](pvc.yaml) создан и применен: 
```bash
kubectl apply -f pvc.yaml -n homework
```
Проверка: 
```bash
kubectl get pvc -n homework 
```
Ответ с сервера:
```bash
NAME     STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
my-pvc   Bound    pvc-992229cd-9cd3-498a-9922-35163b436817   3Gi        RWO            standard       <unset>                 2m42s
```

2. **Задание: необходимо создать манифест cm.yaml для объекта типа configMap,описывающий произвольный набор пар ключ-знаение**

Манифест [`cm.yaml`](cm.yaml) создан и применен: 
```bash
kubectl apply -f cm.yaml 
```
Проверка: 
```bash
kubectl get configmaps my-config -n homework 
```
Ответ сервера: 
```bash
NAME        DATA   AGE
my-config   2      44s
```
а также: 
```bash 
kubectl describe configmaps my-config -n homework 
```
Ответ сервера: 
```bash
Name:         my-config
Namespace:    homework
Labels:       <none>
Annotations:  <none>

Data
====
app.properties:
----
log_level=info
max_connections=10


welcome_message:
----
Hello world!


BinaryData
====

Events:  <none>
```


