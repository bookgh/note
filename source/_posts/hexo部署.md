title: hexo 部署
author: 亦 漩
tags:
  - hexo
  - 部署
categories:
  - hexo
date: 2024-11-23 07:43:00
---
### 什么是 [Hexo？](https://hexo.io/zh-cn)

Hexo 是一个快速、简洁且高效的博客框架。 Hexo 使用 Markdown（或其他标记语言）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

### 使用 Docker 安装 Hexo

#### [安装 Docker](https://docs.docker.com/engine/install/binaries/#install-daemon-and-client-binaries-on-linux) 环境

``` bash
bash <(curl -sSL https://dwz.cn/XOJj0Njx) -i docker
bash <(curl -sSL https://dwz.cn/XOJj0Njx) -i compose
```
#### 构建 Hexo 镜像

> 1. 创建 Dockerfile

``` bash
mkdir -p /home/hexo/app
cat <<'EOF'  > /home/hexo/app/Dockerfile
FROM node:iron-alpine3.20

ENV LANG en_US.UTF-8

RUN sed -i 's#https://dl-cdn.alpinelinux.org#https://mirror.nju.edu.cn#g' /etc/apk/repositories \
 && apk --no-cache add bash git openssh vim expect

RUN npm config set registry https://repo.nju.edu.cn/repository/npm/ \
 && npm install -g hexo-cli hexo-server

COPY docker-entrypoint.sh /

WORKDIR /home/hexo/.hexo

RUN addgroup -S hexo -g 2000 \
 && adduser -S hexo -u 2000 -G hexo \
 && chmod +x /docker-entrypoint.sh \
 && chown hexo:hexo /docker-entrypoint.sh \
 && chown -Rf hexo:hexo /home/hexo/.hexo

USER hexo

ENTRYPOINT ["/docker-entrypoint.sh"]
EOF
```

> 2. 创建 docker-entrypoint.sh

``` bash
cat <<'EOF'  > /home/hexo/app/docker-entrypoint.sh
#!/bin/sh

if [ "$#" -gt 0 ]; then
    exec "$@"
else
    if [ ! -f _config.yml ]; then
        hexo init .
    fi

    if [ ! -d node_modules ]; then
        npm install --save
    fi

    hexo clean
    exec hexo server
fi
EOF
```

> 3. 创建 docker-entrypoint.sh

``` bash
cat <<'EOF'  > /home/hexo/docker-compose.yml
name: hexo

services:
  hexo:
    image: alpine/hexo:latest
    build:
      context: ./app
      dockerfile: Dockerfile
    restart: always
    hostname: hexo
    container_name: hexo
    ports:
      - "4000:4000"
    stdin_open: true
    tty: true
    user: 2000:2000
    stop_grace_period: 1s
    volumes:
      - /etc/localtime:/etc/localtime
      - ./data/hexo:/home/hexo/.hexo
EOF
```

> 4. 构建镜像并启动 Hexo

```bash
mkdir -p /home/hexo/data/hexo
chown -Rf 2000:2000 /home/hexo/data
docker-compose up -d --build
```

### Hexo 配置

#### 配置[主题](https://hexo.io/themes) [fluid](https://github.com/fluid-dev/hexo-theme-fluid)

- [Hexo Fluid 用户手册](https://hexo.fluid-dev.com/docs/guide/)

> [下载主题资源](https://github.com/fluid-dev/hexo-theme-fluid?#2-获取主题最新版本)&&配置权限

```bash
ghp=https://ghp.ci/
git clone ${ghp}https://github.com/fluid-dev/hexo-theme-fluid /home/hexo/data/hexo/themes/fluid
chown -Rf 2000:2000 /home/hexo/data/hexo/themes
```
> [应用主题](https://github.com/fluid-dev/hexo-theme-fluid?#3-指定主题)

```bash
sed -i "s/theme:.*/theme: fluid/" /home/hexo/data/hexo/_config.yml
```
> [语言配置](https://hexo.fluid-dev.com/docs/guide/#语言配置)

```bash
sed -i "s/language:.*/language: zh-CN" /home/hexo/data/hexo/_config.yml
```
> 配置文章[链接](https://hexo.io/zh-cn/docs/configuration#网址)

```bash
sed -i "s@permalink:.*@permalink: :title/@" /home/hexo/data/hexo/_config.yml
```



