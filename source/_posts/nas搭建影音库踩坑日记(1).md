---
title: nas搭建影音库踩坑日记(1)：Jellyfin详解
date: 2025-07-07 19:45:31
tags:
  - nas
  - 影音库搭建
  - docker
  - docker
  - compose
categories:
  - 折腾日记
---
# Jellyfin的安装与配置
## Jellyfin的安装
### 使用一键安装脚本安装Jellyfin服务端

为了简化部署并帮助尽可能多的用户实现自动化，我们提供了一个 BASH 脚本来处理存储库安装以及在 Debian/Ubuntu 及其衍生产品上安装 Jellyfin。 您需要做的就是在系统上运行此命令 （requires ， 或 subsitute with ）：`curl``curl``wget -O-`

```shell
curl https://repo.jellyfin.org/install-debuntu.sh | sudo bash
```

> [!warning] 注意 
> 您可以使用 （requires ）：`sha256sum` 
> ```shell
> diff <( curl -s https://repo.jellyfin.org/install-debuntu.sh -o install-debuntu.sh; sha256sum install-debuntu.sh ) <( curl -s https://repo.jellyfin.org/install-debuntu.sh.sha256sum )
> ```
> 空输出表示一切正常。然后，您可以检查脚本以查看它的作用（可选但推荐）并使用以下命令执行它：
> ```shell
> less install-debuntu.sh
sudo bash install-debuntu.sh
> ```

### 使用 docker 部署 Jellyfin服务端

1. 使用 docker 命令拉取 Jellyfin 镜像
```shell
docker pull jellyfin/jellyfin
```

2. 使用命令创建 Jellyfin 的管理文件夹。
```shell
mkdir /path/to/config   # 配置文件夹，用于存储Jellyfin的配置文件
mkdir /path/to/cache    # 缓存文件夹，用于存储Jellyfin的缓存
```

或者创建两个持久卷：

```shell
docker volume create jellyfin-config
docker volume create jellyfin-cache
```

> [!warning] 注意
> Docker 的默认网络模式是桥接模式。如果省略 host mode，将使用 Bridge 模式。 使用 host networking （） 是可选的，但要使用 DLNA，这是必需的。`--net=host`

3. 使用 docker 命令创建并运行 Jellyfin 容器
```shell
docker run -d \
 --name jellyfin \
 --user uid:gid \
 --net=host \
 --volume /path/to/config:/config \ # 将容器内的配置文件文件夹映射到本地
 --volume /path/to/cache:/cache \ # 将容器内的缓存目录映射到本地
 --mount type=bind,source=/path/to/media,target=/media \ # 配置媒体库文件夹，也可以使用目录映射实现
 --restart=unless-stopped \
 jellyfin/jellyfin
```

### 使用 docker compose 部署 Jellyfin 服务端
创建一个 `docker-compose.yml` ，在 `yml` 文件中写入以下内容：
```yaml
services:
  jellyfin:
    image: jellyfin/jellyfin
    container_name: jellyfin
    user: uid:gid
    network_mode: 'host'
    volumes:
      - /path/to/config:/config
      - /path/to/cache:/cache
      - type: bind
        source: /path/to/media
        target: /media
      - type: bind
        source: /path/to/media2
        target: /media2
        read_only: true
      # Optional - extra fonts to be used during transcoding with subtitle burn-in
      - type: bind
        source: /path/to/fonts
        target: /usr/local/share/fonts/custom
        read_only: true
    restart: 'unless-stopped'
    # Optional - alternative address used for autodiscovery
    environment:
      - JELLYFIN_PublishedServerUrl=http://example.com
    # Optional - may be necessary for docker healthcheck to pass if running in host network mode
    extra_hosts:
      - 'host.docker.internal:host-gateway'
```

在命令行中输入以下命令运行项目：
```shell
docker compose up -d      # 如果不需要在后台运行时，去掉 -d 参数
```

## Jellyfin 基础配置

Jellyfin安装好后，在浏览器地址栏里输入 `http://服务器IP地址:8096` 就可以打开 Jellyfin 的后台管理系统。
![[../_images/Pasted image 20250707214023.png]]

在界面中，可以选择语言为中文。之后，可以设置Jellyfin的用户名和密码：
![[../_images/Pasted image 20250707214204.png]]

接下来就可以设置媒体库，可以添加已经准备好的媒体库目录。
![[../_images/Pasted image 20250707214402.png]]
首选下载语言全部选择中文即可。首选元数据语言全部设置为中文。
最后，点击下一步即可设置完成。
接下来，输入设置好的用户名和密码，即可登录进入Jellyfin。
Jellyfin的基础设置就完成了。