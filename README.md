# lab-08

Данная лабораторная работа посвящена созданию собственного локального сервера и работы с ним при помощи клиент-серверного приложения

# Task 1
Для начала сделаем сервер. Воспользуемся готовым решением в:Python

server.py содержание:
![image](https://github.com/lepeha81/lab08/blob/master/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA.PNG)

client.py содержание
![image](https://github.com/lepeha81/lab08/blob/master/2.PNG)

1. import http.server: импорт модуля http.server для запуска веб-сервера
2. import socketserver: импорт модуля socketserver для использования TCP-сервера
3. handler = http.server.SimpleHTTPRequestHandler: создаем новый обработчик запросов типа SimpleHTTPRequestHandler веб-запросов, который будет использоваться при каждом входящем запросе
4. with socketserver.TCPServer(("", 1234), handler) as httpd: - создаем новый TCP сервер, который будет прослушивать подключения на порту 1234 и передаем уже определенный обработчик веб-запросов
5. httpd.serve_forever(): запускаем сервер и прослушиваем его входящие запросы

Далее, в коде используется модуль urllib.request для отправки запроса на локальный веб-сервер и получения ответа.

6. fp = urllib.request.urlopen("http://localhost:1234/"): создаем объект fp, который открывает URL-адрес "http://localhost:1234/" для чтения. 
7. encodedContent = fp.read(): читаем содержимое URL-адреса и сохраняем его в переменной encodedContent
8. decodedContent = encodedContent.decode("utf8"): декодируем закодированное содержимое веб-страницы и сохраняем в переменной decodedContent
9. print(decodedContent): выводим прочитанное содержимое веб-страницы на экран
10. fp.close(): закрываем объект fp.

Далее мы сделаем для каждой части содержимого приложения для сервера:DockerfileDockerfile
`Dockerfile` Содержание для сервера:
![image](https://github.com/lepeha81/lab08/blob/master/4.PNG)

Dockerfile Содержание для клиента:
![image](https://github.com/lepeha81/lab08/blob/master/3.PNG)

1. FROM python:latest: задает базовый образ для текущего образа. В данном случае мы используем официальный образ Python из Docker Hub со всеми обновлениями.

2. ADD server.py /server/ и ADD index.html /server/: копирует файлы server.py и index.html из текущей директории хостовой машины в контейнер, в директорию /server/. 

3. WORKDIR /server/: устанавливает рабочую директорию в /server/.

4. FROM python:latest: задает новый базовый образ для текущего образа, но эта строка является ошибкой в Dockerfile. 

5. ADD client.py /client/: копирует файл client.py из хостовой машины в контейнер, в директорию /client/.

6. WORKDIR /client/: устанавливает рабочую директорию в /client/.

Также сделаем файл с содержимым вывода в каталоге htmlserver.py 

index.html содержание:

```
Hello 
```

Теперь мы сделаем для сборки нашего Dockerdocker-compose.yml

docker-compose.yml содержание:
![image](https://github.com/lepeha81/lab08/blob/master/5.PNG)
И, наконец, давайте создадим файл для нашего приложения для работы в GitHub Actions.yml

Action.yml содержание:
![image](https://github.com/lepeha81/lab08/blob/master/6.PNG)
version: "3" - указание версии Docker Compose.

services: - определение сервисов.

server: - имя сервиса.

build: server/ - указание директории, в которой находится Dockerfile для серверного сервиса.

command: python ./server.py - команда для запуска сервера.

ports: - определение портов, которые должны быть доступны извне.

1234:1234 - привязка порта на хосте к порту внутри контейнера.

client: - определение клиентского сервиса.

build: client/ - указание директории, в которой находится Dockerfile для клиентского сервиса.

command: python ./client.py - команда для запуска клиента.

network_mode: host - запуск контейнера в режиме хоста.

depends_on: - указание зависимостей сервисов.

- server - зависимость от сервера.

name: Docker Compose - название конфигурации.

on: - определение условий выполнения действия.

push: - выполнение действия на push в репозиторий.

branches: [ master ] - выполнение только для ветки master.

pull_request: - выполнение действия для pull request.

jobs: - определение работы.

build-and-deploy: - определение задачи.

runs-on: ubuntu-latest - выбор контейнера для выполнения задачи.

steps: - массив шагов, которые должны быть выполнены внутри работы.

- name: Checkout code - название шага.

uses: actions/checkout@v2 - клонирование репозитория.

- name: Login to DockerHub - название шага.

uses: docker/login-action@v1 - авторизация в Docker Hub для публикации образов.

with: - передача параметров в действие.

username: ${{ secrets.DOCKER_USERNAME }} - имя пользователя Docker Hub.

password: ${{ secrets.DOCKER_PASSWORD }} - пароль пользователя Docker Hub.

- name: Build Docker images - название шага.

run: docker-compose build - сборка образов Docker.

- name: Push Docker images - название шага.

run: docker-compose push - пуш образов Docker в Docker Hub.

- name: Deploy with Docker Compose - название шага.

run: docker-compose up -d - запуск контейнеров с помощью Docker Compose 
![image](https://github.com/lepeha81/lab08/blob/master/7.PNG)
Команда Docker Compose up развёртывает сервисы веб-приложений и создаёт из docker-образа новые контейнеры, а также сети, тома и все конфигурации, указанные в файле Docker Compose.
