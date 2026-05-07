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

**Задание1: В namespace установите consul из helm-чарта https://github.com/hashicorp/consul-k8s с параметрами 3 реплики для сервера. Приложите команду установки чарта и файл с переменными к результатам ДЗ**

1. создаю namespace consul: 
```bash
kubectl create namespace consul

namespace/consul created
```
Создала файл [`consul.yaml`](consul.yaml).  Так как на infra ноде присутствовали taint, то были добавлены tolerations. Теперь разрешен запуск подов consul на infra-ноде. 

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

**Задание2: В namespace vault установите hashicorp vault из helm-чарта https://github.com/hashicorp/vault-helm . Сконфигурируйте установку для использования ранее установленного consul в HA режимею. Приложите команду установки чарта и файл с переменными к результатам ДЗ**


```bash
kubectl get svc -n consul
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                                            AGE
consul-dns      ClusterIP   10.96.241.208   <none>        53/TCP,53/UDP                                                                      48m
consul-server   ClusterIP   None            <none>        8500/TCP,8502/TCP,8301/TCP,8301/UDP,8302/TCP,8302/UDP,8300/TCP,8600/TCP,8600/UDP   48m
consul-ui       ClusterIP   10.96.193.146   <none>        80/TCP                                                                             48m
```
Отсюда делаем вывод, что адрес server:  consul-server.consul.svc.cluster.local (по этому DNS имени сервис доступен в Kubernetes) порт 8500. 

Создала ns vault: 
```bash
kubectl create namespace vault

namespace/vault created
```
Добавила Helm repo HashiCorp
```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
```
Создала файл [`vault.yaml`](vault.yaml).
Установила Vault: 
```bash
kubernetes-vault % helm install vault hashicorp/vault \
  --namespace vault \
  -f vault.yaml       
```
Проверка: 
```bash
kubectl get pods -n vault

NAME                                   READY   STATUS    RESTARTS   AGE
vault-0                                0/1     Running   0          31s
vault-1                                0/1     Running   0          31s
vault-2                                0/1     Running   0          31s
vault-agent-injector-6b4f84b6c-p7ztw   1/1     Running   0          31s
```
- Vault установлен в namespace vault из официального Helm-чарта hashicorp/vault.
- Vault настроен в HA mode с 3 репликами.
- В качестве storage backend используется Consul:
consul-server.consul.svc.cluster.local:8500.

**Задание3: выполните инициализацию vault и распечатайте с помощью полученного unseal key все поды хранилища**
Запустила инициализацию: 
```bash
kubectl exec -n vault vault-0 -- vault operator init
```bash
Распечатала на vault-0: 
kubectl exec -n vault vault-0 -- vault operator unseal <Unseal_Key_1>

Key                Value
---                -----
Seal Type          shamir
Initialized        true
Sealed             true
Total Shares       5
Threshold          3
Unseal Progress    1/3
Unseal Nonce       edbbe4c1-8ca4-397b-4cc2-f217c4b2d6b0
Version            1.21.2
Build Date         2026-01-06T08:33:05Z
Storage Type       consul
HA Enabled         true
```
```bash
kubernetes-vault % kubectl exec -n vault vault-0 -- vault operator unseal <Unseal_Key_2>

Key                Value
---                -----
Seal Type          shamir
Initialized        true
Sealed             true
Total Shares       5
Threshold          3
Unseal Progress    2/3
Unseal Nonce       edbbe4c1-8ca4-397b-4cc2-f217c4b2d6b0
Version            1.21.2
Build Date         2026-01-06T08:33:05Z
Storage Type       consul
HA Enabled         true
```
```bash
kubectl exec -n vault vault-0 -- vault operator unseal <Unseal_Key_3>

Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    5
Threshold       3
Version         1.21.2
Build Date      2026-01-06T08:33:05Z
Storage Type    consul
Cluster Name    vault-cluster-eada3ee4
Cluster ID      418d6650-3172-0b46-64d5-7cd8baad6ee4
HA Enabled      true
HA Cluster      https://vault-0.vault-internal:8201
HA Mode         active
Active Since    2026-05-06T16:31:57.32001348Z
```
Проверка статуса: 
```bash
kubectl exec -n vault vault-0 -- vault status

Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    5
Threshold       3
Version         1.21.2
Build Date      2026-01-06T08:33:05Z
Storage Type    consul
Cluster Name    vault-cluster-eada3ee4
Cluster ID      418d6650-3172-0b46-64d5-7cd8baad6ee4
HA Enabled      true
HA Cluster      https://vault-0.vault-internal:8201
HA Mode         active
Active Since    2026-05-06T16:31:57.32001348Z
```
Аналогично распечатываются vault-1, vault-2. Покажу просто их статус: 
```bash
kubectl exec -n vault vault-1 -- vault status           

Key                    Value
---                    -----
Seal Type              shamir
Initialized            true
Sealed                 false
Total Shares           5
Threshold              3
Version                1.21.2
Build Date             2026-01-06T08:33:05Z
Storage Type           consul
Cluster Name           vault-cluster-eada3ee4
Cluster ID             418d6650-3172-0b46-64d5-7cd8baad6ee4
HA Enabled             true
HA Cluster             https://vault-0.vault-internal:8201
HA Mode                standby
Active Node Address    http://10.112.130.12:8200
```
```bash
kubectl exec -n vault vault-2 -- vault status

Key                    Value
---                    -----
Seal Type              shamir
Initialized            true
Sealed                 false
Total Shares           5
Threshold              3
Version                1.21.2
Build Date             2026-01-06T08:33:05Z
Storage Type           consul
Cluster Name           vault-cluster-eada3ee4
Cluster ID             418d6650-3172-0b46-64d5-7cd8baad6ee4
HA Enabled             true
HA Cluster             https://vault-0.vault-internal:8201
HA Mode                standby
Active Node Address    http://10.112.130.12:8200
```
Проверка всех подов: 
```bash
kubectl get pods -n vault

NAME                                   READY   STATUS    RESTARTS   AGE
vault-0                                1/1     Running   0          24m
vault-1                                1/1     Running   0          24m
vault-2                                1/1     Running   0          24m
vault-agent-injector-6b4f84b6c-p7ztw   1/1     Running   0          24m
```
- Vault был инициализирован командой vault operator init на pod vault-0.
- После инициализации были получены unseal keys и Initial Root Token.
- С помощью трёх unseal keys были распечатаны (unsealed) все pod’ы Vault: vault-0, vault-1 и vault-2.
- После распечатывания pod’ы перешли в состояние Ready, Vault работает в HA mode.

**Задание4: создать хранилище секретов otus/ с Secret Engine KV, а в нем секрет otus/cred, содержащий username='otus' password='asajkjkahs’**

Проверяю поды: 
```bash
kubectl get pods -n vault

NAME                                   READY   STATUS    RESTARTS      AGE
vault-0                                1/1     Running   1 (16h ago)   17h
vault-1                                1/1     Running   1 (16h ago)   17h
vault-2                                1/1     Running   1 (16h ago)   17h
vault-agent-injector-6b4f84b6c-p7ztw   1/1     Running   1 (16h ago)   17h
```
Все запущено => Хранилище распечатано. 
Логинимся в Vault: 
```bash
kubectl exec -it -n vault vault-0 -- sh
```
попала внутрь пода. Указываю адрес Vault: 
```bash
export VAULT_ADDR='http://127.0.0.1:8200'

vault login

ault status
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    5
Threshold       3
Version         1.21.2
Build Date      2026-01-06T08:33:05Z
Storage Type    consul
Cluster Name    vault-cluster-eada3ee4
Cluster ID      418d6650-3172-0b46-64d5-7cd8baad6ee4
HA Enabled      true
HA Cluster      https://vault-0.vault-internal:8201
HA Mode         active
Active Since    2026-05-07T09:26:38.184199654Z
```
Создала Secret Engine: 
``bash
vault secrets enable -path=otus kv

Success! Enabled the kv secrets engine at: otus/
``

Создала секрет: 
```bash
vault kv put otus/cred \
  username="otus" \
  password="asajkjkahs"

  Success! Data written to: otus/cred
```
Секрет создан. Проверяю: 
```bash
vault kv get otus/cred

====== Data ======
Key         Value
---         -----
password    asajkjkahs
username    otus
```
- Был создан Secret Engine типа KV по пути otus/.
- В хранилище создан секрет: otus/cred
    с параметрами:
        sername=otus
        assword=asajkjkahs

**Задание5: в namespace vault создать serviceAccount с именем vault-auth и ClusterRoleBinding для него с ролью system:auth-delegator. Приложить получившиеся манифесты к ДЗ**

Создала файл [`vault-auth.yaml`](vault-auth.yaml).
```bash
kubectl apply -f vault-auth.yaml 

serviceaccount/vault-auth created
clusterrolebinding.rbac.authorization.k8s.io/vault-auth-delegator created
```
Проверила sa: 
```bash
kubectl get serviceaccount vault-auth -n vault

NAME         SECRETS   AGE
vault-auth   0         42s
```
Проверила CRB: 
```bash
kubectl get clusterrolebinding vault-auth-delegator

NAME                   ROLE                                AGE
vault-auth-delegator   ClusterRole/system:auth-delegator   80s
```
- в namespace vault создан ServiceAccount vault-auth.
- создан ClusterRoleBinding vault-auth-delegator с ролью system:auth-delegator.Данная роль нужна Vault для проверки Kubernetes ServiceAccount токенов при использовании Kubernetes Auth.

**Задание6: в vault включить авторизацию auth/kubernetes и сконфигурировтаь ее используя токен и сертификат ранее созданного ServiceAccount**

Создала [`vault-auth-token.yaml`](vault-auth-token.yaml).
```bash
kubectl apply -f vault-auth-token.yaml

secret/vault-auth-token created
```
Проверка: 
```bash
kubectl get secret vault-auth-token -n vault

NAME               TYPE                                  DATA   AGE
vault-auth-token   kubernetes.io/service-account-token   3      23s
```
```bash
TOKEN_REVIEWER_JWT=$(kubectl get secret vault-auth-token -n vault -o jsonpath='{.data.token}' | base64 --decode)
KUBE_CA_CERT=$(kubectl get secret vault-auth-token -n vault -o jsonpath='{.data.ca\.crt}' | base64 --decode)
```
Проверила, что переменные не пустые: 
```bash
echo "$TOKEN_REVIEWER_JWT" | cut -c1-30
echo "$KUBE_CA_CERT" | head
```
Зашла в Vault и залогинилась root token. Далее передала CA-сертификат внутрь пода vault-0:
```bash
kubernetes-vault % kubectl exec -i -n vault vault-0 -- sh -c 'cat > /tmp/kubernetes-ca.crt' <<< "$KUBE_CA_CERT"
```
Включила auth/kubernetes в Vault: 
```bash
kubectl exec -n vault vault-0 -- sh -c '
export VAULT_ADDR="http://127.0.0.1:8200"
vault auth enable kubernetes
'
Success! Enabled kubernetes auth method at: kubernetes/
```
Сконфигурировала Kubernetes Auth
```bash
kubectl exec -n vault vault-0 -- sh -c "
export VAULT_ADDR='http://127.0.0.1:8200'
vault write auth/kubernetes/config \
  token_reviewer_jwt='$TOKEN_REVIEWER_JWT' \
  kubernetes_host='https://kubernetes.default.svc:443' \
  kubernetes_ca_cert=@/tmp/kubernetes-ca.crt
"

Success! Data written to: auth/kubernetes/config
```
Проверка: 
```bash
kubectl exec -n vault vault-0 -- sh -c '
export VAULT_ADDR="http://127.0.0.1:8200"
vault read auth/kubernetes/config
'

Key                                  Value
---                                  -----
disable_iss_validation               true
disable_local_ca_jwt                 false
issuer                               n/a
kubernetes_ca_cert                   n/a
kubernetes_host                      https://kubernetes.default.svc:443
pem_keys                             []
token_reviewer_jwt_set               true
use_annotations_as_alias_metadata    false
```
Конфиг прочитан! 

**Задание7: создать и применить полтику otus-policy для секретов /otus/cred с capabilities = [“read”, “list”]. Файл .hcl с политикой приложить к ДЗ**

Создала [`otus-policy.hcl`](otus-policy.hcl).
Скопировала в под Vault: 
```bash
kubectl cp otus-policy.hcl vault/vault-0:/tmp/otus-policy.hcl
```
Применила policy: 
```bash
kubernetes-vault % kubectl exec -n vault vault-0 -- sh -c '
export VAULT_ADDR="http://127.0.0.1:8200"
vault policy write otus-policy /tmp/otus-policy.hcl
'

Success! Uploaded policy: otus-policy
```
Проверка, что policy создано: 
```bash
kubectl exec -n vault vault-0 -- sh -c '
export VAULT_ADDR="http://127.0.0.1:8200"
vault policy list
'

default
otus-policy
root
```
=> Все ок

Можно просмотреть содержимое: 
```bash
kubectl exec -n vault vault-0 -- sh -c '
export VAULT_ADDR="http://127.0.0.1:8200"
vault policy read otus-policy
'

path "otus/cred" {
  capabilities = ["read", "list"]
}
```
- Создана и применена Vault policy otus-policy.
- Политика разрешает read и list доступ к секрету otus/cred.

**Задание8: создать роль auth/kubernetes/role/otus в vault с использвоанием ServiceAccount vault-auth из namespace Vault и политикой otus-policy**

Создаю роль: 
```bash
kubectl exec -n vault vault-0 -- sh -c '
export VAULT_ADDR="http://127.0.0.1:8200"

vault write auth/kubernetes/role/otus \
  bound_service_account_names=vault-auth \
  bound_service_account_namespaces=vault \
  policies=otus-policy \
  ttl=24h
'
```
Проверяю, что роль создана: 
```bash
kubectl exec -n vault vault-0 -- sh -c '
export VAULT_ADDR="http://127.0.0.1:8200"
vault read auth/kubernetes/role/otus
'

Key                                         Value
---                                         -----
alias_name_source                           serviceaccount_uid
bound_service_account_names                 [vault-auth]
bound_service_account_namespace_selector    n/a
bound_service_account_namespaces            [vault]
policies                                    [otus-policy]
token_bound_cidrs                           []
token_explicit_max_ttl                      0s
token_max_ttl                               0s
token_no_default_policy                     false
token_num_uses                              0
token_period                                0s
token_policies                              [otus-policy]
token_ttl                                   24h
token_type                                  default
ttl                                         24h
```
- В Vault создана Kubernetes Auth role otus по пути auth/kubernetes/role/otus.
- Роль привязана к ServiceAccount vault-auth из namespace vault.
- При успешной Kubernetes-аутентификации выдаётся Vault token с политикой otus-policy.

**Задание9: установить External Secrets Operator из helm-чарта в namespace vault. Команду установки чарта и файл с переменными, если вы их используете, приложите к результатам ДЗ**

```bash
helm repo add external-secrets https://charts.external-secrets.io
helm repo update external-secrets
```
Создала [`external-secrets-values.yaml`](external-secrets-values.yaml).
Установила ESO в vault: 
```bash
helm upgrade --install external-secrets external-secrets/external-secrets \
  --namespace vault \
  -f external-secrets-values.yaml
```
Проверяю поды: 
```bash
kubectl get pods -n vault | grep external-secrets

external-secrets-675bb64586-hctkx                   1/1     Running   0             50s
external-secrets-cert-controller-6fb859d7bb-4tr6h   1/1     Running   0             50s
external-secrets-webhook-569bd94445-sc5wl           1/1     Running   0             50s
```
- External Secrets Operator установлен в namespace vault из Helm-чарта external-secrets/external-secrets.
- При установке включена установка CRD через параметр installCRDs: true.

**Задание10: создать и применить манифест crd объекта SecretStore в namespace vault, сконфигурированный для доступа к KV секретам Vault с использованием ранее созданной роли otus и сервис аккаунта vault-auth. Убедитесь, что созданный SecretStore успешно подключился к vault. Получившийся манифест приложите к результатам ДЗ**

Создала [`secretstore-vault.yaml`](secretstore-vault.yaml).
```bash 
kubectl apply -f secretstore-vault.yaml 

secretstore.external-secrets.io/vault-secretstore created
```
Проверка: 
```bash
kubectl get secretstore -n vault
NAME                AGE   STATUS   CAPABILITIES   READY
vault-secretstore   28s   Valid    ReadWrite      True
```

**Задание11: создать и применить манифест crd объекта External Secrets со следующими параметрами: 
- ns - vault
- SecretStore - созданный на прошлом шаге
- Target.name = otus-cred
- получает значения KV секрета /otus/cred из vault и отображает их в два ключа - username и password соответственно**

Создала [`ES-otus-cred.yaml`](ES-otus-cred.yaml).
Применила: 
```bash
kubectl apply -f ES-otus-cred.yaml

externalsecret.external-secrets.io/otus-cred created
```
Проверка: 
```bash
kubectl get externalsecret -n vault

NAME        STORETYPE     STORE               REFRESH INTERVAL   STATUS         READY   LAST SYNC
otus-cred   SecretStore   vault-secretstore   1m                 SecretSynced   True    35s
```
Проверила, что Kubernetes Secret создан: 
```bash
kubectl get secret otus-cred -n vault
NAME        TYPE     DATA   AGE
otus-cred   Opaque   2      2m49s
```
