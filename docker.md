# Dockerfile

todo 11个 Dockerfile 命令

搞明白 RUN、CMD 和 ENTRYPOINT 的区别

## 使用 Docker 部署 React App

```bash
npx create-react-app react-for-docker

cd react-for-docker
npm run build

touch Dockerfile
touch default.conf

docker build -t react-app .
docker images

docker run -d -p 3000:80 --name app react-app
docker ps -a
docker logs -f --tail 100 [Container ID]

curl -v -i localhost:3000

docker tag react-app shaohsiung/reactapp:beta
docker login
docker push shaohsiung/reactapp:beta
```

Dockerfile

```dockerfile
FROM nginx
COPY build/ /usr/share/nginx/html/
COPY default.conf /etc/nginx/conf.d/default.conf
```

default.conf

```bash
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    access_log  /var/log/nginx/host.access.log  main;
    error_log  /var/log/nginx/error.log  error;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

todo for nginx：

- [ ] Nginx 配置文件
- [ ] Nginx 配置反向代理
- [ ] Nginx 配置虚拟主机
- [ ] Nginx 配置负载均衡





