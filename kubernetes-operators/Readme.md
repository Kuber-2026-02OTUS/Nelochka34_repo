## Выполнение ДЗ № 7

1. **Задание: создать манифест объекта CRD со следующими параметрами:** 
```text
- объект уровня namespace
- api group - otus.homework
- kind - MySQL
- plural name - mysql
- версия - v1
```
Манифест [`crd.yaml`](crd.yaml) создан и применен: 

```bash
kubectl apply -f crd.yaml 
```
Проверка: 
```bash
kubectl get crd
NAME                  CREATED AT
mysql.otus.homework   2026-04-02T03:31:18Z
```
Поверка API: 
```bash
kubectl api-resources | grep mysql
mysql                               my           otus.homework/v1                  true         MySQL
```

2. **Задание: объект должен иметь следующие обязательные атрибуты и правила их валидации (все поля строковые):** 
```text
- Image - определяет docker-образ для создания
- Database - имя базы данных
- Password - пароль от БД
- Storage_size - размер хранилища под базу
```
в соответствии с этим создала новый манифест: 
Манифест [`crd2.yaml`](crd2.yaml) создан и применен: 
удалила прошлый, применила этот
```bash 
```bash
kubectl apply -f crd2.yaml 
```
3. **Задание: Создать манифесты ServiceAccount, ClusterRole и ClusterRoleBinding, описывающий сервис аккаунт с полными правами на доступ к api серверу.**

Cоздала
- [`sa.yaml`](sa.yaml) 
- [`cr.yaml`](cr.yaml) 
- [`crb.yaml`](crb.yaml) 

Применила: 
```bash
kubectl apply -f sa.yaml 
kubectl apply -f cr.yaml 
kubectl apply -f crb.yaml 
```
Проверка:
```bash
kubectl get sa
NAME                            AGE
default                         3h56m
mysql-operator-sa               32s
```
```bash
get clusterrole | grep mysql
mysql-operator-role                                                    2026-04-04T20:48:59Z
```
```bash
kubectl get clusterrolebinding | grep mysql
mysql-operator-binding                                          ClusterRole/mysql-operator-role                                                    2m19s
```

4. **Задание: Создать манифест deployment  для оператора, указав созданный ранее ServiceAccount и образ roflmaoinmysoul/mysql-operator:1.0.0**

Буду использовать образ percona/percona-server-mysql-operator:1.0.0, потому что у меня архитектура aarch64 и на этой архитектуре roflmaoinmysoul/mysql-operator:1.0.0 не работает. 

Создала и запустила: [`deployment.yaml`](deployment.yaml) 
Проверка: 
```bash
kubectl get pods -n default 
NAME                                             READY   STATUS    RESTARTS   AGE
percona-server-mysql-operator-6b689f4f4c-775p4   1/1     Running   0          6m38s
```

5. **Задание: Создать манифест кастомного объекта kind: MySQL валидный для применения (см. атрибуты CRD созданного ранее)**

обязательные поля:
	•	image — Docker-образ MySQL
	•	database — имя базы
	•	password — пароль
	•	storage_size — размер хранилища

Создала и запустила: [`mysql_instance.yaml`](mysql_instance.yaml) 

Проверка: 
```bash
kubectl get mysqls -n default 
NAME         AGE
example-db   35s
```
Все манифесты применила и убедилась, что все работатет. 
CRD создался. Оператор работает и при создании кастомного ресурса. При удалении объекта - все удаляется: 
```bash
kubectl delete mysql example-db -n default
mysql.otus.homework "example-db" deleted from default namespace
```