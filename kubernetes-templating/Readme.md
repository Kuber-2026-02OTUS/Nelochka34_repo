## Выполнение ДЗ № 6

- выполнила установку **Helm CLI**
```bash
helm version
version.BuildInfo{Version:"v3.20.1", GitCommit:"a2369ca71c0ef633bf6e4fccd66d634eb379b371", GitTreeState:"clean", GoVersion:"go1.25.8"}
```
- выполнила установку helmfile
```bash 
helmfile --version
helmfile version 1.4.3
```

**Задание 1: создать helm-chart, позволяющий деплоить приложение, которое получилось в ДЗ № 3. При этом необходимо учесть:**

    - основные параметры манифестах, такие как имена обьектов, имена контейнеров, используемые образы, хосты, порты, количество запускаемых реплик должны быть заданы как переменные в шаблонах и конфигурироваться через values.yaml либо через параметры при установке релиза;

    - репозиторий и тег образа не должны быть одним параметром;

    - пробы должны быть включаемы/отключаемы через конфиг;

    - в notes должно быть описано сообщение после установки релиза, отображающее адрес, по которому можно обратиться к сервису;

    - добавить в свой чарт сервис-зависисмость из доступных cummunity-чартов. Например mysql или redis.

1. Создала helm chart:
```bash
helm create homework-chart
```
Структура: 
```bash
tree
```
```bash
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── httproute.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml

4 directories, 11 files
```
Создала: 

2. [`values.yaml`](homework-chart/values.yaml)
    - repository и tag разделены согласно заданию
    - все параметры венесены наружу
3. [`namespace.yaml`](homework-chart/templates/namespace.yaml)
4. [`deployment.yaml`](homework-chart/templates/deployment.yaml)
5. [`service.yaml`](homework-chart/templates/service.yaml)
6. [`ingress.yaml`](homework-chart/templates/ingress.yaml)
7. [`NOTES.txt`](homework-chart/templates/NOTES.txt)
8. [`Chart.yaml`](homework-chart/Chart.yaml) - добавление зависимостей

Подтянула зависисмости: 
```bash 
helm dependency update
```
В результате в каталоге charts/ появился архив с Redis chart. 

Установила: 
```bash
helm install homework ./homework-chart   --set replicaCount=2   --set ingress.host=test.local
```
Получила: 
```bash
NAME: homework
LAST DEPLOYED: Mon Mar 30 22:02:49 2026
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Приложение успешно установлено!

Открой в браузере:

http://test.local

Или если ingress выключен:

kubectl port-forward svc/homework-service 8080:80

И открой:
http://localhost:8080
```

Проверяю Helm релиз: 
```bash
helm list 
```
Получила: 
```bash
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
homework        default         1               2026-03-30 22:02:49.22700224 +0300 MSK  deployed        homework-chart-0.1.0    1.16.0 
```
Смотрю, что создалось в Kubernetes:
```bash 
kubectl get all -n homework 
```
Получила: 
```bash
pod/homework-deployment-6df4d4d58f-s4j88   1/1     Running   1 (2m56s ago)   13h
pod/homework-deployment-6df4d4d58f-zj5w2   1/1     Running   1 (2m56s ago)   13h

NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/homework-service   ClusterIP   10.101.114.141   <none>        80/TCP    13h

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/homework-deployment   2/2     2            2           13h

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/homework-deployment-6df4d4d58f   2         2         2       13h
```
Поды - Running 
Сервис - запущен

Ingress:
```bash
kubectl get ingress -n homework
```
```bash
NAME               CLASS   HOSTS        ADDRESS   PORTS   AGE
homework-ingress   nginx   test.local             80      13h
```

Сделав форводинг: 
```bash
kubectl port-forward svc/homework-service 8080:80 -n homework
```
Могу открыть в браузере: 
```bash
http://localhost:8080
```
Получаю: 
```bash
<h1>Privet from Init Container!</h1>
```

Проверяю Helm notes: 
```bash
helm status homework 
```
Получаю текст из NOTES.txt

