---
title: 解决wsl2使用遇到的问题
date: 2022-05-08 16:19:33
tags:
    - 环境配置
    - wsl
categories: 环境配置
math: false
index_img: /img/article/wsl.png
---
## 1. 更新系统提示

### 1.1 安装cuda前

你会发现没有这个文件夹

`/usr/lib/wsl/lib/libcuda.so.1 is not a symbolic link`

解决方法：

1. 从`C:\Windows\System32\lxss\lib` 删除`libcuda.so` and `libcuda.so.1` 

2. 以管理员打开powershell，输入`wsl`,进入wsl环境,执行

   ```bash
   sudo ln -sr /mnt/c/Windows/System32/lxss/lib/libcuda.so.1.1 /mnt/c/Windows/System32/lxss/lib/libcuda.so.1
   sudo ln -sr /mnt/c/Windows/System32/lxss/lib/libcuda.so.1.1 /mnt/c/Windows/System32/lxss/lib/libcuda.so
   ```

### 1.2 安装cuda后

不同的系统安装位置不一样，一般来说会安装在`/usr/local/cuda/`文件夹下，但是我使用的是arch linux，使用pacman 直接安装会安装在`/opt/cuda`文件夹中

使用以下命令建立软连接

```bash
sudo ln -s /usr/lib/wsl/lib/libcuda.so.1 /opt/cuda/lib64/libcuda.so
```

## 2. 系统代理

wsl1没有独立的IP,可以直接给`127.0.0.1:port`，进行代理

更新wsl2后，需要使用正则获取IP地址

在.`bashrc`,`.zshrc`中加上

```bash
# >>> setproxy >>>
export hostip=$(cat /etc/resolv.conf |grep -oP '(?<=nameserver\ ).*')
alias setproxy='export https_proxy="http://${hostip}:10811";export http_proxy="http://${hostip}:10811";'
alias unsetproxy='unset all_proxy'
# <<< setproxy <<<
```

config.fish

```bash
# 设置代理
function unsetproxy
    set -e ALL_PROXY
end

function setproxy
    set hostip $(cat /etc/resolv.conf |grep -oP '(?<=nameserver\ ).*')
    set -xg ALL_PROXY http://$hostip:10811
end
```

注意端口，在windows代理软件中打开`allow LAN`，或者`允许局域网代理`

*测试*：

执行`setproxy`后，`curl google.com`,返回html文件

![image-20220508164357859](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202205081643996.png)

## 3. 为git设置代理

写一个简单的python 脚本`setgitproxy.py`，为git设置代理

先使用pip安装`IPy`

```python
# coding=utf-8

import socket
import fcntl
import struct
import os
from IPy import IP

# 本地代理端口，你的和我不一定一样，按照自己的设置改动一下
PORT = 10811


def get_ip_address(ifname):
    s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    return socket.inet_ntoa(fcntl.ioctl(
        s.fileno(),
        0x8915,  # SIOCGIFADDR
        struct.pack(('256s').encode("utf-8"), (ifname[:15]).encode("utf-8"))
    )[20:24])


def get_windows_ip(wsl_ip):
    ip = IP(wsl_ip).make_net("20").strNormal()
    ip = ip.split('/')[0][:-1] + '1'
    return ip


def config_git_proxy(ip):
    config_str = (
        "git config --global https.proxy 'https://%s:%d'" % (ip, PORT))
    # print(config_str)
    ret = os.system(config_str)
    if ret:
        print("set git porxy fail")
        return
    else:
        config_str = (
            "git config --global http.proxy 'http://%s:%d'" % (ip, PORT))
        # print(config_str)
        ret = os.system(config_str)
    if ret:
        print("set git porxy fail")
        return
    else:
        print("set git porxy success")
        return


if __name__ == "__main__":
    wsl_ip = get_ip_address('eth0')
    windows_ip = get_windows_ip(wsl_ip)
    # print(windows_ip)
    config_git_proxy(windows_ip)

```

使用git 命令或者其他git GUI客户端前，执行`python setgitproxy.py`为git添加代理

![image-20220508164137335](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202205081641462.png)