### 部署教程

1.将本目录放置在你linux 服务器的 /home/chatgpt 下面, 如图所示
推荐和我放置在同一目录

<img src="https://jameshao.pro/upload/2023/04/pathDemo.png" alt="Alt text" title="Optional title"/>

#### 服务器环境准备

准备好一台Linux服务器，安装好 Docker 和 Docker-Compose ，并且开放 80 和 443 端口

```yml
# 此处有安装教程
https://yeasy.gitbook.io/docker_practice/install/ubuntu
```

以下是快速安装Docker和Docker Compose的步骤：

1.安装Docker

可以使用以下shell命令安装Docker：

```
curl -fsSL https://get.docker.com -o get-docker.sh 
sh get-docker.sh

```

2.启动Docker服务

在安装完Docker后，可以使用以下命令启动Docker服务：

```
sudo systemctl start docker   # Ubuntu/CentOS

```

3.安装Docker Compose

在Linux系统上，可以使用以下命令安装Docker Compose：

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
```

4.检查安装

最后，您可以使用以下命令检查Docker和Docker Compose是否成功安装：

```
docker --version
docker-compose --version
```

如果这些命令都输出了版本信息，则表示Docker和Docker Compose已经成功安装并准备好使用了。


#### 域名和证书准备

1.申请域名，推荐 https://www.namesilo.com/ 去这里申请，价格很低

2.做好域名解析， 以下是推荐解析
- chat.example.com  解析到你的服务器ip 用于部署 chatgpt-web
- pt.example.com    解析到你的服务器ip 用于部署 portainer

![0e956000ca21](https://jameshao.pro/upload/2023/04/92a8b0e956000ca21312d20f0b2a8d8.png)
![21312d20f0b2](https://jameshao.pro/upload/2023/04/92a8b0e956000ca21312d20f0b2a8d8.png)



3.使用acme申请`ssl`证书

```sh
# 安装acme
curl https://get.acme.sh | sh
# 添加软链接
ln -s /root/.acme.sh/acme.sh /usr/local/bin/acme.sh
# 切换CA机构
acme.sh --set-default-ca --server letsencrypt
# 申请证书
acme.sh --issue -d 域名 --standalone --keylength ec-256
# 安装证书
acme.sh --install-cert -d 域名 --ecc --fullchain-file /etc/ssl/private/fullchain.cer --key-file /etc/ssl/private/private.key

这是使用acme.sh（一个开源的ACME客户端）来安装SSL证书的命令，具体含义如下：

--install-cert：用于安装证书。
-d 域名：指定要为其申请和安装证书的域名。
--ecc：表示生成椭圆曲线加密算法（ECC）证书。
--fullchain-file /etc/ssl/private/fullchain.cer：将服务器的完整证书链保存到指定的文件路径。
--key-file /etc/ssl/private/private.key：将私钥保存到指定的文件路径。

综合起来，该命令的作用是在服务器上为指定域名安装ECC SSL证书，并将完整证书链保存到/etc/ssl/private/fullchain.cer文件中，并将私钥保存到/etc/ssl/private/private.key文件中

# 赋予执行权限
chmod -R +x /etc/ssl/private/

```

4.把chatgpt-web 项目的证书和私钥放置在 `docker-compose\nginx\ssl` 目录下

5.把 Portainer 容器管理工具项目的证书和私钥放置在   `docker-compose\nginx\portainer_ssl` 目录下

注意：如果你的文件目录和我的位置不一样的情况你需要自行修改 docker-compose.yml -volumes 挂载的数据卷目录，和我的保持一致，你不需要修改，省很多事

```yml
 volumes:
      - ./nginx/html:/usr/share/nginx/html
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf
      - ./nginx/portainer.conf:/etc/nginx/conf.d/portainer.conf
      - ./nginx/cockpit.conf:/etc/nginx/conf.d/cockpit.conf
      - /home/chatgpt/docker-compose/nginx/ssl:/etc/nginx/ssl
      - /home/chatgpt/docker-compose/nginx/portainer_ssl:/etc/nginx/portainerSSL
    depends_on:
      - app
```


#### 修改配置文件

需要修改的文件，按准注释标识修改就行
```
docker-compose\nginx\nginx.conf
docker-compose\nginx\portainer.conf
```
- docker-compose\nginx\nginx.conf 是 chatGPT-WEB 项目的nginx 配置 
- docker-compose\nginx\portainer.conf 是 Portainer 项目的nginx 配置 


```conf
# 暂时没用上，不用管
docker-compose\nginx\cockpit.conf
```

#### 启动项目
当你一起准备就绪之后，直接 cd /home/chatgpt/docker-compose
注意： 一定要cd到 /home/chatgpt/docker-compose 目录才能运行 docker-compose up -d

```
docker-compose up -d
```

#### 启动成功
当你看到如下结果，说明你启动成功了
![20230408112350](https://jameshao.pro/upload/2023/04/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20230408112350.png)

浏览器输入 chat.example.com 即可进入你的chatGPT-web 项目

#### 设置 Portainer  开源的容器管理工具
当你启动成功之后你需要打开浏览器输入 `pt.example.com` 进入到你的 Portainer 项目，
首次登陆需要注册用户，给admin用户设置密码
一定要快。他是有时间限制的，超时进不去的，超时了你只能重启容器 `docker-compose restart`

![Portainer-project](https://jameshao.pro/upload/2023/04/ui%E5%88%9B%E5%BB%BA%E7%94%A8%E6%88%B7%E5%AF%86%E7%A0%81.png)


#### Portainer是什么？

Portainer是一款免费且开源的容器管理工具，可以用来简化Docker容器的部署、管理和监控。Portainer提供一个易于使用的Web UI，使用户能够轻松地管理单个Docker主机或群集中的多个Docker节点。Portainer支持几乎所有Docker API，并为容器、镜像、网络、卷等对象提供了可视化的管理界面。此外，Portainer还支持Swarm模式，并提供了一些高级功能，如模板和堆栈管理、访问控制等。

!['0xjfdsa44fds'](https://jameshao.pro/upload/2023/04/image.png);


#### Docker Compose 是什么？如何使用

Docker Compose是一个用于编排Docker容器的工具，可以通过简单的YAML文件来定义和运行多个相关容器。下面是一些常见的Docker Compose基本操作：

Docker Compose 是一个用于定义和运行多容器 Docker 应用程序的工具。以下是一些常用的命令

`docker-compose up` - 构建并启动应用程序容器
`docker-compose up -d` - 后台运行构建并启动应用程序容器

`docker-compose down` - 停止并移除应用程序容器和相关网络等资源。

`docker-compose ps` - 列出正在运行的所有容器。

`docker-compose logs` - 查看容器的日志输出。

`docker-compose pull` - 从注册表拉取最新的镜像版本。

`docker-compose build` - 构建应用程序的镜像。

`docker-compose restart` - 重启应用程序的所有容器。

`docker-compose stop` - 停止应用程序的所有容器。

`docker-compose start` - 启动应用程序的所有容器。

`docker-compose exec`- 在指定容器中执行命令。
