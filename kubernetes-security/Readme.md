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
```bash
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

**4. Задание: "Создать kubeconfig для service account cd"**

Для этого сначала нужно получить токен: 
```bash
kubectl create token cd -n homework
```
Ответ: 
```bash
eyJhbGciOiJSUzI1NiIsImtpZCI6IjVNX09LMzRIbTBYdzdDc01JMlZMZ01PT0JsdVVZOHJpaTk4T1J6aXZxVDAifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzc0ODMxNzAzLCJpYXQiOjE3NzQ4MjgxMDMsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwianRpIjoiOThiMGY1ZmUtNTAyZS00NzhiLThiYmYtNzRlYmMwNTIzZDdkIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJob21ld29yayIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJjZCIsInVpZCI6IjVmMjFjODNiLTNiMDAtNDBlOC1iYTNiLTIxZWQ4ZGU0OWJlZCJ9fSwibmJmIjoxNzc0ODI4MTAzLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6aG9tZXdvcms6Y2QifQ.X4G8YRxEWx9l38I8YVxjV0bIVNHZcrGdcpCNpjC9u9YfsUdIqIf-x14ReNGdRPD4HJL7sJbAVSJvmANuiwKtx7aYx6uNT5PgLbX4APDsXAsr9UqUAcE2vx43IIbe-9WSE1bMFtJbEp53Veqk4WmfY2-GS5vt5kCVLu1aMUfJFdTT1EmCNxUmCdOcB-aziyHjxMtPUJfeZTkEGUqzkldqW9pqYGGhF-AjuyLwUu31VWOgyilSmQU05hQVmviyI_WV3rHWRjwR1w15bfbZ8vJgcQB337NERkRYzH8JC_sxWcSQhLP96Jt8nS63srQt8q9uQ2RVC_dHQriK7SSasOU4UQ
```
Нужно получить адрес API сервера: 
```bash
kubectl config view --minify
```
Отсюда взяли IP-адрес сервера: 
```bash
server: https://192.168.49.2:8443
```
Получаю CA сертификат (он у меня по информации из kubectl config view --minify в каталоге): 
 ```bash
 cat /home/nela/.minikube/ca.crt > ca.crt
 ```
 
Теперь используя эти параметры я создала service accaunt - cd [`cd-kubeconfig.yaml`](cd-kubeconfig.yaml) и запустила: 
Проверяю: 
```bash
KUBECONFIG=cd-kubeconfig.yaml kubectl get pods -n homework
NAME                                   READY   STATUS    RESTARTS   AGE
homework-deployment-66784c9bb4-lr9pn   1/1     Running   0          13h
```
Проверка прав: 
```bash
KUBECONFIG=cd-kubeconfig.yaml kubectl auth can-i create pods
yes
```
Т.о пользователь cd имеет доступ к ns homework без доступа к кластеру целиком. 


**5. Задание: "Сгенерировать для service account cd токен с временем действия 1 день и сохранить его в файл token"**

Получила токен для ServiceAccount - cd, ns - homework, срок жизни - 24h, сохранила в файл: 
```bash
kubectl create token cd -n homework --duration=24h > token
```

**6. Задание: "модифицировать deployment.yaml так, чтобы в процессе запуска pod происходило обращение к endpoint /metrics кластера (механика вызова не принципиальна), результат ответа сохранялся в файл metrics.html и содержимое этого файла можно было бы получить при обращении по адресу /metrics.html сервиса"**

Создала файл [`deployment.yaml`](task_C/deployment.yaml)

Применила. Проверяю: 
```bash 
ubectl get pods -n homework 
NAME                                   READY   STATUS    RESTARTS   AGE
homework-deployment-64df569fdb-7jc7d   1/1     Running   0          2m34s
```
=> под поднялся!

Проерим, что файл создался: 
```bash
kubectl exec -it homework-deployment-64df569fdb-7jc7d -n homework -- sh
Defaulted container "web-server" out of: web-server, init-content (init), fetch-metrics (init)
/ # ls /homework/
index.html    metrics.html
```
ПРоверяем с браузера: 
```bash
curl http://localhost:8000/
```
Ответ: 
```bash
<h1>Privet from Init Container!</h1>
```
а также в браузере (при запросе: http://localhost:8000) появился ответ: Privet from Init Container!
