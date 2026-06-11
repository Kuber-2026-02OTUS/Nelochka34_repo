Выполнение ДЗ № 14

Для выполнения данного задания вам потребуется создать минимум 4 виртуальных машин в YC следущей конфигурации:
- Для master - 1 узел, 2vCPU, 8GB RAM
- Для worker – 3 узла, 2vCPU, 8GB RAM

```bash
yc compute instance list
+----------------------+---------------+---------------+---------+----------------+-------------+
|          ID          |     NAME      |    ZONE ID    | STATUS  |  EXTERNAL IP   | INTERNAL IP |
+----------------------+---------------+---------------+---------+----------------+-------------+
| epd5to6udbvnmqjmo5jf | k8s-worker-01 | ru-central1-b | RUNNING | 89.169.181.48  | 10.129.0.20 |
| epdb92f18404bp0k7bl5 | k8s-worker-03 | ru-central1-b | RUNNING | 51.250.17.91   | 10.129.0.22 |
| epdktrvgjd7ru7unn1p5 | k8s-worker-02 | ru-central1-b | RUNNING | 111.88.147.171 | 10.129.0.34 |
| epdssgvjqf524iftcdb5 | k8s-master-01 | ru-central1-b | RUNNING | 103.76.54.11   | 10.129.0.14 |
+----------------------+---------------+---------------+---------+----------------+-------------+
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
