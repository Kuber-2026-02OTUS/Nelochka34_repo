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

**Задание 1: создать helm-chart, позволяющий деплоить приложение, которое получилось в ДЗ № 3. При этом необходимо учесть:
    - основные параметры манифестах, такие как имена обьектов, имена контейнеров, используемые образы, хосты, порты, количество запускаемых реплик должны быть заданы как переменные в шаблонах и конфигурироваться через values.yaml либо через параметры при установке релиза;
    - репозиторий и тег образа не должны быть одним параметром;
    - пробы должны быть включаемы/отключаемы через конфиг;
    - в notes должно быть описано сообщение после установки релиза, отображающее адрес, по которому можно обратиться к сервису;
    - добавить в свой чарт сервис-зависисмость из доступных cummunity-чартов. Например mysql или redis.**

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



- Установила 
Манифест [`pvc.yaml`](pvc.yaml) создан и применен: 
