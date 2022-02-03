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
### Запуск контейнеров
Запуск контейнеров производится с помощью команды `docker run`, которая имеет общий синтаксис:
````bash
docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
````
Пример: запуск контейнера из образа hello-world
````bash
docker run hello-world
````

Основные опции
+ `-it` - запуск контейна с подключенным стандартным вводом
+ `--name` - установка имени запущенного контейнера
+ `-d` - запуск в фоновом режиме
+ `--rm` - удалить контейнер после завершения работы
+ `-p` - сопоставление портов внутри контейнера с портами хост системы ( `p [in]:[out]`)

Пример: запуск контейнера из образа docker/getting-started
````bash
docker run -d -p 80:80 --name myname docker/getting-started
````
### Просмотр списка контейнеров
Список существующих контейнеров (запущенных и остановленных) осуществляется с помощью команды `docker ps`
````bash
docker ps -a
````
Основные опции
+ `-a` - просмотр всех контейнеров
+ `-q` - вывод только идентификаторов

### Запуск и остановка контейнеров
Запуск существующего контейнера
````bash
docker start [name | id]
````
Остановка существующего контейнера
````bash
docker stop [name | id]
````
Перзапуск существующего контейнера
````bash
docker restart [name | id]
````
### Удаление контейнеров
Для удаления контейнеров используется команда `ps`.
Опция `-f` используется для удаления запущенных контейнеров
````bash
docker rm [name | id]
````
Удаление всех контейнеров
````bash
docker rm $(docker ps -aq)
````
## Работа с образами

Просмотр имеющихся в системе образов
````bash
docker images
````
### Dockerfile
Пример сбокри образа на базе образа alpine, выполняющего созданный скрипт

Создаем скрипт
````bash
echo "echo Starting backup..." > script.sh
````

Открываем текстовый редактор nano для создания Файла Dockerfile
````bash
nano Dockerfile 
````
Содержимое файла:
````Dockerfile
# Указание базового образа
FROM alpine

# Создание рабочего каталога
RUN mkdir /myapp

# Копирование скрипта
COPY ./script.sh /myapp/script.sh

# Указание старотового рабочего каталога
WORKDIR /myapp

# Команда, выполняемая при старте контейнера
ENTRYPOINT sh script.sh
````
Выполняем сборку образа с именем my-script
````bash
docker build -t my-script .
````
## Работа с томами

Пример монтирования с использованием образа httpd
````bash
docker run -dit --name my-apache-app -p 8080:80 -v "$PWD":/usr/local/apache2/htdocs/ httpd:2.4
````
