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
Проверка прав: 
```bash
kubectl auth can-i --as=system:serviceaccount:default:mysql-sa '*' '*'
```
Ответ: yes

4. **Задание: Создать манифест deployment  для оператора, указав созданный ранее ServiceAccount и образ roflmaoinmysoul/mysql-operator:1.0.0**

Создала и запустила: [`deployment.yaml`](deployment.yaml) 

```bash