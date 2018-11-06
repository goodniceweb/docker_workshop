# On this step

Ensure DOCKERID env variable is set
```
echo $DOCKERID
```

Then use following commands:
```
docker build --tag $DOCKERID/php-web-app:1.0 .
docker run -d -p 8080:80 --name php-web-app $DOCKERID/php-web-app:1.0
```

