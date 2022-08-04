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

