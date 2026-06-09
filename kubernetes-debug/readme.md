## Выполнение ДЗ № 13

Проверяю, что кластер созданный в прошлом ДЗ жив: 
```bash
c managed-kubernetes cluster list

**Задание1: Создать манифест, описывающий pod с distroless образом для создания контейнера, например kyos0109/nginx-distroless и применить его в кластере. Приложить манифест к результатам ДЗ **

Создала [`distroless-pod.yaml`](distroless-pod.yaml).
ПРименила: 
```bash
kubectl apply -f distroless-pod.yaml 
```
Проверяю: 
```bash
kubectl get pods                    
NAME               READY   STATUS    RESTARTS      AGE
nginx-distroless   1/1     Running   0             17s
s3-test-pod        1/1     Running   1 (16d ago)   16d
```
```bash
kubectl get pod nginx-distroless
NAME               READY   STATUS    RESTARTS   AGE
nginx-distroless   1/1     Running   0          61s
```

**Задание2: С помощью команды kubectl debug создайте эфемерный контейнер для отладки этого пода. Отладочный контейнер должен иметь доступ к пространству имен pid для основного контейнера пода.**

distroless-образы сложно отслеживать через kubectl exec, потому что там нет shell и debug tools. Создаю ephemeral container - временный контейнер для отладки. 
```bash
kubectl debug -it nginx-distroless \
  --image=busybox:1.36 \
  --target=nginx \
  -- sh
```
Внутри debug-контейнера выполнила (виден процесс основного контейнера): 
```bash
ps

PID   USER     TIME  COMMAND
    1 root      0:00 nginx: master process nginx -g daemon off;
    7 101       0:00 nginx: worker process
   21 root      0:00 sh
   27 root      0:00 ps
```
Проверяю, что ephemeral container добавился в под: 
```bash
kubectl describe pod nginx-distroless
....
ephemeralContainers:
  - image: busybox
    imagePullPolicy: Always
    name: debugger-cwfmq
    resources: {}
    stdin: true
    targetContainerName: nginx
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    tty: true
```
**Задание3: Получите доступ к файловой системе отлаживаемого контейнера из эфемерного. Приложите к результатам ДЗ ввод команд ls –la для директории /etc/nginx **

Зашла в эфемерный контейнер: 
```bash
kubectl debug -it nginx-distroless \
  --image=busybox:1.36 \
  --target=nginx \
  -- sh

```
Процессы внутри контейнера: 
```bash
ps
PID   USER     TIME  COMMAND
    1 root      0:00 nginx: master process nginx -g daemon off;
    7 101       0:00 nginx: worker process
   28 root      0:00 sh
   34 root      0:00 ps
```
PID основного контейнера - 1. 
Для просмотра его файловой системы из эфемерного контейнера использую PID: 
```bash
ls -la /proc/1/root/etc/nginx/

total 48
drwxr-xr-x    3 root     root          4096 Oct  5  2020 .
drwxr-xr-x    1 root     root          4096 Jun  9 08:32 ..
drwxr-xr-x    2 root     root          4096 Oct  5  2020 conf.d
-rw-r--r--    1 root     root          1007 Apr 21  2020 fastcgi_params
-rw-r--r--    1 root     root          2837 Apr 21  2020 koi-utf
-rw-r--r--    1 root     root          2223 Apr 21  2020 koi-win
-rw-r--r--    1 root     root          5231 Apr 21  2020 mime.types
lrwxrwxrwx    1 root     root            22 Apr 21  2020 modules -> /usr/lib/nginx/modules
-rw-r--r--    1 root     root           643 Apr 21  2020 nginx.conf
-rw-r--r--    1 root     root           636 Apr 21  2020 scgi_params
-rw-r--r--    1 root     root           664 Apr 21  2020 uwsgi_params
-rw-r--r--    1 root     root          3610 Apr 21  2020 win-utf
```
**Задание4: Запустите в отладочном контейнере команду tcpdump -nn -i any -e port 80 (или другой порт, если у вас приложение на нем). Вполните несколко сетевых обращений к nginx в отлаживаемом поде любым удобным вам способом. Убедитесь, что tcpdump отображает сетевые пакеты этих подключений.**
Запускаю debug-контейнер с tcpdump: 
```bash
kubectl debug -it nginx-distroless \
  --image=nicolaka/netshoot \
  --target=nginx \
  -- bash
```
Внутри контейнера запускаю: 
```bash
tcpdump -nn -i any -e port 80
```
Во втором терминале создаю трафик на Под: 
```bash
get pod nginx-distroless -o wide
NAME               READY   STATUS    RESTARTS   AGE   IP              NODE                        NOMINATED NODE   READINESS GATES
nginx-distroless   1/1     Running   0          95m   10.112.133.12   cl1j1on23inasjurbm8h-ored   <none>           <none>

nela@Nelas-MacBook-Pro Nelochka34_repo % kubectl run curl-test --rm -it --image=curlimages/curl -- \
  curl http://10.112.133.12:80
```
Проверяю в первом терминале tcpdump: 
```bash
10:09:40.506675 eth0  Out ifindex 2 ba:06:13:fa:47:74 ethertype IPv4 (0x0800), length 72: 10.112.133.12.80 > 10.112.133.17.38100: Flags [.], ack 78, win 509, options [nop,nop,TS val 643553249 ecr 3720298399], length 0
10:09:40.507022 eth0  Out ifindex 2 ba:06:13:fa:47:74 ethertype IPv4 (0x0800), length 310: 10.112.133.12.80 > 10.112.133.17.38100: Flags [P.], seq 1:239, ack 78, win 509, options [nop,nop,TS val 643553250 ecr 3720298399], length 238: HTTP: HTTP/1.1 200 OK
```

**Задание5: С помощью kubectl debug создайте отладочный под для ноды, на которой запущен ваш под с distroless nginx. Получите доступ к файловой системе нод, и затем доступ к логам
пода с distrolles nginx. Приложите сами логи, и команду их
получения к резултатам ДЗ.**
Проверю сколько нод: 
```bash
kubectl get nodes
NAME                        STATUS   ROLES    AGE    VERSION
cl1j1on23inasjurbm8h-ixen   Ready    <none>   119m   v1.32.1
cl1j1on23inasjurbm8h-ored   Ready    <none>   119m   v1.32.1
cl1t8pdv6rtd4oqb6ekr-ysos   Ready    <none>   120m   v1.32.1
```
На какой ноде запущен под: 
```bash
kubectl get pod nginx-distroless -o wide
NAME               READY   STATUS    RESTARTS   AGE    IP              NODE                        NOMINATED NODE   READINESS GATES
nginx-distroless   1/1     Running   0          119m   10.112.133.12   cl1j1on23inasjurbm8h-ored   <none>           <none>
```
