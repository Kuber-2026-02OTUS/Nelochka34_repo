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
