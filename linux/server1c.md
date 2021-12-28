# Установка сервера 1С с web публикацией (Ubuntu 20.04 + 1С 8.3.20.1647 + Postgres Pro 14 + Apache)
## Установка web-сервера apache
Установка производится из репозитория ubuntu с помощью пакетного менеджера **apt**

```bash
sudo apt install apache2
```
По завершении установки процесс запущен со стандартными настройками и добавлен в автозагрузку

## Установка сервера 1С версии 8.3.20.1647
На сервере выполняем обновление данных о пакетах репозитория
```bash
sudo apt update
```
Устанавливаем необходимые для работы 1С компоненты (шрифты, библиотеки)
```bash
sudo apt install ttf-mscorefonts-installer libodbc1
```
Предварительно скаченный дистрибутив передаем на сервер в домашний каталог пользователя, под которым выполняем подключение через ssh (команда выполняется на машине, где находится дистрибутив)
```cmd
scp server64_8_3_20_1674.tar [username]@[servername]:/home/[homedir]
```
Распаковываем файлы архива в текущий каталог
```bash
tar -xzf server64_8_3_20_1674.tar.gz
```
Запускаем установщик 1С Предприятия. Указываем компоненты: кластер серверов, модули расширения веб-сервера
```bash
./setup-full-8.3.20.1674-x86_64.run --mode unattended --enable-components server,ws
```
Копируем скрипт запуска **srv1cv83** из каталога **/opt/1cv8/x86_64/8.3.20.1674/** в файл **/etc/init.d**
```bash
sudo cp /opt/1cv8/x86_64/8.3.20.1674/srv1cv83 /etc/init.d/
```
Копируем конфигурационный файл сервера 1с **srv1cv83.conf** из каталога **opt/1cv8/x86_64/8.3.20.1674/** в каталог **/etc/default/srv1cv83** (в файл без расширения .conf)
```bash
sudo cp /opt/1cv8/x86_64/8.3.20.1674/srv1cv83.conf /etc/default/srv1cv83
```
Добавляем в автозагрузку скопированный скрипт
```bash
sudo update-rc.d srv1cv83 defaults
```
Запускаем сервер 1С Предпрятия
```bash
sudo systemctl start srv1cv83
```
### Дополнительно
В текущем релизие имеется проблема: при установке сервера 1С устанавливаются элементы графической оболочки, которые в дальнейшем не нужны.
Удалим их.
```bash
sudo apt purge gnome-shell gnome-control-center gnome-keyring
sudo apt autoremove
```
## Установка СУБД PostgreSQL Pro v.14
Установка совершается из репозитория **Postgres Professional**. Предварительно репозиторий добавляется в список доступных репоизториев сервера с помощью скаченного скрипта
Скачиваем скрипт добавления репозитория для версии PostgresPro Standard 14 (https://repo.postgrespro.ru/pgpro-14/keys/apt-repo-add.sh)
```bash
curl -o apt-repo-add.sh https://repo.postgrespro.ru/pgpro-14/keys/apt-repo-add.sh
```
Запускаем скачанный скрипт с помощью оболочки sh
```bash
sudo sh apt-repo-add.sh
```
Устанавливаем **PostgresPro Standard 14** с помощью пакетного менеджера **apt**
```bash
sudo apt install postgrespro-std-14
```
### Первоначальная настройка Postgres Pro 14
Для возможности создания базы данных 1С Предприятия после установки Postgres Pro необходимо выполнить минимальные настройки, связанные с доступом.

Для возможности подключения к СУБД с других машин необходимо в файле `postgresql.conf` из каталога `/var/lib/pgpro/std-14/data/` прописать параметры подключения
>Здесь и далее примеры редактирования файлов приводятся с использованием текстового редактора **nano**
```bash
sudo nano /var/lib/pgpro/std-14/data/postgresql.conf
```
Находим строку с параметрами подключения
```bash
#listen_addresses = 'localhost'
```
Необходимо раскомментировать и заменить **localhost** на адрес машины, с которой осуществляется подключение (или \* для подключения с любого адреса)
![image](https://user-images.githubusercontent.com/36333345/147487973-5b51258d-1cff-4c15-9afc-4772f4a100cc.png)

В файле pg_hba.conf из каталога /var/lib/pgpro/std-14/data/ прописать разрешения подключения для пользователей
```bash
sudo nano /var/lib/pgpro/std-14/data/pg_hba.conf
```
В секции `# IPv6 local connections:` указать необходимый адрес, с которого разрешено подключение и пользователя СУБД. Пример разрешений для любого адреса и любого пользователя ниже
![image](https://user-images.githubusercontent.com/36333345/147547654-c637b310-2e87-42fd-88d9-34abb0cb53d0.png)

Установим пароль для автоматически созданного пользователя postgres в системе
```bash
sudo passwd postgres
```
Сменим текущего пользователя на postgres
```bash
su postgres
```
Запустим терминальный клиент для подключения к Postgres Pro

```bash
psql
```
Введем запрос на смену пароля для пользователя СУБД пользователя СУБД с именем postgres
```
ALTER USER postgres WITH PASSWORD [new_password]
```
### Публикация базы данных
Публикация выполняется с помощью утилиты **webinst**, поставляемой с платформой 1С Предприятия.
При запуске утилиты указываются параметры публикации:
+ `-publish` ключ, указывающий на необходимость создания публикации  
+ `-apache24` версия web-сервера  
+ `-wsdir` виртуальный каталог (в дальнейшем указывается в адресной строке после адреса сервера)
+ `-dir` физический каталог с расположением файлов публикации
+ `-connstr` строка соединения
+ `-confPath` полный путь к конфигурационному файлу Apache  
Выполним команду
```bash
sudo /opt/1cv8/x86_64/8.3.20.1674/webinst -publish -apache24 -wsdir [path_name] -dir /www -connstr "Srvr=localhost;Ref=[base_name];" -confpath /etc/apache2/apache2.conf
```
Перезапускаем web-сервер
```bash
sudo systemctl restart apache2
```
