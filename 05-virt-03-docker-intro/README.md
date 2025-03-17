
# Домашнее задание к занятию 4 «Оркестрация группой Docker контейнеров на примере Docker Compose». Наталия Ханова.

### Инструкция к выполению

1. Для выполнения заданий обязательно ознакомьтесь с [инструкцией](https://github.com/netology-code/devops-materials/blob/master/cloudwork.MD) по экономии облачных ресурсов. Это нужно, чтобы не расходовать средства, полученные в результате использования промокода.
2. Практические задачи выполняйте на личной рабочей станции или созданной вами ранее ВМ в облаке.
3. Своё решение к задачам оформите в вашем GitHub репозитории в формате markdown!!!
4. В личном кабинете отправьте на проверку ссылку на .md-файл в вашем репозитории.

## Задача 1

Сценарий выполнения задачи:
- Установите docker и docker compose plugin на свою linux рабочую станцию или ВМ.
- Если dockerhub недоступен создайте файл /etc/docker/daemon.json с содержимым: ```{"registry-mirrors": ["https://mirror.gcr.io", "https://daocloud.io", "https://c.163.com/", "https://registry.docker-cn.com"]}```
- Зарегистрируйтесь и создайте публичный репозиторий  с именем "custom-nginx" на https://hub.docker.com (ТОЛЬКО ЕСЛИ У ВАС ЕСТЬ ДОСТУП);
- скачайте образ nginx:1.21.1;
- Создайте Dockerfile и реализуйте в нем замену дефолтной индекс-страницы(/usr/share/nginx/html/index.html), на файл index.html с содержимым:
```
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I will be DevOps Engineer!</h1>
</body>
</html>
```
- Соберите и отправьте созданный образ в свой dockerhub-репозитории c tag 1.0.0 (ТОЛЬКО ЕСЛИ ЕСТЬ ДОСТУП). 
- Предоставьте ответ в виде ссылки на https://hub.docker.com/<username_repo>/custom-nginx/general .

https://hub.docker.com/repository/docker/notburga/custom-nginx/general

## Задача 2
1. Запустите ваш образ custom-nginx:1.0.0 командой docker run в соответвии с требованиями:
- имя контейнера "ФИО-custom-nginx-t2"
- контейнер работает в фоне
- контейнер опубликован на порту хост системы 127.0.0.1:8080
```
docker run -d --name khanovanv-custom-nginx-t2 -p 127.0.0.1:8080:80 custom-nginx:1.0.0
```
2. Не удаляя, переименуйте контейнер в "custom-nginx-t2"
```
docker rename khanovanv-custom-nginx-t2 custom-nginx-t2
```
3. Выполните команду ```date +"%d-%m-%Y %T.%N %Z" ; sleep 0.150 ; docker ps ; ss -tlpn | grep 127.0.0.1:8080  ; docker logs custom-nginx-t2 -n1 ; docker exec -it custom-nginx-t2 base64 /usr/share/nginx/html/index.html```
4. Убедитесь с помощью curl или веб браузера, что индекс-страница доступна.
```
curl -i 127.0.0.1:8080
```

В качестве ответа приложите скриншоты консоли, где видно все введенные команды и их вывод.

![console](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/virtd-Docker1-t2-1.png)

![browser](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/virtd-Docker1-t2-2.png)

## Задача 3
1. Воспользуйтесь docker help или google, чтобы узнать как подключиться к стандартному потоку ввода/вывода/ошибок контейнера "custom-nginx-t2".
2. Подключитесь к контейнеру и нажмите комбинацию Ctrl-C.
```
docker attach custom-nginx-t2
```
3. Выполните ```docker ps -a``` и объясните своими словами почему контейнер остановился.

![attach](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/docker-attach.png)

![stopped](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/docker-ps.png)

При нажатии Ctrl-C контейнер получает сигнал SIGINT, завершающий основной процесс контейнера (nginx), поэтому контейнер прекратил свою работу. 

4. Перезапустите контейнер
Для успешного перезапуска контейнера надо предварительно освободить порт. 
```
lsof -i :8080
kill 166156
docker start custom-nginx-t2
```

![start](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/docker-start_cleanports.png)

5. Зайдите в интерактивный терминал контейнера "custom-nginx-t2" с оболочкой bash.
```
docker exec -it custom-nginx-t2 bash
```

6. Установите любимый текстовый редактор(vim, nano итд) с помощью apt-get.
```
apt-get update && apt-get install -y nano
```

![nano](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/docker-install-nano.png)

7. Отредактируйте файл "/etc/nginx/conf.d/default.conf", заменив порт "listen 80" на "listen 81".
```
nano /etc/nginx/conf.d/default.conf
```

![changeport1](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/nginx-changeport.png)

8. Запомните(!) и выполните команду ```nginx -s reload```, а затем внутри контейнера ```curl http://127.0.0.1:80 ; curl http://127.0.0.1:81```.

![curls](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/nginx-reload-curls.png)

9. Выйдите из контейнера, набрав в консоли  ```exit``` или Ctrl-D.
10. Проверьте вывод команд: ```ss -tlpn | grep 127.0.0.1:8080``` , ```docker port custom-nginx-t2```, ```curl http://127.0.0.1:8080```. Кратко объясните суть возникшей проблемы.

![wrongPort](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/nginx-wrong-port.png)

Проблема связана с тем, что мы изменили слушающий порт внутри контейнера, в то время как на хосте маппинг остался прежним. Таким образом, хост пытается направить трафик на порт 80, в то время как nginx в контейнере ожидает трафика с порта 81. 

11. * Это дополнительное, необязательное задание. Попробуйте самостоятельно исправить конфигурацию контейнера, используя доступные источники в интернете. Не изменяйте конфигурацию nginx и не удаляйте контейнер. Останавливать контейнер можно. [пример источника](https://www.baeldung.com/linux/assign-port-docker-container)

Находим файлы с конфигурацией портов внутри нашего контейнера:

![configs](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/docker-config-ports.png)

Исправляем конфигурацию в файлах:
* hostconfig.json

![hostconfig](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/hostconfig.png)

* config.v2.json

![config.v2](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/configV2.png)

Вот как изменится результат вывода команд из пункта 10 после правок конфигурации портов внутри контейнера:

![redirect](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/docker-redirected.png)

12. Удалите запущенный контейнер "custom-nginx-t2", не останавливая его.(воспользуйтесь --help или google)

```docker rm -f custom-nginx-t2```

![removeContainer](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/docker-rm-cont.png)

В качестве ответа приложите скриншоты консоли, где видно все введенные команды и их вывод.

## Задача 4


- Запустите первый контейнер из образа ***centos*** c любым тегом в фоновом режиме, подключив папку  текущий рабочий каталог ```$(pwd)``` на хостовой машине в ```/data``` контейнера, используя ключ -v.

```docker run -d -v $(pwd):/data --name centos-container -it centos:latest```

- Запустите второй контейнер из образа ***debian*** в фоновом режиме, подключив текущий рабочий каталог ```$(pwd)``` в ```/data``` контейнера. 

```docker run -d -v $(pwd):/data --name debian-container -it debian:latest```

![centos+debian](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/virtd-Docker1-t4-12.png)

- Подключитесь к первому контейнеру с помощью ```docker exec``` и создайте текстовый файл любого содержания в ```/data```.
- Добавьте ещё один файл в текущий каталог ```$(pwd)``` на хостовой машине.

![files](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/virtd-Docker1-t4-34.png)

- Подключитесь во второй контейнер и отобразите листинг и содержание файлов в ```/data``` контейнера.

![listing](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/virtd-Docker1-t4-5.png)

В качестве ответа приложите скриншоты консоли, где видно все введенные команды и их вывод.


## Задача 5

1. Создайте отдельную директорию(например /tmp/netology/docker/task5) и 2 файла внутри него.
"compose.yaml" с содержимым:
```
version: "3"
services:
  portainer:
    network_mode: host
    image: portainer/portainer-ce:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```
"docker-compose.yaml" с содержимым:
```
version: "3"
services:
  registry:
    image: registry:2

    ports:
    - "5000:5000"
```

И выполните команду "docker compose up -d". Какой из файлов был запущен и почему? (подсказка: https://docs.docker.com/compose/compose-application-model/#the-compose-file )

![compose](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/docker-run-compose.png)

В соответствии с правилами Docker для запуска осуществлятся поиск файлов compose.yaml и docker-compose.yaml, но в случае обнаружения двух файлов в директории приоритет отдаётся файлу с именем compose.yaml. 

2. Отредактируйте файл compose.yaml так, чтобы были запущены оба файла. (подсказка: https://docs.docker.com/compose/compose-file/14-include/)

В compose.yaml добавляем настройку:
```
include:
  - docker-compose.yaml
```

![include](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/docker-compose-include.png)

![compose-ed](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/docker-run-compose+docker.png)

3. Выполните в консоли вашей хостовой ОС необходимые команды чтобы залить образ custom-nginx как custom-nginx:latest в запущенное вами, локальное registry. Дополнительная документация: https://distribution.github.io/distribution/about/deploying/

```
docker pull notburga/custom-nginx:1.0.0
docker tag notburga/custom-nginx:1.0.0 127.0.0.1:5000/custom-nginx:latest
docker push 127.0.0.1:5000/custom-nginx:latest
```

![latest](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/docker-run-tag-push.png)

4. Откройте страницу "https://127.0.0.1:9000" и произведите начальную настройку portainer.(логин и пароль администратора)

![portainer](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/portainer-create-user.png)

5. Откройте страницу "http://127.0.0.1:9000/#!/home", выберите ваше local  окружение. Перейдите на вкладку "stacks" и в "web editor" задеплойте следующий компоуз:

```
version: '3'

services:
  nginx:
    image: 127.0.0.1:5000/custom-nginx
    ports:
      - "9090:80"
```

![stack](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/portainer-create-stack.png)

![stack1](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/portainer-stack-created.png)

6. Перейдите на страницу "http://127.0.0.1:9000/#!/2/docker/containers", выберите контейнер с nginx и нажмите на кнопку "inspect". В представлении <> Tree разверните поле "Config" и сделайте скриншот от поля "AppArmorProfile" до "Driver".

![tree1](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/portainer-tree1.png)

![tree2](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/portainer-tree2.png)

7. Удалите любой из манифестов компоуза (например compose.yaml).  Выполните команду "docker compose up -d". Прочитайте warning, объясните суть предупреждения и выполните предложенное действие. Погасите compose-проект ОДНОЙ(обязательно!!) командой.

![orphans](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/docker-orphans.png)

В предупреждении сообщается, что при запуске проекта обнаружены контейнеры, не указанные в оставшемся манифесте. Их предлагается удалить при новом запуске посредством команды ```docker compose up -d --remove-orphans```. 

Запуск с удалением непривязанных контейнеров:

![remove-orphans](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/docker-remove-orphans.png)

Гасим проект:
```docker compose down```

![docker-compose-down](https://github.com/NataliyaKh/virtd-homeworks/blob/main/05-virt-03-docker-intro/docker-compose-down.png)

В качестве ответа приложите скриншоты консоли, где видно все введенные команды и их вывод, файл compose.yaml , скриншот portainer c задеплоенным компоузом.

---

### Правила приема

Домашнее задание выполните в файле readme.md в GitHub-репозитории. В личном кабинете отправьте на проверку ссылку на .md-файл в вашем репозитории.


