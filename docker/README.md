## Install 
using new docker 23.0.1 on debian 10

`docker-compose` is no more, supposed to use `docker compose`, but its still rough

i converted to run vanilla docker
```bash
git clone https://github.com/k0a1a/hotglue2.git hotglue
cd hotglue/docker

docker volume create app

# Build and configure the hotglue container
docker build -t hotglue-image .
docker hotglue git clone https://github.com/k0a1a/hotglue2.git /app/
docker hotglue chmod -R 0777 /app/content
docker hotglue cp /app/user-config.inc.php-dist /app/user-config.inc.php

# create var PASSWORD=xxxxx in a .env
# Load the .env file
source .env
# Replace the password using the PASSWORD variable from the .env file
docker hotglue sed -i "s/changeme/${PASSWORD}/" /app/user-config.inc.php

#run container as hotglue
docker run -d --name hotglue -v app:/app hotglue-image
```

we running an nginx as service on docker host  
so route `hotglue.idiot.io` to the docker container ip address.  

get the running docker i using   
`docker netwrok inspect bridge`

```conf
server {
    server_name hotglue.idiot.io;

    location / {
        proxy_pass http://<DOCKER IP>:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    access_log /var/log/nginx/hotglue.access.log;
    error_log /var/log/nginx/hotglue.error.log;
}
```