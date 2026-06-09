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
NAME               READY   STATUS                       RESTARTS      AGE
nginx-distroless   0/1     CreateContainerConfigError   0             8m11s
s3-test-pod        1/1     Running                      1 (16d ago)   16d

nela@Nelas-MacBook-Pro kubernetes-debug % kubectl get pod nginx-distroless
NAME               READY   STATUS                       RESTARTS   AGE
nginx-distroless   0/1     CreateContainerConfigError   0          8m41s
```
