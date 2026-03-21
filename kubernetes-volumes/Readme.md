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

