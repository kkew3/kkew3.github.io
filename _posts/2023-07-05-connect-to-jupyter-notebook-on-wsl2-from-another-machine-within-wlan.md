---
layout: post
title:  "在 WLAN 下从另一台计算机连接到 WSL2 中的 Jupyter Notebook"
date:   2023-07-05 15:57:59 +0800
categories: network
---

# 摘要

[上一篇文章](2023-07-05-connect-to-wsl2-from-another-machine-within-wlan.md)介绍了如何从本地连接到同一 WLAN 下的另一台 Windows 计算机中的 WSL2 实例.
这篇文章进一步介绍如何连接到该 WSL2 实例中运行的 [Jupyter Notebook](https://jupyter.org).

# 原理

1. 如上一篇文章所述建立由本地到 Windows (IP 地址本文为 `192.168.0.105`, 用户名本文为 `ubuntu`) 的 SSH 连接 (端口本文为 `4000`)
2. 在 WSL2 实例的端口 `8890` 运行无浏览器的 Jupyter Notebook
3. 在本地建立将本地端口 (本文为 `8889`) 转发到远程 Windows 端口 `8890` 的 SSH 隧道
4. 在本地 `localhost:8889` 访问远程 Jupyter Notebook

# 具体流程

## 运行 Jupyter Notebook

*以下命令在 WSL2 的终端中执行*

```bash
jupyter notebook --no-browser --port 8890
```

## 建立 SSH 隧道

*以下命令在本地终端执行*

```bash
ssh -p 4000 -NL 8889:localhost:8890 ubuntu@192.168.0.105
```

# 参考

- [关于 SSH 隧道](https://medium.com/@apbetahouse45/how-to-run-jupyter-notebooks-on-remote-server-part-1-ssh-a2be0232c533)
