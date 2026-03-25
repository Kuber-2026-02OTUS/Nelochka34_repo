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

3. **Задание: в манифесте deployment.yaml изменить спецификацию
volume типа emptyDir, который монтируется в init и
основной контейнер, на pvc, созданный в предыдущем
пункте**

Внесла изменения в манифест [`deployment.yaml`](deployment.yaml) (теперь используется PVC, данные сохраняются, том общий для всех контейнеров в поде): 
```bash
spec:
      volumes:
        - name: shared-storage
          persistentVolumeClaim:
            claimName: my-pvc
```
Применяю: 
```bash
kubectl apply -f deployment.yaml -n homework 
```
Проверяю: 
```bash
ubectl get pods -n homework 
```
Ответ с сервера: 
```bash 
kubectl get pods -n homework 
NAME                                  READY   STATUS    RESTARTS   AGE
homework-deployment-dff494fdd-72sgh   0/1     Running   0          8s
```
Проверяю (захожу в под): 
```bash
kubectl exec -it homework-deployment-dff494fdd-72sgh -n homework -- sh
```
смотрю внутри: 
```bash
ls /homework/
index.html
 ```
что и ожидалось! 


4. **Задание: в манифесте deployment.yaml добавить монтирование
ранее созданного configMap как volume к основному
контейнеру пода в директории /homework/conf, так, чтобы
его содержимое можно было получить, обратившись по url
/conf/file**

Внесла изменения в манифест [`deployment_cm.yaml`](deployment_cm.yaml) (теперь  ConfifMap подключается как volume, смонтирован в контейнер в /homework/conf, файл открывается по url). 

Применила созданный deployment: 
```bash 
kubectl apply -f deployment_cm.yaml -n homework 
```
Получила ответ: 
```bash
kubectl get pods -n homework 
NAME                                  READY   STATUS    RESTARTS   AGE
homework-deployment-9f9bd4559-6k47r   0/1     Running   0          5s
```
Убедилась, что welcome_message: "Hello world!" из cm.yaml станет внутри пода: 
```bash 
kubectl exec -it homework-deployment-9f9bd4559-6k47r -n homework -- sh

/ # ls /homework/conf/
app.properties   welcome_message

/ # cat /homework/conf/welcome_message 
Hello world!/ # 
```


5. **Задание С: создать манифест StorageClass.yaml, описывающий объект типа StorageClass с provisioner https://k8s.io/minikube-hostpath и reclaimPolicy Retain**

Создала манифест [`storageClass.yaml`](storageClass.yaml), применила: 
```bash
kubectl apply -f storageClass.yaml 
```
Проверила: 
```bash
kubectl get storageclasses

NAME                 PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
homework-storage     k8s.io/minikube-hostpath   Retain          Immediate           false                  23s
```

6. **Задание: изменить манифест pvc.yaml так, чтобы в нем запрашивалось хранилище созданного storageClass**

Создала манифест [`pvc_sc.yaml`](pvc_sc.yaml), применила: 
(для этого нужно было удалить старый pvc,  не удалялся, его держал deployment, пришлось удалить deployment, только потом применить pvc_sc.yaml)
```bash
kubectl apply -f pvc_sc.yaml -n homework 
```
Проверка: 
```bash 
kubectl get pvc -n homework 

NAME     STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS       VOLUMEATTRIBUTESCLASS   AGE
my-pvc   Bound    pvc-af36c2e2-55d7-4465-9fb8-882b59876853   3Gi        RWO            homework-storage   <unset>                 5m30s
```
Заметила, что в STORAGECLASS: homework-storage 

