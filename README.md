# Docker Workshop

Для начала давайте дадим некоторые определения:
Docker - 
Container - 
Image - 
Service - 
DockerHub - 

Кто завёл аккаунт на Docker Hub - тот красавчик.

Меньше слов, больше дела. Не зря же вы ноуты тащили.
У нас будет N блоков. Каждый будет состоять чуть больше
чем полностью из примеров, которые вы будете пробовать
выполнять на своих локальных машинах.

Блок 1: основы основ

Для начала проверим установку:

```
# 1.1
docker version
```

Должно вывести что-то вроде следующего:

```
Client:
 Version:      17.05.0-ce
 API version:  1.29
 Go version:   go1.7.5
 Git commit:   89658be
 Built:        Thu May  4 22:10:54 2017
 OS/Arch:      linux/amd64

Server:
 Version:      17.05.0-ce
 API version:  1.29 (minimum version 1.12)
 Go version:   go1.7.5
 Git commit:   89658be
 Built:        Thu May  4 22:10:54 2017
 OS/Arch:      linux/amd64
 Experimental: false
```

Если с этой коммандой возникли проблемы, дайте знать сейчас или
молчите вечно :)

Теперь запустим контейнер на основе образа alpine. Сделать это можно
при помощи комманды `docker run`. Она имеет следующий синтаксис:

```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

Испробуем её в действии:

```
# 1.2
docker run alpine hostname
```

Скорее всего эта команда выдаст output что-то вроде следующего:

```
 Unable to find image 'alpine:latest' locally
 latest: Pulling from library/alpine
```

Нам говорят, что локально не было найдено образа alpine с тегом latest.
Поэтому docker послал запрос в DockerHub, где успешно нашел нужный нам образ.

Что ж, давайте посмотрим как отработал запущенный нами контейнер. Для этого
просмотрим все запущенные на текущий момент контейнеры. Сделать это можно
при помощи команды ps

```
# 1.3
docker ps
```

Как видим, она ничего не возвращает. Это связано с тем, что Docker вырубает
контейнер как только тот закончил выполнение своего задания.

Чтобы посмотреть все имеющиеся контейнеры, можно передать комманде ps флаг --all

```
# 1.4
docker ps -a
```

Теперь видно что у нас действительно был запущен контейнер на основании образа alpine

CONTAINER ID  IMAGE   COMMAND     CREATED        STATUS                    PORTS  NAMES
d9b13d86dc03  alpine  "hostname"  4 seconds ago  Exited (0) 3 seconds ago         wizardly_roentgen

объяснить про поля: container ID, image, command, created, status, ports, names

А теперь давайте попробуем покопаться внутри контейнера и посмотреть как он там живет.

```
# 1.5
docker run -it --rm ubuntu bash
```

-i -- short form of --interactive, says you want interactive session
-t -- short form of --tty, allocates pseudo tty
--rm -- indicate you want to remove container after it's done executing

you can play around:
```
ls /
whoami
cat /etc/issue
rm -rf --no-preserve-root /
```

Если попытаетесь теперь найти только что завершивший свою работу контейнер в
`docker ps -a`, то это не получится. Всё потому что была использована опция
`--rm` которая удаляет контейнер как только тот закончил своё выполнение.
В наше случае это и была комманда `bash`, поэтому завершив её, он просто
удалился.

Когда использовать интерактивный режим? Например, вы хотите задеплоить из докера.
Нужно понимать какие степы необходимы конкретно под ваш случай. Можно подключиться
к чистому образу, сделать деплой один раз в ручную, посмотреть какие комманды
нужно было использовать и перенести их в Dockerfile. Конечно, можно и контейнер
закоммитать, но это не лучшая практика, так как тогда невозжможно будет
изменить степы подготовки окружение. Детальнее об этом чуть позже.

Всё это конечно здорово, но как же запустить контейнер в фоне, чтобы, например,
базу в него поставить? Очень легко:

```
# 1.6
docker run -d --name mydb -e MYSQL_ROOT_PASSWORD=password1 mysql:latest
```

-d -- short form of --detach. Run container in background
--name -- set alias for your container
-e -- use environment variable to specify some settings for container.
      root mysql user password in our case. You can find full list here:
      https://hub.docker.com/_/mysql/

First of all we need to be sure our container is still running:
```
# 1.7
docker logs mydb
```

Lets see what is going on there:
```
# 1.8
docker logs mydb
```

Если попытаемся постучаться сейчас в дефолтный MySQL порт(какой он кстати),
как думаете, что произойдет? Почему?

Давайте узнаем точнее версию mysql, которая у нас скачалась. Для этого
можно воспользоваться коммандой exec. Эта комманда позволяет подключаться
к уже запущенному контейнеру и выполнять в нем то, что нам нужно.
```
# 1.9
docker exec -it mydb mysql --user=root --password=$MYSQL_ROOT_PASSWORD --version
```

Если надо выполнить сразу несколько комманд, чтобы не писать их все
через `docker exec`, можно подключиться к контейнеру интерактивно:
```
# 1.10
docker exec -it mydb sh
```

Теперь внутри мы можем выполнить всё что было необходимо
```
mysql --user=root --password=$MYSQL_ROOT_PASSWORD --version
exit
```

Итак, базу в фоне запустить получилось. Как на счет web сервачка?

```
# 1.11
git clone https://github.com/dockersamples/linux_tweet_app
cd linux_tweet_app
vi Dockerfile
```

Давайте осмотримся чуть-чуть. Что мы тут видим:
FROM   -- specifies the base image to use as the starting point
          for this new image you’re creating. For this example
          we're starting from nginx:latest.
COPY   -- copies files from the Docker host into the image,
          at a known location. In this example, COPY is used
          to copy two files into the image: index.html
          and a graphic that will be used on our webpage.
EXPOSE -- documents which ports the application uses.
CMD    -- specifies what command to run when a container
          is started from the image. Notice that we can specify
          the command, as well as run-time arguments.

Сейчас можно воспользоваться своим DockerID, если делали
аккаунт себе на DockerHub. 
```
# 1.12
export DOCKERID=<yourIDhere>
echo $DOCKERID
```

У кого нету аккаунта, можете туда свой github nickname прописать пока.
А вот и комманда для создания вашего первого образа:
```
# 1.13
docker image build --tag $DOCKERID/linux_tweet_app:1.0 .
```

В output видно как докер выполняет каждую строку из Dockerfile

Теперь можем посмотреть на результат и создать контейнер на основании
того образа, который только что сбилдался.
```
# 1.14
docker run -d -p 8000:80 --name tweet_app_container $DOCKERID/linux_tweet_app:1.0
```

Можете теперь открыть в браузере http://localhost:8000/
И убедиться что всё работает :)

Убедились? А теперь время всё это уничтожить:
```
# 1.15
docker rm -f tweet_app_container
```

We used the --force parameter to remove the running container
without shutting it down. This will ungracefully shutdown
the container and permanently remove it from the Docker host.

In a production environment you may want to use `docker stop`
to gracefully stop the container and leave it on the host.
You can then use `docker rm` to permanently remove it.

Зачем мы только что его уничтожили? Чтобы сделать еще лучше!
Во время разработки мы ожидаем, что файлы сами будут подтягиваться
и не нужно будет каждый раз пересобирать образ. Достичь этого
результата можно с помощью `bind mount`
```
# 1.16
docker run -d -p 8000:80 \
  --name tweet_app_container \
  --mount type=bind,source="$(pwd)",target=/usr/share/nginx/html \
  $DOCKERID/linux_tweet_app:1.0
```

Теперь попробуем заменить файл 
```
# 1.17
cp index-new.html index.html
```

Как видно, за счет использования `bind mount` изменения сразу же подтянулись.

Even though we’ve modified the index.html local filesystem and seen it
reflected in the running container, we’ve not actually changed
the Docker image that the container was started from.

To show this, stop the current container and re-run the 1.0 image
without a bind mount.

```
# 1.18
docker rm -f tweet_app_container
docker run -d -p 8000:80 --name tweet_app_container $DOCKERID/linux_tweet_app:1.0
```

Теперь фон снова голубенький. Это произошло потому, что только что
запущенный контейнер основывается не на текущем состоянии папки,
а на том состоянии, когда был сбилджен образ.

Похоже, пришло время обновить его, этот image. Прежде всего убедитесь,
что у вас есть изменения (например фон стал ораньжевеньким)

```
# 1.19
docker image build --tag $DOCKERID/linux_tweet_app:2.0 .
```

Заметили, что билд прошёл быстрее, чем в первый раз? Это потому, что
Docker перебилдал только те шаги, которые поменялись, а именно -
сейчас скопировал другой файл. В этом и есть прелесть слоёв, на которых
строятся образы - пересобрать что-то не так сложно, если разбиение
на инструкции дробное в Dockerfile.

Убедимся что новый образ создался:

```
# 1.20
docker image ls -f reference="$DOCKERID/*"
```

Отлично! Оттестируем его:

```
# 1.21
docker run -d -p 8080:80 --name new_tweet_app_container $DOCKERID/linux_tweet_app:2.0
```

Теперь можем сравнить старую и новую версии.

Старая: http://localhost:8000/
Новая: http://localhost:8080/

Следующая секция только для тех, у кого есть аккаунт на Docker Hub

```
# 1.22
docker login
```

You will need to supply your Docker ID credentials when prompted.

```
Username: <your docker id>
Password: <your docker id password>
Login Succeeded
```

Теперь можно пушнуть в DockerHub ваши творения:

```
# 1.23
docker image push $DOCKERID/linux_tweet_app:1.0
```

Как видим, не все слои пушаться туда. Некоторые он подхватывает из
других образов.

```
86df2a1b653b: Mounted from library/nginx 
bc5b41ec0cfa: Mounted from library/nginx 
237472299760: Mounted from library/nginx
```

```
# 1.23
docker image push $DOCKERID/linux_tweet_app:1.0
```

То же касается и ваших личных образов. В итоге получаем максимальное
экономию рабочего пространства и гибкую, быструю систему сборки.

```
86df2a1b653b: Layer already exists 
bc5b41ec0cfa: Layer already exists 
237472299760: Layer already exists
```

Check out your profile there, for ex:
```
https://hub.docker.com/r/goodniceweb/linux_tweet_app/
```



 












