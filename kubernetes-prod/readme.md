Выполнение ДЗ № 14

Для выполнения данного задания вам потребуется создать минимум 4 виртуальных машин в YC следущей конфигурации:
- Для master - 1 узел, 2vCPU, 8GB RAM
- Для worker – 3 узла, 2vCPU, 8GB RAM

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
