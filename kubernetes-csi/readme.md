## Выполнение ДЗ № 12

Проверю кластер, созданный ранее: 
```bash
kubectl cluster-info
``
```bash
kubectl get nodes
NAME                        STATUS   ROLES    AGE   VERSION
cl1j1on23inasjurbm8h-ekod   Ready    <none>   14d   v1.32.1
cl1j1on23inasjurbm8h-ozuq   Ready    <none>   34d   v1.32.1
cl1t8pdv6rtd4oqb6ekr-ahav   Ready    <none>   35d   v1.32.1
```


**Задание1: Создайте бакет в s3 object storage Yandex cloud. Он будет использоваться для монтирования volume внутри подов.**

Проверяю, что yc смотрит в нужное облако и каталог: 
 ```bash
 yc config list

token: 
cloud-id: b1gf9tc2sd9jtv1nqa84
folder-id: b1gbocdespuklnjtfkbf
compute-default-zone: ru-central1-a
```
Придумала имя бакета: 
```bash
export BUCKET_NAME=otus-k8s-s3-csi-nelochka34
```
Создала бакет: 
```bash
yc storage bucket create \
  --name $BUCKET_NAME

name: otus-k8s-s3-csi-nelochka34
folder_id: b1gbocdespuklnjtfkbf
anonymous_access_flags: {}
default_storage_class: STANDARD
versioning: VERSIONING_DISABLED
created_at: "2026-05-20T18:31:58.429018Z"
resource_id: e3ehq4f80qfec5b0t4c6
```
Проверяю, что бакет создан: 
```bash
yc storage bucket list    

+----------------------------+----------------------+----------+-----------------------+---------------------+
|            NAME            |      FOLDER ID       | MAX SIZE | DEFAULT STORAGE CLASS |     CREATED AT      |
+----------------------------+----------------------+----------+-----------------------+---------------------+
| loki-logs-bucket-nela      | b1gbocdespuklnjtfkbf |        0 | STANDARD              | 2026-04-15 14:52:45 |
| otus-k8s-s3-csi-nelochka34 | b1gbocdespuklnjtfkbf |        0 | STANDARD              | 2026-05-20 18:31:58 |
+----------------------------+----------------------+----------+-----------------------+---------------------+
```
Проверить информацию о бакете: 
```bash
yc storage bucket get --name $BUCKET_NAME
```

**Задание2: Создайте ServiceAccount для доступа к бакету с правами,
которые необходимы согласно инструкции YC и сгенерируйте
ключи доступа.**

```bash
export SA_NAME=s3-csi-sa

yc iam service-account create \
  --name $SA_NAME
Получила: 
id: ajed90j8bk1233udh822
folder_id: b1gbocdespuklnjtfkbf
created_at: "2026-05-20T18:38:39Z"
name: s3-csi-sa
```
Проверяю: 
```bash
yc iam service-account list

+----------------------+-----------+--------+---------------------+-----------------------+
|          ID          |   NAME    | LABELS |     CREATED AT      | LAST AUTHENTICATED AT |
+----------------------+-----------+--------+---------------------+-----------------------+
| aje8a6b6hef6q8f5pngv | k8s-sa    |        | 2026-04-14 14:03:35 | 2026-05-20 18:20:00   |
| ajed90j8bk1233udh822 | s3-csi-sa |        | 2026-05-20 18:38:39 |                       |
+----------------------+-----------+--------+---------------------+-----------------------+
```
Сохраняем ID sa: 
```bash
export SA_ID=$(yc iam service-account get $SA_NAME --format json | jq -r .id)
Проверка: 
echo $SA_ID

ajed90j8bk1233udh822
```
Выдаю права (storage.editor):
```bash
 yc resource-manager folder add-access-binding \
  --role storage.editor \
  --id $(yc config get folder-id) \
  --service-account-id $SA_ID
```
Создаю статические ключи доступа: 
```bash
yc iam access-key create \
  --service-account-name $SA_NAME

access_key:
  id: ajehhsv9ag4avqfcs2o2
  service_account_id: ajed90j8bk1233udh822
  created_at: "2026-05-20T18:53:31.071185381Z"
  key_id: XXX
secret: YYY
```
Сохранила секретный ключ: 
```bash
export ACCESS_KEY_ID=XXX
export SECRET_ACCESS_KEY=YYY
```
**Задание3: Создайте secret c ключами для доступа к Object Storage и
приложите манифест для проверки ДЗ** 

создала и применила [`secret.yaml`](secret.yaml).
Но чтобы не писать ключи в манифест, я применила secret прямо из CLI: 
```bash
kubectl create secret generic csi-s3-secret \
  -n kube-system \
  --from-literal=accessKeyID="$ACCESS_KEY_ID" \
  --from-literal=secretAccessKey="$SECRET_ACCESS_KEY" \
  --from-literal=endpoint="https://storage.yandexcloud.net"\
```
Проверка: 
```bash
kubectl get secret csi-s3-secret -n kube-system

NAME            TYPE     DATA   AGE
csi-s3-secret   Opaque   3      35s
```

**Задание4: Создайте storageClass описывающий класс хранилища и
приложите манифест для проверки ДЗ**
