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
