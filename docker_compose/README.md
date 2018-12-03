# Docker Workshop (D10, iTechArt)

Our goals for part one:
- Understand and see how docker works
- Run docker via CLI
- Push your first docker image to DockerHub
- Try out docker-composer

### Block 1

List of commands you need:

```
# 1.1
docker version
```

Example of output:

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

Next command:

```
# 1.2
docker run alpine hostname
```

Example of output:

```
 Unable to find image 'alpine:latest' locally
 latest: Pulling from library/alpine
```

Next command:

```
# 1.3
docker ps

# 1.4
docker ps -a
```

Example of output:

```
CONTAINER ID  IMAGE   COMMAND     CREATED        STATUS                    PORTS  NAMES
d9b13d86dc03  alpine  "hostname"  4 seconds ago  Exited (0) 3 seconds ago         wizardly_roentgen
```

Next command:

```
# 1.5
docker run -it --rm ubuntu bash
```

You can play around inside your container

```
ls /
whoami
cat /etc/issue
# end even...
rm -rf --no-preserve-root /
# just kidding :)
```

Next commands: 

```
# 1.6
docker run -d --name mydb -e MYSQL_ROOT_PASSWORD=password1 mysql:latest

# 1.7
docker ps

# 1.8
docker logs mydb

# 1.9
docker exec -it mydb mysql --user=root --password=$MYSQL_ROOT_PASSWORD --version

# 1.10
docker exec -it mydb sh
```

Example of commands you can execute inside of container:
```
mysql --user=root --password=$MYSQL_ROOT_PASSWORD --version
exit
```

Next commands: 

```
# 1.11
git clone https://github.com/goodniceweb/docker_workshop
cd docker_workshop/first_block_app
vi Dockerfile

# 1.12
export DOCKERID=<yourIDhere>
echo $DOCKERID

# 1.13
docker image build --tag $DOCKERID/linux_tweet_app:1.0 .

# 1.14
docker run -d -p 8000:80 --name tweet_app_container $DOCKERID/linux_tweet_app:1.0

# 1.15
docker rm -f tweet_app_container
```

Next commands:

```
# 1.16
docker run -d -p 8000:80 \
  --name tweet_app_container \
  --mount type=bind,source="$(pwd)",target=/usr/share/nginx/html \
  $DOCKERID/linux_tweet_app:1.0

# 1.17
cp index-new.html index.html

# 1.18
docker rm -f tweet_app_container
docker run -d -p 8000:80 --name tweet_app_container $DOCKERID/linux_tweet_app:1.0

# 1.19
docker image build --tag $DOCKERID/linux_tweet_app:2.0 .

# 1.20
docker image ls -f reference="$DOCKERID/*"

# 1.21
docker run -d -p 8080:80 --name new_tweet_app_container $DOCKERID/linux_tweet_app:2.0
```

Next little section (only for those have account on DockerHub):

```
# 1.22
docker login
```

Example of output:

```
Username: <your docker id>
Password: <your docker id password>
Login Succeeded
```

Next command:

```
# 1.23
docker image push $DOCKERID/linux_tweet_app:1.0
```

Example of output:

```
86df2a1b653b: Mounted from library/nginx 
bc5b41ec0cfa: Mounted from library/nginx 
237472299760: Mounted from library/nginx
```

Next command:

```
# 1.24
docker image push $DOCKERID/linux_tweet_app:1.0
```

Example of output:

```
86df2a1b653b: Layer already exists 
bc5b41ec0cfa: Layer already exists 
237472299760: Layer already exists
```

Check out your profile there, for ex:

```
https://hub.docker.com/r/<your_profile_id>/linux_tweet_app/
# ex
# https://hub.docker.com/r/goodniceweb/linux_tweet_app/
```

### End of Block #1

### Block #2

Inside `second_block_app/step1`:

```
# 2.1
echo $DOCKERID
docker build --tag $DOCKERID/php-web-app:1.0 .
docker run -d -p 8080:80 --name php-web-app $DOCKERID/php-web-app:1.0
```

Inside `second_block_app/step2`:

```
# 2.2
docker build --tag $DOCKERID/ruby-api:1.0 api/

# 2.3
docker run -d -p 4567:4567 --name api $DOCKERID/ruby-api:1.0

# 2.4
docker build --tag $DOCKERID/php-web-app:2.0 www/

# 2.5
docker run -d -p 8081:80 --link api:api --name php-web-app-2 -e API_ENDPOINT=http://api:4567/api/ $DOCKERID/php-web-app:2.0
```

Inside `second_block_app/step3`:

```
# 2.6
docker-compose up --build
```
