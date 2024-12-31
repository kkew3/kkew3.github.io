---
layout: post
title:  "在 macOS 上折腾 RSS 阅读器"
date:   2024-12-31 16:39:31 +0800
tags:   rss os--macos
---

## 背景

本来之前一直用 [`NetNewsWire`][netnewswire] 阅读 RSS，直到我自己 host 的 RSS 源的 IP 地址变了，而貌似 NetNewsWire 无法修改已订阅的 RSS 源的 IP 地址（以前的版本是[可以][netnewswire-feed-url]修改的）。不得已切换到了 [`Vivaldi`][vivaldi]，这是一个浏览器，但它有一个附加的阅读 RSS 的小功能。随着 RSS 积累得越来越多，最近逐渐开始变卡，明明 `NetNewsWire` 用了那么久都很流畅的说。在[这篇博客][best-rss-reader]的指点下我找到了 [`Miniflux`][miniflux]，于是开始了一天的折腾。

## Docker 安装 Miniflux

首先尝试最简单的 [Docker 安装][miniflux-docker]，就按照官网文档的样子写好 `docker-compose.yaml`，然后

```bash
docker-compose up -d
```

就行了，不用安装 postgres 环境。但是服务起来后发现连不上自己 host 的 RSS，原来是因为那个 RSS 源也是在 Docker 容器里跑的，而且它的 network scope 为 `Local`，意思是除非其它 Docker container 和它在一个 host 上，否则就无法连接。因为我在自己 Mac 上安装的 Miniflux 容器，而 RSS 源没有 host 在本地，那显然这条路是走不通的。

## 本地安装 Miniflux

首先安装 postgres，就用最简单的 [`Postgres.app`][postgres-app]，按照网站流程一路安装即可，但是不用修改 `$PATH` 环境变量，见下文。

然后按照 Miniflux 的 [Database Configuration][miniflux-db] 页面配置 postgres，在*交互式* bash 下：

```bash
# 暂时把 Postgres.app 附带的命令行工具引入 PATH
export PATH="/Applications/Postgres.app/Contents/Versions/latest/bin:$PATH"

# 新建 postgres 用户，名为 miniflux。`-P` 是用来输入密码的。
createuser -P miniflux
# 这里输入密码，然后再输一次。我输的是 secret。之后要用。

# 在用户 miniflux 下新建名为 miniflux 的数据库。
createdb -O miniflux miniflux

# 启用 hstore 扩展
psql miniflux -c 'create extension hstore'
```

第三步新建配置文件。这里也可以用环境变量的方式，但我倾向于用配置文件。文档见[这里][miniflux-config]。我把配置文件搁在 `~/.config/miniflux/` 下。名字叫 `config`。配置文件路径其实是随意的。我是这么写的：

```bash
# 格式为 postgres://{用户}:{密码}@localhost/{数据库名}?sslmode=disable。
DATABASE_URL=postgres://miniflux:secret@localhost/miniflux?sslmode=disable

# 创建第一个用户，要不然 Miniflux 运行起来后无法登录。省去运行 `miniflux -create-admin`
# 的麻烦。
CREATE_ADMIN=1
ADMIN_USERNAME=admin
# 密码必须是六个字母及以上。
ADMIN_PASSWORD=admin123

# 省去运行 `miniflux -migrate` 的麻烦。
RUN_MIGRATIONS=1

# 我喜欢 8050 这个端口号。
LISTEN_ADDR=127.0.0.1:8050

# 如果 RSS 里有 youtube，不要忘记在这里设置 HTTP_PROXY 和 HTTPS_PROXY。
```

第四步，从 Miniflux 的 [Release][miniflux-release] 页面下载可执行程序。我是因特尔芯片的 Mac，所以下载 `miniflux-darwin-amd64`。把它放到方便的路径下，然后 bash 中运行：

```bash
# 赋予可执行权限。
chmod u+x miniflux-darwin-amd64

# 重命名一下以后敲起来方便。
mv miniflux-darwin-amd64 miniflux

# 运行 Miniflux。
./miniflux -c ~/.config/miniflux/config

# 也可以用 tmux 后台运行：
#tmux new -ds miniflux ./miniflux -c ~/.config/miniflux/config
```

最后，浏览器访问 `127.0.0.1:8050`，用户名填 `admin`, 密码填 `admin123`（即配置文件里填的那些），就可以使用了。


[netnewswire]: https://netnewswire.com/
[netnewswire-feed-url]: https://apple.stackexchange.com/a/20282
[vivaldi]: https://vivaldi.com/zh-hans/
[best-rss-reader]: https://lukesingham.com/rss-feed-reader/
[miniflux]: https://miniflux.app/
[miniflux-docker]: https://miniflux.app/docs/docker.html
[postgres-app]: https://postgresapp.com/
[miniflux-db]: https://miniflux.app/docs/database.html
[miniflux-config]: https://miniflux.app/docs/configuration.html
[miniflux-release]: https://github.com/miniflux/v2/releases
