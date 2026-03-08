## Выполнение ДЗ № 1

1. **Установка Minikube по инструкции:** ```https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download```

Установка выполнена.  
Запуск minikube:  

```bash 
minikube start
``` 

#### Ответ с сервера: 
```text
😄  minikube v1.38.1 on Ubuntu 24.04 (amd64)
✨  Automatically selected the docker driver
❗  Starting v1.39.0, minikube will default to "containerd" container runtime. See #21973 for more info.

🧯  The requested memory allocation of 1966MiB does not leave room for system overhead (total system memory: 1966MiB). You may face stability issues.
💡  Suggestion: Start minikube with less memory allocated: 'minikube start --memory=1966mb'

📌  Using Docker driver with root privileges
👍  Starting "minikube" primary control-plane node in "minikube" cluster
🚜  Pulling base image v0.0.50 ...
💾  Downloading Kubernetes v1.35.1 preload ...
    > preloaded-images-k8s-v18-v1...:  272.45 MiB / 272.45 MiB  100.00% 2.61 Mi
    > gcr.io/k8s-minikube/kicbase...:  519.58 MiB / 519.58 MiB  100.00% 3.39 Mi
🔥  Creating docker container (CPUs=2, Memory=1966MB) ...
🐳  Preparing Kubernetes v1.35.1 on Docker 29.2.1 ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

2. **Установка kubectl по инструкции:** `https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/`

Проверка установленной версии: 
```bash 
kubectl version --client
```
Ответ: 
```text
Client Version: v1.35.2
Kustomize Version: v5.7.1
```
```
kubectl version --client --output=yaml
```
Ответ:
```text
clientVersion:
  buildDate: "2026-02-26T20:05:34Z"
  compiler: gc
  gitCommit: fdc9d74cbf2da6754ebf81d56f80ae2948cd6425
  gitTreeState: clean
  gitVersion: v1.35.2
  goVersion: go1.25.7
  major: "1"
  minor: "35"
  platform: linux/amd64
kustomizeVersion: v5.7.1
```
3. **Проверяю ноды:** 
```bash
kubectl get nodes
```
 Ответ с сервера:
```
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   10m   v1.35.1
```
Запускаю Манифест namespace.yaml для namespace с именем homework
```bash
kubectl apply -f namespace.yaml 
```
Ответ c сервера: 
```text
namespace/homework created
```

Проверка namespace: 

```bash
kubectl get ns
```
Ответ с сервера:
```
NAME              STATUS   AGE
default           Active   12m
homework          Active   110s
kube-node-lease   Active   12m
kube-public       Active   12m
kube-system       Active   12m
```
4. **Создала манифест pod.yaml, запустила:** 

```bash 
kubectl apply -f pod.yaml
``` 

Проверила, что запустился: 
```bash
kubectl get pods -n homework
```

Для проверки пробросила порт на свой хост: 

```bash
kubectl port-forward pod/homework-pod 8000:8000 -n homework
```

Проверила ответ с хоста: 

```bash 
curl -s http://localhost:8000

Получила ответ: 

`<h1>Privet from Init Container!</h1>`

**Иначе:** 

Запустила k9s, зашла в нужный namespace. Убедилась, что нужный Pod запущен. Далее выделила под - shift+f Далее - ok. 

В другом терминале выполнила: 

```bash
curl -I http://localhost:8000
```

Получила: 
```text
HTTP/1.0 200 OK
Server: SimpleHTTP/0.6 Python/3.9.25
Date: Sun, 08 Mar 2026 19:02:15 GMT
Content-type: text/html
Content-Length: 37
Last-Modified: Sun, 08 Mar 2026 18:42:17 GMT
```

Выполнила: 

```bash
curl -s http://localhost:8000
```
Получила: 
```text
<h1>Privet from Init Container!</h1>
```

**Убедилась, что работает!** 




