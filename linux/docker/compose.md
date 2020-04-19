### Docker Compose
https://docs.docker.com/compose/
> 简单理解为 "批处理"，是一个工具，通过一个yaml文件定义多容器的docker应用，方便本地开发而非生产环境

场景：多容器的APP部署繁琐复杂
* 要从Dockerfile build image 或者 Dockerhub拉取image
* 要创建多个comtainer
* 要管理这些container(启动停止删除)

#### docker-compose.yaml(可自定义名字)
三大概念
Services
一个service代表一个container，这个container可以从dockerhub的image来创建，或者从本地的Dockerfile build出来的image来创建
Service的启动类似docker run，我们可以给其指定network和volume，所以可以给service指定network和Volume的引用
e.g. 
如下 yaml 和 `docker run -d --network back-tier -v db-data:/var/lib/postgresql/data postgres:9.4`等价
```
services:
  db:
    image: postgres:9.4
    volumes:
      - "db-data:/var/lib/postgresql/data"
    networks:
      - back-tier
```

如下 yaml 和 `docker run -d --network back-tier -link db --link redis  $(docker build ./worker)`等价，./worker 的位置就是Dockerfile的位置（其实对于自建bridge的情况下，如这里的back-tier是不需要link进行DNS翻译的）
```
services:
  worker:
    build: ./worker
    links:
      - db
      - redis
    networks:
      - back-tier
```
Networks
Volumes

### API
```
Usage:
  docker-compose [-f <arg>...] [options] [COMMAND] [ARGS...]
  docker-compose -h|--help

Options:
  -f, --file FILE             Specify an alternate compose file
                              (default: docker-compose.yml)
  -p, --project-name NAME     Specify an alternate project name
                              (default: directory name)
  --verbose                   Show more output
  --log-level LEVEL           Set log level (DEBUG, INFO, WARNING, ERROR, CRITICAL)
  --no-ansi                   Do not print ANSI control characters
  -v, --version               Print version and exit
  -H, --host HOST             Daemon socket to connect to

  --tls                       Use TLS; implied by --tlsverify
  --tlscacert CA_PATH         Trust certs signed only by this CA
  --tlscert CLIENT_CERT_PATH  Path to TLS certificate file
  --tlskey TLS_KEY_PATH       Path to TLS key file
  --tlsverify                 Use TLS and verify the remote
  --skip-hostname-check       Don't check the daemon's hostname against the
                              name specified in the client certificate
  --project-directory PATH    Specify an alternate working directory
                              (default: the path of the Compose file)
  --compatibility             If set, Compose will attempt to convert keys
                              in v3 files to their non-Swarm equivalent
  --env-file PATH             Specify an alternate environment file

Commands:
  build              Build or rebuild services
  config             Validate and view the Compose file
  create             Create services
  down               Stop and remove containers, networks, images, and volumes
  events             Receive real time events from containers
  exec               Execute a command in a running container
  help               Get help on a command
  images             List images
  kill               Kill containers
  logs               View output from containers
  pause              Pause services
  port               Print the public port for a port binding
  ps                 List containers
  pull               Pull service images
  push               Push service images
  restart            Restart services
  rm                 Remove stopped containers
  run                Run a one-off command
  scale              Set number of containers for a service
  start              Start services
  stop               Stop services
  top                Display the running processes
  unpause            Unpause services
  up                 Create and start containers
  version            Show the Docker-Compose version information
  ```

#### scale 参数
> 扩展相同的container实现水平扩展

e.g. `docker-compose up --scale web=3 -d`
创建3个相同配置的web container
如果要再次水平扩展，则可以 docker-compose up --scale web=10 -d
通过配合 'dockercloud/haproxy'实现负载均衡，如下 docker-compose.yaml
```
version: "3"

services:

  redis:
    image: redis

  web:
    build:
      context: .
      dockerfile: Dockerfile
    environment:
      REDIS_HOST: redis

  lb:
    image: dockercloud/haproxy
    links:
      - web
    ports:
      - 8080:80
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock 
```

