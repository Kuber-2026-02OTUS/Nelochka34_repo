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
 cat /home/nela/.minikube/ca.crt
 ```
 Получила сертификат: 
 ```bash
AxMKbWluaWt1YmVDQTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAPYZ
qiNtr/g+U8Y1ORtXRwkogvfIwCmjVi/bpz+LxJ9812Vt8+rLGh7SHRusLKQDRlGX
ESNfpDUmP6COrf7Mq8ToODM78glX6d1b3m3mmj2ba3TP0DY1FwNclOmmumz+8vTw
R6MvvSyE+Lo9kEqONyafQUZIAdDYH+bL05jK56ggNzYuEdCgJSDEnTtnF3g8hHhd
EZigptkQaW9doeGLHTrHMhT8LXprmApR/8uT/vlLsAfO85+KdoIFdIqdQQWJlndK
Hhgslgb0wdyYPzDsp6TyaBzwz/iHkbzu3zRrOYmZaNT8M5Kgg6Pw0fBfh580wInt
nj2EZY6MCyQ7uz9j9Y8CAwEAAaNhMF8wDgYDVR0PAQH/BAQDAgKkMB0GA1UdJQQW
MBQGCCsGAQUFBwMCBggrBgEFBQcDATAPBgNVHRMBAf8EBTADAQH/MB0GA1UdDgQW
BBQ7H5yQ9sExAaD+HTYSJEv9qF4uTTANBgkqhkiG9w0BAQsFAAOCAQEA4H2oUwSU
Wxb/+A1s0d/UXZW0KQh4H4OW3D80sC0xLEETkODng+884COPYyhJlgbO67tKbuIS
JEeYeUkLAfc9jlSfLdmPbKYVwgM662XAFCGYyWNjhDlHC7Ha0E5Im5BTwvqKWGtA
ZWTHoh6CxzZL9hGDxyXN1xTKf0CCU4go7itctBoIz/q3j1WL8WykA/2b0sF5y0qz
uis79PmUJnJLzjFKDcy9HmFKmaILRrc0DajhbAL30pF4t1OWZEGamhwkgMGNIfaO
Ba4hfdxlODe0hiZ90wg0+F8IV6DDAQir5YKvP02Dfier8vSqOZfCUPpgwvoQyWHn
rC2hX8pxt8qG2g==
```
Теперь используя эти параметры я создала service accaunt - cd [`sa-cd.yaml`](sa-cd.yaml) и запустила: 