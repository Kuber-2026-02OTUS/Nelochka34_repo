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
создаю debug под для этой ноды: 
```bash
kubernetes-debug % kubectl debug node/cl1j1on23inasjurbm8h-ored -it --image=ubuntu -- bash
--profile=legacy is deprecated and will be removed in the future. It is recommended to explicitly specify a profile, for example "--profile=general".
Creating debugging pod node-debugger-cl1j1on23inasjurbm8h-ored-45x56 with container debugger on node cl1j1on23inasjurbm8h-ored.
All commands and output from this session will be recorded in container logs, including credentials and sensitive information passed through the command prompt.
If you don't see a command prompt, try pressing enter.

```
перехожу в файловую систему: 
```bash
root@cl1j1on23inasjurbm8h-ored:/# chroot /host
# 
```
проверяю: 
```bash
# ls -la
total 68
drwxr-xr-x  18 root root  4096 Jun  9 08:31 .
drwxr-xr-x  18 root root  4096 Jun  9 08:31 ..
lrwxrwxrwx   1 root root     7 Dec  6  2024 bin -> usr/bin
drwxr-xr-x   4 root root  4096 Apr 20 13:50 boot
drwxr-xr-x  15 root root  3780 Jun  9 08:51 dev
drwxr-xr-x 104 root root  4096 Jun  9 08:31 etc
drwxr-xr-x   4 root root  4096 Apr 20 13:32 home
lrwxrwxrwx   1 root root     7 Dec  6  2024 lib -> usr/lib
lrwxrwxrwx   1 root root     9 Dec  6  2024 lib32 -> usr/lib32
lrwxrwxrwx   1 root root     9 Dec  6  2024 lib64 -> usr/lib64
lrwxrwxrwx   1 root root    10 Dec  6  2024 libx32 -> usr/libx32
drwx------   2 root root 16384 Dec  6  2024 lost+found
drwxr-xr-x   2 root root  4096 Dec  6  2024 media
drwxr-xr-x   2 root root  4096 Dec  6  2024 mnt
drwxr-xr-x   4 root root  4096 Apr 20 13:31 opt
dr-xr-xr-x 225 root root     0 Jun  9 08:31 proc
drwx------   5 root root  4096 Apr 20 13:32 root
drwxr-xr-x  34 root root   980 Jun  9 08:32 run
lrwxrwxrwx   1 root root     8 Dec  6  2024 sbin -> usr/sbin
drwxr-xr-x   2 root root  4096 Apr 20 13:52 srv
dr-xr-xr-x  13 root root     0 Jun  9 08:31 sys
drwxrwxrwt  10 root root  4096 Jun  9 10:37 tmp
drwxr-xr-x  14 root root  4096 Dec  6  2024 usr
drwxr-xr-x  12 root root  4096 Aug 11  2025 var
# 
```
Нахожу логи под-а на ноде: 
```bash
ls -la /var/log/pods/default_nginx-distroless*
total 28
drwxr-xr-x  7 root root 4096 Jun  9 10:07 .
drwxr-x--- 21 root root 4096 Jun  9 10:36 ..
drwxr-xr-x  2 root root 4096 Jun  9 10:00 debugger-98cjq
drwxr-xr-x  2 root root 4096 Jun  9 08:34 debugger-cwfmq
drwxr-xr-x  2 root root 4096 Jun  9 10:07 debugger-n6jtz
drwxr-xr-x  2 root root 4096 Jun  9 08:41 debugger-s955w
drwxr-xr-x  2 root root 4096 Jun  9 08:32 nginx
```
```bash
ls -la /var/log/pods/default_nginx-distroless_18cd19c2-ccbd-48cd-9457-9ca7252897b6/nginx
total 12
drwxr-xr-x 2 root root 4096 Jun  9 08:32 .
drwxr-xr-x 7 root root 4096 Jun  9 10:07 ..
-rw-r----- 1 root root  647 Jun  9 10:13 0.log
```
Смортю логи distroless nginx: 
```bash
cat /var/log/pods/default_nginx-distroless_18cd19c2-ccbd-48cd-9457-9ca7252897b6/nginx/0.log
2026-06-09T10:09:40.509545482Z stdout F 10.112.133.17 - - [09/Jun/2026:18:09:40 +0800] "GET / HTTP/1.1" 200 612 "-" "curl/8.20.0" "-"
2026-06-09T10:09:42.229669996Z stdout F 10.112.133.17 - - [09/Jun/2026:18:09:42 +0800] "GET / HTTP/1.1" 200 612 "-" "curl/8.20.0" "-"
2026-06-09T10:09:57.570611505Z stdout F 10.112.133.17 - - [09/Jun/2026:18:09:57 +0800] "GET / HTTP/1.1" 200 612 "-" "curl/8.20.0" "-"
2026-06-09T10:10:24.588279626Z stdout F 10.112.133.17 - - [09/Jun/2026:18:10:24 +0800] "GET / HTTP/1.1" 200 612 "-" "curl/8.20.0" "-"
2026-06-09T10:13:11.221791755Z stderr F 2026/06/09 18:13:11 [alert] 1#1: unknown process 43 exited on signal 9
```

**ЗаданиеС: Выполните команду strace для корневого процесса nginx в рассматриваемом ранее поде. Опишите в результатах ДЗ какие
операции необходимо сделать, для успешного вполнения
команд, и также приложите ее ввод к резултатам ДЗ.**

Использую новый debug контейнер (netshoot): 
```bash
kubectl debug -it nginx-distroless \
  --image=nicolaka/netshoot \
  --target=nginx \
  -- bash
Targeting container "nginx". If you don't see processes from this container it may be because the container runtime doesn't support this feature.
--profile=legacy is deprecated and will be removed in the future. It is recommended to explicitly specify a profile, for example "--profile=general".
Defaulting debug container name to debugger-2g2wp.
All commands and output from this session will be recorded in container logs, including credentials and sensitive information passed through the command prompt.
If you don't see a command prompt, try pressing enter.
nginx-distroless:~# 
```
Проверила наличие strace: 
```bash
which strace

/usr/bin/strace
```
```bash
ps -ef

PID   USER     TIME  COMMAND
    1 root      0:00 nginx: master process nginx -g daemon off;
    7 101       0:00 nginx: worker process
   60 root      0:00 bash
   71 root      0:00 ps -ef
```
