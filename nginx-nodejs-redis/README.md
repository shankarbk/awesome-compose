## Compose sample application

## Node.js application with Nginx proxy and Redis database

Project structure:
```
.
├── README.md
├── compose.yaml
├── nginx
│   ├── Dockerfile
│   └── nginx.conf
└── web
    ├── Dockerfile
    ├── package.json
    └── server.js

2 directories, 7 files


```
[_compose.yaml_](compose.yaml)
```
redis:
    image: 'redislabs/redismod'
    ports:
      - '6379:6379'
  web1:
    restart: on-failure
    build: ./web
    hostname: web1
    ports:
      - '81:5000'
  web2:
    restart: on-failure
    build: ./web
    hostname: web2
    ports:
      - '82:5000'
  nginx:
    build: ./nginx
    ports:
    - '80:80'
    depends_on:
    - web1
    - web2
```
The compose file defines an application with four services `redis`, `nginx`, `web1` and `web2`.
When deploying the application, docker compose maps port 80 of the nginx service container to port 80 of the host as specified in the file.


> ℹ️ **_INFO_**  
> Redis runs on port 6379 by default. Make sure port 6379 on the host is not being used by another container, otherwise the port should be changed.

## Deploy with docker compose

```
$ docker compose up -d
[+] Running 24/24
 ⠿ redis Pulled                                                                                                                                                                                                                      ...
   ⠿ 565225d89260 Pull complete                                                                                                                                                                                                      
[+] Building 2.4s (22/25)
 => [nginx-nodejs-redis_nginx internal] load build definition from Dockerfile                                                                                                                                                         ...
[+] Running 5/5
 ⠿ Network nginx-nodejs-redis_default    Created                                                                                                                                                                                      
 ⠿ Container nginx-nodejs-redis-web2-1   Started                                                                                                                                                                                      
 ⠿ Container nginx-nodejs-redis-redis-1  Started                                                                                                                                                                                      
 ⠿ Container nginx-nodejs-redis-web1-1   Started                                                                                                                                                                                      
 ⠿ Container nginx-nodejs-redis-nginx-1  Started
```


## Expected result

Listing containers must show three containers running and the port mapping as below:


```
docker-compose ps
```

## Testing the app

After the application starts, navigate to `http://localhost:80` in your web browser or run:

```
curl localhost:80
curl localhost:80
web1: Total number of visits is: 1
```

```
curl localhost:80
web1: Total number of visits is: 2
```
```
$ curl localhost:80
web2: Total number of visits is: 3
```



## Stop and remove the containers

```
$ docker compose down
```

## Request Flow
Complete Request Flow (Numbered Steps)
1. Step 1 (External Request): An external client sends an HTTP request to the host machine on Port 80.
2. Step 2 (Nginx Ingress): The host forwards port 80 directly into the nginx reverse proxy container.
3. Step 3 (Service Discovery & Proxying): Nginx reads its configuration and routes the request internally across the Docker network bridge:
    - Internal Proxy: Nginx routes to http://web1:5000 or http://web2:5000 using Docker's internal DNS.
    - External Host Port Access: Alternatively, if Nginx accesses services via host ports, it routes to http://host:81 for web1 or http://host:82 for web2.
4. Step 4 (Container Namespace Execution): The request arrives inside web1 or web2's isolated network namespace, where the application process is listening internally on Port 5000.

## Clarifying the Roles
Role of nginx: Nginx acts as the single entry point (reverse proxy / load balancer). Clients talk only to Nginx on port 80; Nginx then dispatches requests to backend containers (web1 and web2).

Role of Host Ports (81:5000, 82:5000): Exposing host ports 81 and 82 lets you optionally bypass Nginx or test web1 and web2 directly from your host machine browser.

Role of hostname (web1-hn, web2-hn): Defines the explicit network identity/DNS name inside Docker's bridge network. Other containers (like Nginx) can reference these hostnames directly over the internal network on port 5000 without needing host port mapping.

<img width="1024" height="914" alt="image" src="https://github.com/user-attachments/assets/0c26a162-5297-4d25-8028-ba126422e645" />
<img width="1024" height="914" alt="image" src="https://github.com/user-attachments/assets/a3a02fbf-f936-4d17-ba77-39785c0cd955" />
