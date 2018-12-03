# Docker Workshop - part 3: Swarm Mode

Was tested on [https://labs.play-with-docker.com/](https://labs.play-with-docker.com/)

```
# 3.1
git clone https://github.com/goodniceweb/congregation_phoenix
cd congregation_phoenix
git checkout -b docker-workshop origin/docker-workshop

# 3.2
docker-compose up --build

# 3.3
# some mystery command...

# 3.4
docker swarm init

# 3.5
docker swarm init --advertise-addr 192.168.0.xx
# replace xx with your IP ending

# 3.6
docker swarm join-token manager
# copy command from the output of previous command
# create two another instances and execute just copied command there

# 3.7
docker swarm join-token worker
# copy command from the output of previous command
# create another instance and execute just copied command there

# 3.7.1 ensure you have 3 managers and 1 worker

# 3.8
docker stack deploy --compose-file docker-stack.yml congregation

# 3.9
docker stack ls

# 3.10
docker stack ps congregation

# 3.11
ab -n 1000 -c 50 YOUR_URL
# replace YOUR_URL with browser url from port 4000 link

# 3.12 (same as 3.7)
docker swarm join-token worker
# copy command from the output of previous command
# create another instance and execute just copied command there

# 3.13
docker service scale congregation_web=5

# 3.14 (same as 3.11)
ab -n 1000 -c 50 YOUR_URL
# For me it was
# Requests per second:    268.75 [#/sec] (2 workers)
# Requests per second:    150.56 [#/sec] (4 workers)

# 3.13
docker service scale congregation_web=4

# 3.14
docker service update --image goodniceweb/congregation_phoenix_web:0.0.4 congregation_web

# 3.15
# lets kill one of worker nodes
```


