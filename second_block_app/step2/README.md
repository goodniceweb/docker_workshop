# On this step

Please use following commands:
```
docker build --tag $DOCKERID/ruby-api:1.0 api/
docker run -d -p 4567:4567 --name api $DOCKERID/ruby-api:1.0

docker build --tag $DOCKERID/php-web-app:2.0 www/
docker run -d -p 8080:80 --link api:api --name php-web-app-2 -e API_ENDPOINT=http://api:4567/api/ $DOCKERID/php-web-app:2.0
```

After all demo
```
docker rm -f api
docker rm -f php-web-app
```

Cleanup if needed (you might mess up something):
```
docker rmi $DOCKERID/ruby-api:1.0
docker rmi $DOCKERID/php-web-app:2.0
```
