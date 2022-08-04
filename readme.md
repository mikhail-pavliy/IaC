#Packer. Создание "Золотых образов" 

1. Первым делом установим Yandex.CLI 

```ruby
mikhail@mikhail-GL703VD:~/Desktop/otus/02-Packer$ curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash
Downloading yc 0.93.0
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 86.1M  100 86.1M    0     0  7932k      0  0:00:11  0:00:11 --:--:-- 9188k
Yandex Cloud CLI 0.93.0 linux/amd64
To complete installation, start a new shell (exec -l $SHELL) or type 'source "/home/mikhail/.bashrc"' in the current one
```
2. Установим Packer согласно интрукции на официальном сайте

```ruby
mikhail@mikhail-GL703VD:~/Desktop/otus/02-Packer$ wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
--2022-08-04 12:38:09--  https://apt.releases.hashicorp.com/gpg
Распознаётся apt.releases.hashicorp.com (apt.releases.hashicorp.com)… [sudo] пароль для mikhail: 18.165.122.122, 18.165.122.100, 18.165.122.19, ...
Подключение к apt.releases.hashicorp.com (apt.releases.hashicorp.com)|18.165.122.122|:443... соединение установлено.
HTTP-запрос отправлен. Ожидание ответа… 200 OK
Длина: 3195 (3,1K) [binary/octet-stream]
Сохранение в: ‘STDOUT’

-                                                 100%[==========================================================================================================>]   3,12K  --.-KB/s    за 0,002s  

/2022-08-04 12:38:09 (1,24 MB/s) - записан в stdout [3195/3195]
```

```ruby
mikhail@mikhail-GL703VD:~/Desktop/otus/02-Packer$ echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com impish main
```

```ruby
mikhail@mikhail-GL703VD:~/Desktop/otus/02-Packer$ sudo apt update && sudo apt install packer
```
3. Выполним подключение ЯО
```ruby
mikhail@mikhail-GL703VD:~/Desktop/otus/02-Packer$ yc init
```
далее вводим свои данные

после успешного подключения можем проверить доступность подключения командой  ``` yc config list ```
```ruby
mikhail@mikhail-GL703VD:~/Desktop/otus/02-Packer$ yc config list
token: ********************************
cloud-id: **********************
folder-id: *******************
compute-default-zone: ru-central1-a
```
4. Клонируем к себе репозиторий
```ruby
mikhail@mikhail-GL703VD:~/Desktop/otus/02-Packer$ git clone https://github.com/timurb/otus-packer 
```
в каталоге ```json``` создаем перменные ```YC_TOKEN``` и ```YC_FOLDER```

5. Создаем сервисную учетную запись для packer

```ruby

mikhail@mikhail-GL703VD:~/Desktop/otus/02-Packer/otus-packer/json$ yc iam service-account create --name packer --folder-id b1gra********
mikhail@mikhail-GL703VD:~/Desktop/otus/02-Packer/otus-packer/json$ yc iam service-account get ajeq8k7*********
id: ajeq8k7****************
folder_id: b1gra*********
created_at: "2022-08-04T08:54:34Z"
name: packer
```
```ruby
mikhail@mikhail-GL703VD:~/Desktop/otus/02-Packer/otus-packer/json$ yc resource-manager folder add-access-binding --id b1gra********** --role editor --service-account-id ajeq8***********
done (1s)
```

```ruby
mikhail@mikhail-GL703VD:~/Desktop/otus/02-Packer/otus-packer/json$ yc iam key create --service-account-id ajeq8k7l8oo3nup5f84b --output key.json
id: ajecm***********
service_account_id: ajeq8*********
created_at: "2022-08-04T09:00:52.275723258Z"
key_algorithm: RSA_2048
```
6. Вставляем путь к нашему ключу в template.json в "service_account_key_file" в блок builders
7. Запускаем сборку
```ruby
mikhail@mikhail-GL703VD:~/Desktop/otus/02-Packer/otus-packer/json$ packer build template.json

..............

Build 'yandex' finished after 2 minutes 48 seconds.

==> Wait completed after 2 minutes 48 seconds

==> Builds finished. The artifacts of successful builds are:
--> yandex: A disk image was created: ubuntu-2004-lts-nginx-2022-08-04t09-49-00z (id: fd8afpoirk5pqaain9mr) with family name ubuntu-web-server
```
8. Проверим, что получилось командой ```yc compute image list```
```ruby
mikhail@mikhail-GL703VD:~/Desktop/otus/02-Packer/otus-packer/json$ yc compute image list
+----------------------+--------------------------------------------+-------------------+----------------------+--------+
|          ID          |                    NAME                    |      FAMILY       |     PRODUCT IDS      | STATUS |
+----------------------+--------------------------------------------+-------------------+----------------------+--------+
| fd8afpoirk5pqaain9mr | ubuntu-2004-lts-nginx-2022-08-04t09-49-00z | ubuntu-web-server | f2emflrscp4rd664n8o7 | READY  |
+----------------------+--------------------------------------------+-------------------+----------------------+--------+
```
9. Удалим собранный образ
```ruby
mikhail@mikhail-GL703VD:~/Desktop/otus/02-Packer/otus-packer/json$ yc compute image delete --id fd8afpoirk5pqaain9mr
done (7s)
```
10. Проверим
```ruby
mikhail@mikhail-GL703VD:~/Desktop/otus/02-Packer/otus-packer/json$ yc compute image list
+----+------+--------+-------------+--------+
| ID | NAME | FAMILY | PRODUCT IDS | STATUS |
+----+------+--------+-------------+--------+
+----+------+--------+-------------+--------+

```
