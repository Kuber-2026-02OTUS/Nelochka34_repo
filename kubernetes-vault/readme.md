## Выполнение ДЗ № 11

Проверю кластер, созданный ранее: 
```bash
kubectl cluster-info

Kubernetes control plane is running at https://111.88.157.60
CoreDNS is running at https://111.88.157.60/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```
Проверю количество в нем нод: 
```bash
kubectl get nodes

NAME                        STATUS   ROLES    AGE   VERSION
cl1j1on23inasjurbm8h-ozuq   Ready    <none>   19d   v1.32.1
cl1t8pdv6rtd4oqb6ekr-ahav   Ready    <none>   20d   v1.32.1
```
А в задании требуется три. Смотрю сколько у меня групп нод: 
```bash
yc managed-kubernetes node-group list
+----------------------+----------------------+----------------+----------------------+---------------------+---------+------+
|          ID          |      CLUSTER ID      |      NAME      |  INSTANCE GROUP ID   |     CREATED AT      | STATUS  | SIZE |
+----------------------+----------------------+----------------+----------------------+---------------------+---------+------+
| catn91386rvjejm3lo68 | cat264jtvu7s0tk1og8s | infra-nodes    | cl1t8pdv6rtd4oqb6ekr | 2026-04-15 12:23:58 | RUNNING |    1 |
| catsj3bq7kg0q7fd4gsa | cat264jtvu7s0tk1og8s | workload-nodes | cl1j1on23inasjurbm8h | 2026-04-16 08:23:30 | RUNNING |    1 |
+----------------------+----------------------+----------------+----------------------+---------------------+---------+------+
```
У меня две node-group по 1 ноде. 
Увеличу workload-nodes до 2. 
```bash
yc managed-kubernetes node-group update catsj3bq7kg0q7fd4gsa \
  --fixed-size 2
```
Проверка: 
```bash
kubernetes-vault % kubectl get nodes

NAME                        STATUS   ROLES    AGE     VERSION
cl1j1on23inasjurbm8h-ekod   Ready    <none>   9m49s   v1.32.1
cl1j1on23inasjurbm8h-ozuq   Ready    <none>   19d     v1.32.1
cl1t8pdv6rtd4oqb6ekr-ahav   Ready    <none>   20d     v1.32.1
```
```bash
kubectl get nodes --show-labels

NAME                        STATUS   ROLES    AGE   VERSION   LABELS
cl1j1on23inasjurbm8h-ekod   Ready    <none>   15m   v1.32.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=standard-v3,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/zone=ru-central1-b,kubernetes.io/arch=amd64,kubernetes.io/hostname=cl1j1on23inasjurbm8h-ekod,kubernetes.io/os=linux,node.kubernetes.io/instance-type=standard-v3,node.kubernetes.io/kube-proxy-ds-ready=true,node.kubernetes.io/masq-agent-ds-ready=true,node.kubernetes.io/node-problem-detector-ds-ready=true,role=workload,topology.kubernetes.io/zone=ru-central1-b,yandex.cloud/node-group-id=catsj3bq7kg0q7fd4gsa,yandex.cloud/pci-topology=k8s,yandex.cloud/preemptible=false
cl1j1on23inasjurbm8h-ozuq   Ready    <none>   19d   v1.32.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=standard-v3,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/zone=ru-central1-b,kubernetes.io/arch=amd64,kubernetes.io/hostname=cl1j1on23inasjurbm8h-ozuq,kubernetes.io/os=linux,node.kubernetes.io/instance-type=standard-v3,node.kubernetes.io/kube-proxy-ds-ready=true,node.kubernetes.io/masq-agent-ds-ready=true,node.kubernetes.io/node-problem-detector-ds-ready=true,role=workload,topology.kubernetes.io/zone=ru-central1-b,yandex.cloud/node-group-id=catsj3bq7kg0q7fd4gsa,yandex.cloud/pci-topology=k8s,yandex.cloud/preemptible=false
cl1t8pdv6rtd4oqb6ekr-ahav   Ready    <none>   20d   v1.32.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=standard-v3,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/zone=ru-central1-b,kubernetes.io/arch=amd64,kubernetes.io/hostname=cl1t8pdv6rtd4oqb6ekr-ahav,kubernetes.io/os=linux,node.kubernetes.io/instance-type=standard-v3,node.kubernetes.io/kube-proxy-ds-ready=true,node.kubernetes.io/masq-agent-ds-ready=true,node.kubernetes.io/node-problem-detector-ds-ready=true,role=infra,topology.kubernetes.io/zone=ru-central1-b,yandex.cloud/node-group-id=catn91386rvjejm3lo68,yandex.cloud/pci-topology=k8s,yandex.cloud/preemptible=false
```
labels на нодах: 
- role=workload на нодах: 
        cl1j1on23inasjurbm8h-ekod
        cl1j1on23inasjurbm8h-ozuq
- role=infra: 
        cl1t8pdv6rtd4oqb6ekr-ahav

**Задание: В namespace установите consul из helm-чарта https://github.com/hashicorp/consul-k8s с параметрами 3 реплики для сервера. Приложите команду установки чарта и файл с переменными к результатам ДЗ**

1. создаю namespace consul: 
```bash
kubectl create namespace consul

namespace/consul created
```
Создала файл Создала [`consul.yaml`](consul.yaml).  Так как на infra ноде присутствовали taint, то были добавлены tolerations. Теперь разрешен запуск подов consul на infra-ноде. 

Добавила Helm repo HashiCorp:
```bash
helm repo add hashicorp https://helm.releases.hashicorp.com

"hashicorp" has been added to your repositories

nela@Nelas-MacBook-Pro kubernetes-vault % helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "argo" chart repository
...Successfully got an update from the "hashicorp" chart repository
...Successfully got an update from the "grafana" chart repository
Update Complete. ⎈Happy Helming!⎈
```
Установила consul со своими натсройками: 
```bash
helm install consul hashicorp/consul \
  --namespace consul \
  -f consul.yaml       
```
Проверяю: 
```bash
kubectl get pods -n consul

NAME                  READY   STATUS    RESTARTS   AGE
consul-client-6sdv6   1/1     Running   0          31m
consul-client-vzmqf   1/1     Running   0          31m
consul-server-0       1/1     Running   0          2m31s
consul-server-1       1/1     Running   0          2m54s
consul-server-2       1/1     Running   0          3m51s
```
