# Docker

## Установка Docker на Ubuntu 20.04

### Подготовительные процедуры
Выполняем обновление данных о пакетах репозитория
````bash
sudo apt update
````
Устанавливаем все доступные обновления
````bash
sudo apt upgrade
````
Установка необходимых пакетов для работы с репозиторием по протоколу HTTPS
````bash
sudo apt-get install ca-certificates curl gnupg lsb-release
````

### Установка
Добавляем ключ GPG репозитория docker
````bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
````
Добавляем данные о репозитории в список используемых репозиториев системы
````bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
  https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list
````
Выполняем обновление данных о пакетах репозитория
````bash
sudo apt update
````
Устанавливаем пакты docker
````bash
sudo apt install docker-ce docker-ce-cli containerd.io
````
После окончания установки проверим наличие пакеты выводм версии docker
````bash
docker -v
````

### Постустановочные процедуры
Добавим пользователя, от именя которого будем работать в группу docker, чтобы исключить необходимость использования sudo
````bash
sudo usermod -aG docker
````
## Работа с контейнерами
Запуск контейнеров произвоидся с помощью команды `docker run`, которая имеет общий синтаксисЖ
````bash
docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
````
Пример: запуск контейнера из образа hello-world
````bash
docker run hello-world
````

Оснонве опции
+ `-it` - запуск контейна с подключенным стандартным вводом
+ `--name` - установка имени запущенного контейнера
+ `-d` - запуск в фоновом режиме
+ `-p` - сопостоавление портов внутри контейнера с портами хост системы ( `p [in]:[out]`)

Пример: запуск контейнера из образа docker/getting-started
````bash
docker run -d -p 80:80 --name myname docker/getting-started
````
