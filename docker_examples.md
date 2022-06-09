## Docker 示例

### redis

```bash
# 如果执行了至少 1 次写入操作， 将每 60 秒保存一次数据库快照
docker run \
--name myredis \
-p 6379:6379 \
--restart=always \
-v /data/redis:/data \
-d redis redis-server \
--save 60 1 \
--loglevel warning

# 连接redis
docker run \
-it \
--rm redis redis-cli \
-h hostAddr

# 自定义config
docker run \
-v /myredis/conf:/usr/local/etc/redis \
--name myredis redis redis-server /usr/local/etc/redis/redis.conf
```

### nginx

```bash
docker run \
-p 80:80 \
--name mynginx \
-d nginx

# 自定义webroot（/data/root:/usr/share/nginx/html）
# 自定义配置（/data/nginx:/etc/nginx）
# 只读模式（:ro）
docker run \
-p 80:80 \
-p 443:443 \
--name mynginx \
-v /data/root:/usr/share/nginx/html:ro \
-v /data/nginx:/etc/nginx:ro \
--restart=always  \
-d nginx
```

默认配置

```bash
# --rm 容器退出时自动移除
docker run --rm nginx cat /etc/nginx/nginx.conf
```

/etc/nginx/nginx.conf

```conf
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}

```

### mysql

```bash
docker run \
--name mymysql \
--restart=always \
-v /data/mysql:/var/lib/mysql \
-p 3306:3306 \
-e MYSQL_ROOT_PASSWORD=123456 \
-d mysql:5

# 开启binlog
docker run \
--name mymysql \
-e MYSQL_ROOT_PASSWORD=123456 \
-d mysql:5 \
--log-bin=mysql-bin.log \
--binlog-format=ROW \
--server-id=1


# 连接mysql
docker run \
-it \
--rm mysql mysql \
-hmyhost \
-uroot \
-p123456
```

## 其他

### busybox

BusyBox 是一个集成了一百多个最常用 Linux 命令和工具（如 cat、echo、grep、mount、telnet 等）的精简工具箱，它只需要几 MB 的大小，很方便进行各种快速验证，被誉为“Linux 系统的瑞士军刀”。


telnet

```bash
docker run --rm busybox telnet google.com 443
# Connected to google.com
```


nslookup

```bash
docker run --rm busybox:1.28 nslookup zhihu.com
# Unable to find image 'busybox:1.28' locally
# 1.28: Pulling from library/busybox
# Digest: sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
# Status: Downloaded newer image for busybox:1.28
# Server:    10.xx.xx.1
# Address 1: 10.xx.xx.1

# Name:      zhihu.com
# Address 1: 103.41.167.234
```


运行自定义可执行文件

```dockerfile
# Use busybox as the base image
FROM busybox
# Copy over the executable file
COPY ./main /home/
# Run the executable file
CMD /home/main
```

### dnsutils

nslookup

```bash
docker run --rm tutum/dnsutils nslookup zhihu.com
# Server:        10.xx.xx.1
# Address:    10.xx.xx.1#53

# Non-authoritative answer:
# Name:    zhihu.com
# Address: 103.41.167.234

```

dig

```bash
docker run --rm tutum/dnsutils dig zhihu.com
# ; <<>> DiG 9.9.5-3ubuntu0.2-Ubuntu <<>> zhihu.com
# ;; global options: +cmd
# ;; Got answer:
# ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 31544
# ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

# ;; QUESTION SECTION:
# ;zhihu.com.            IN    A

# ;; ANSWER SECTION:
# zhihu.com.        120    IN    A    103.41.167.234

# ;; Query time: 0 msec
# ;; SERVER: 10.52.44.1#53(10.52.44.1)
# ;; WHEN: Wed Jun 01 12:12:42 UTC 2022
# ;; MSG SIZE  rcvd: 43

```

### openssh-server

生成一个 ssh 私钥/公钥

```bash
docker run --rm -it --entrypoint /keygen.sh linuxserver/openssh-server

# Please select your key type to generate
# 1.) ecdsa
# 2.) rsa
# 3.) ed25519
# 4.) dsa
# [default ecdsa]:
# blank or unknown option choosing ecdsa
# YOUR KEY/PUBFILE IS BELOW PLEASE SAVE THIS DATA AS WE WILL NOT
# -----BEGIN OPENSSH PRIVATE KEY-----
# b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAArAAAABNlY2RzYS
# 1zaGEyLW5pc3RwNTIxAAAACG5pc3RwNTIxAAAAhQQBN9BCHJ1hQRs1wZeNjxC5lkKeXm21
# qGyseO8XFmwtsNO4WgLaqsKOVBFa+U97LevtrmKMlDx5NGw5Jlk0PnGQ+lIAkdgaXjm5UZ
# j6UqM1mZRdtP9EjoJhoFZ/UXdl/i/mAmvZEsmwpEvi31yDbZAW3htc1Q0GIWOCUsPcFtpg
# AAUtOIMAAAEQ29mYA9vZmAMAAAATZWNkc2Etc2hhMi1uaXN0cDUyMQAAAAhuaXN0cDUyMQ
# AAAIUEATfQQhydYUEbNcGXjY8QuZZCnl5ttahsrHjvFxZsLbDTuFoC2qrCjlQRWvlPey3r
# 7a5ijJQ8eTRsOSZZND5xkPpSAJHYGl45uVGY+lKjNZmUXbT/RI6CYaBWf1F3Zf4v5gJr2R
# LJsKRL4t9cg22QFt4bXNUNBiFjglLD3BbaYAAFLTiDAAAAQgEJnV/U0XnMcdVNYPO8M3wR
# /vHlHTGzJZPw72LbtSkplmLbA4A5LQiHoKlCYF0Xx+ERBw4mWbwxK2QppDhfUExHKwAAAB
# Fyb290QDU0MGIzZjhiMzcyNQE=
# -----END OPENSSH PRIVATE KEY-----
# ecdsa-sha2-nistp521 AAAAE2VjZHNhLXNoYTItbmlzdHA1MjEAAAAIbmlzdHA1MjEAAACFBAE30EIcnWFBGzXBl42PELmWQp5ebbWobKx47xcWbC2w07haAtqqwo5UEVr5T3st6+2uYoyUPHk0bDkmWTQ+cZD6UgCR2BpeOblRmPpSozWZlF20/0SOgmGgVn9Rd2X+L+YCa9kSybCkS+LfXINtkBbeG1zVDQYhY4JSw9wW2mAABS04gw== root@xxx
```

## 参考

- [redis - docker hub](https://hub.docker.com/_/redis)
- [nginx - docker hub](https://hub.docker.com/_/nginx)
- [mysql - docker hub](https://hub.docker.com/_/mysql)
- [BusyBox.net](https://busybox.net/downloads/BusyBox.html)
- [busybox guide - sohamkamani](https://www.sohamkamani.com/docker/busybox-guide/)