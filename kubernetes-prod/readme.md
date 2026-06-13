## Выполнение ДЗ № 14

Для выполнения данного задания вам потребуется создать минимум 4 виртуальных машин в YC следущей конфигурации:
- Для master - 1 узел, 2vCPU, 8GB RAM
- Для worker – 3 узла, 2vCPU, 8GB RAM

Создала ВМ согласно заданию: 

```bash
yc compute instance list
+----------------------+---------------+---------------+---------+---------------+-------------+
|          ID          |     NAME      |    ZONE ID    | STATUS  |  EXTERNAL IP  | INTERNAL IP |
+----------------------+---------------+---------------+---------+---------------+-------------+
| epd5to6udbvnmqjmo5jf | k8s-worker-01 | ru-central1-b | RUNNING | 130.193.55.98 | 10.129.0.20 |
| epdb92f18404bp0k7bl5 | k8s-worker-03 | ru-central1-b | RUNNING | 130.193.40.93 | 10.129.0.22 |
| epdktrvgjd7ru7unn1p5 | k8s-worker-02 | ru-central1-b | RUNNING | 130.193.40.81 | 10.129.0.34 |
| epdssgvjqf524iftcdb5 | k8s-master-01 | ru-central1-b | RUNNING | 130.193.52.73 | 10.129.0.14 |
+----------------------+---------------+---------------+---------+---------------+-------------+

```
На сегодняшний день последняя актуальная версия Kubernetes: Latest Release:1.36.1 (released: 2026-05-13). 
Значит будем ставить предыдущую - 1.35. 
Подключилась ко всем нодам: ssh <name>@IP
- на всех нодах проверила hostname: 
 ```
    k8s-master-01
    k8s-worker-01
    k8s-worker-02
    k8s-worker-03
```
- прописла sudo vim /etc/hosts: 
    ```bash
10.129.0.20 k8s-worker-01
10.129.0.22 k8s-worker-03
10.129.0.34 k8s-worker-02
10.129.0.14 k8s-master-01
```
- отключила swap:
- включила маршрутищацию через bridge
```bash
sudo tee /etc/modules-load.d/k8s.conf <<EOF
overlay
br_netfilter
EOF
```
- загрузила модули: 
```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```
- настрилла sysctl
```bash
sudo tee /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1
EOF
```
- применила: 
```bash
sudo sysctl --system
```
- установила containerd
```bash
sudo apt update
sudo apt install -y containerd
```
- создала конфиги: 
 ```bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```
- включила SystemdCgroup
```bash
sudo vim /etc/containerd/config.toml
    SystemdCgroup = true 
```
- перезапуслила: 
```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
```

**Установите containerd, kubeadm, kubelet, kubectl на все ВМ** 
containerd уже установила, настроила и перезапустила. 

- добавила репозиторий Kubernetes 1.35 (согласно заданию)
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```
добавила ключи: 
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.35/deb/Release.key \
| sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
добавила репозиторий: 
```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.35/deb/ /' \
| sudo tee /etc/apt/sources.list.d/kubernetes.list
```
обновила: 
```bash
sudo apt update
```
- установила Kubernetes 1.35 (выбираю последний патч 1.35.5)
```bash
sudo apt install -y \
kubelet=1.35.5-1.1 \
kubeadm=1.35.5-1.1 \
kubectl=1.35.5-1.1
```
- зафиксировала версию: 
```bash
sudo apt-mark hold kubelet kubeadm kubectl
```
проверяю: 
```bash
apt-mark showhold

google-compute-engine-oslogin
kubeadm
kubectl
kubelet
```
- включаю kubelet:
```bash
sudo systemctl enable kubelet
```
- проверяю: 
```bash
kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"35", EmulationMajor:"", EmulationMinor:"", MinCompatibilityMajor:"", MinCompatibilityMinor:"", GitVersion:"v1.35.5", GitCommit:"6636cbce3bbef91ff61d36658757179426f9e1b2", GitTreeState:"clean", BuildDate:"2026-05-12T09:53:04Z", GoVersion:"go1.25.9", Compiler:"gc", Platform:"linux/amd64"}
```
```bash
sudo kubectl version --client
Client Version: v1.35.5
Kustomize Version: v5.7.1
```
```bash
kubelet --version
Kubernetes v1.35.5
```
**Вполните kubeadm init на мастер-ноде**
мастер нода:  10.129.0.14/24 k8s-master-01
```bash
sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16
```
получила токен и discovery-token-ca-cert-hash

- настраиваю kubelet: 
```bash
mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
проверяю: 
```bash
kubectl get nodes

NAME            STATUS     ROLES           AGE     VERSION
k8s-master-01   NotReady   control-plane   4m35s   v1.35.5
```

**Установить Flannel - сетевой плагин**
```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

namespace/kube-flannel created
serviceaccount/flannel created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created
```
Проверяю: 
```bash
kubectl get pods -n kube-flannel
NAME                    READY   STATUS    RESTARTS   AGE
kube-flannel-ds-66trw   1/1     Running   0          52s
```

**Вполните kubeadm join на воркер нодах**

Выполняю join на каждом worker: 
```bash
kubeadm join 10.129.0.14:6443 --token XXX \
        --discovery-token-ca-cert-hash sha256:YYY
```
Проверяю на мастере: 
```bash
kubectl get nodes
NAME            STATUS   ROLES           AGE     VERSION
k8s-master-01   Ready    control-plane   74m     v1.35.5
k8s-worker-01   Ready    <none>          2m47s   v1.35.5
k8s-worker-02   Ready    <none>          80s     v1.35.5
k8s-worker-03   Ready    <none>          64s     v1.35.5
```
**Приложите к результатам ДЗ ввод команд kubectl get nodes -o wide, показващий статус и версию k8s всех нод кластера**
```bash
kubectl get nodes -o wide
NAME            STATUS   ROLES           AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
k8s-master-01   Ready    control-plane   77m     v1.35.5   10.129.0.14   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic   containerd://2.2.1
k8s-worker-01   Ready    <none>          5m49s   v1.35.5   10.129.0.20   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic   containerd://2.2.1
k8s-worker-02   Ready    <none>          4m22s   v1.35.5   10.129.0.34   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic   containerd://2.2.1
k8s-worker-03   Ready    <none>          4m6s    v1.35.5   10.129.0.22   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic   containerd://2.2.1
```

**Выполните обновление master ноды до последней актуалной версии k8s с помощью kubeadm**

 Сначала обновляется kubeadm, потом выполняется kubeadm upgrade, и только потом обновляются kubelet и  kubectl. 

 Снимаю блокировку: 
 ```bash
 sudo apt-mark unhold kubeadm kubelet kubectl
 ```
 Текущий репозиторий: 
 ```bash
cat /etc/apt/sources.list.d/kubernetes.list
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.35/deb/ /
```
меняю: 
```bash
sudo rm -f /etc/apt/sources.list.d/kubernetes.list

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
обновляю кэш: 
```bash
sudo apt update
```
Обновляю только kubeadm: 
```bash
sudo apt install -y kubeadm=1.36.1-1.1
```
Проверяю (на каждой ноде): 
```bash
kubeadm version

kubeadm version: &version.Info{Major:"1", Minor:"36", EmulationMajor:"", EmulationMinor:"", MinCompatibilityMajor:"", MinCompatibilityMinor:"", GitVersion:"v1.36.1", GitCommit:"756939600b9a7180fc2df6550a4585b638875e67", GitTreeState:"clean", BuildDate:"2026-05-12T09:53:52Z", GoVersion:"go1.26.2", Compiler:"gc", Platform:"linux/amd64"}
```
Смотрю план обновления: 
```bash
sudo kubeadm upgrade plan

[preflight] Running pre-flight checks.
[upgrade/config] Reading configuration from the "kubeadm-config" ConfigMap in namespace "kube-system"...
[upgrade/config] Use 'kubeadm init phase upload-config kubeadm --config your-config-file' to re-upload it.
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: 1.35.5
[upgrade/versions] kubeadm version: v1.36.1
[upgrade/versions] Target version: v1.36.1
[upgrade/versions] Latest version in the v1.35 series: v1.35.5

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   NODE            CURRENT   TARGET
kubelet     k8s-master-01   v1.35.5   v1.36.1
kubelet     k8s-worker-01   v1.35.5   v1.36.1
kubelet     k8s-worker-02   v1.35.5   v1.36.1
kubelet     k8s-worker-03   v1.35.5   v1.36.1

Upgrade to the latest stable version:

COMPONENT                 NODE            CURRENT   TARGET
kube-apiserver            k8s-master-01   v1.35.5   v1.36.1
kube-controller-manager   k8s-master-01   v1.35.5   v1.36.1
kube-scheduler            k8s-master-01   v1.35.5   v1.36.1
kube-proxy                                1.35.5    v1.36.1
CoreDNS                                   v1.13.1   v1.14.2
etcd                      k8s-master-01   3.6.6-0   3.6.8-0

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.36.1

_____________________________________________________________________


The table below shows the current state of component configs as understood by this version of kubeadm.
Configs that have a "yes" mark in the "MANUAL UPGRADE REQUIRED" column require manual config upgrade or
resetting to kubeadm defaults before a successful upgrade can be performed. The version to manually
upgrade to is denoted in the "PREFERRED VERSION" column.

API GROUP                 CURRENT VERSION   PREFERRED VERSION   MANUAL UPGRADE REQUIRED
kubeproxy.config.k8s.io   v1alpha1          v1alpha1            no
kubelet.config.k8s.io     v1beta1           v1beta1             no
_____________________________________________________________________
```

Выполняю обновление control-plane
```bash
sudo kubeadm upgrade apply v1.36.1
```
Обновляю kubelet и kubectl: 
```bash
sudo apt install -y \
kubelet=1.36.1-1.1 \
kubectl=1.36.1-1.1
```
перезапускаю kubelet: 
```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```
Проверяю: 
```bash
kubectl version 

Client Version: v1.36.1
Kustomize Version: v5.8.1
Server Version: v1.36.1
```
После оновления: 
```bash
kubectl get nodes

NAME            STATUS   ROLES           AGE    VERSION
k8s-master-01   Ready    control-plane   113m   v1.36.1
k8s-worker-01   Ready    <none>          42m    v1.35.5
k8s-worker-02   Ready    <none>          40m    v1.35.5
k8s-worker-03   Ready    <none>          40m    v1.35.5
```

**Последовательно введите из планирования все воркер-ноды, обновите их до последней актуалной версии и верните в планирование**

на мастере: 
```bash
kubectl drain k8s-worker-01 \
  --ignore-daemonsets \
  --delete-emptydir-data
```
Проверяю: 
```bash
kubectl get nodes

NAME            STATUS                     ROLES           AGE    VERSION
k8s-master-01   Ready                      control-plane   157m   v1.36.1
k8s-worker-01   Ready,SchedulingDisabled   <none>          86m    v1.35.5
k8s-worker-02   Ready                      <none>          85m    v1.35.5
k8s-worker-03   Ready                      <none>          84m    v1.35.5
```
На worker-01 обновляю: 
```bash 
sudo apt update

sudo apt install -y kubeadm=1.36.1-1.1
```
Проевряю: 
```bash
kubeadm version

kubeadm version: &version.Info{Major:"1", Minor:"36", EmulationMajor:"", EmulationMinor:"", MinCompatibilityMajor:"", MinCompatibilityMinor:"", GitVersion:"v1.36.1", GitCommit:"756939600b9a7180fc2df6550a4585b638875e67", GitTreeState:"clean", BuildDate:"2026-05-12T09:53:52Z", GoVersion:"go1.26.2", Compiler:"gc", Platform:"linux/amd64"}
```
Обновляю конфигурацию ноды: 
```bash
sudo kubeadm upgrade node
```

Обновляю kubelet и  kubectl: 
```bash
sudo apt install -y \
kubelet=1.36.1-1.1 \
kubectl=1.36.1-1.1
```
Перезапускаю kubelet:
```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```
Возвращаю ноду в планирование на мастере: 
```bash
kubectl uncordon k8s-worker-01
```
Проверяю: 
```bash
kubectl get nodes

NAME            STATUS   ROLES           AGE    VERSION
k8s-master-01   Ready    control-plane   164m   v1.36.1
k8s-worker-01   Ready    <none>          93m    v1.36.1
k8s-worker-02   Ready    <none>          91m    v1.35.5
k8s-worker-03   Ready    <none>          91m    v1.35.5
```
Аналогично делаю на worker-02, worker-03. 
```bash
kubectl get nodes
NAME            STATUS   ROLES           AGE     VERSION
k8s-master-01   Ready    control-plane   6h20m   v1.36.1
k8s-worker-01   Ready    <none>          5h9m    v1.36.1
k8s-worker-02   Ready    <none>          5h7m    v1.36.1
k8s-worker-03   Ready    <none>          5h7m    v1.36.1
```

**Приложите к результатам ДЗ ввод команд kubectl get nodes -o wide, показывающий статус и версию k8s всех нод кластера после обновлени**

```bash
kubectl get nodes -o wide
NAME            STATUS   ROLES           AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION              CONTAINER-RUNTIME
k8s-master-01   Ready    control-plane   6h24m   v1.36.1   10.129.0.14   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
k8s-worker-01   Ready    <none>          5h13m   v1.36.1   10.129.0.20   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
k8s-worker-02   Ready    <none>          5h12m   v1.36.1   10.129.0.34   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
k8s-worker-03   Ready    <none>          5h11m   v1.36.1   10.129.0.22   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
```
