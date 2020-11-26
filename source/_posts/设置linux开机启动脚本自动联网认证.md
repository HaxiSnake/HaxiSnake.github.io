---
title: 设置linux开机启动脚本自动联网认证
date: 2019-05-29 21:59:42
tags:
- 服务器维护
---

由于学校的网络需要认证上网,每次服务器上网都用个人账号认证，因此需要配置下开机启动脚本使其自动上网。(其实就是没钱惹的祸)

# 一、上网认证的脚本

先安装依赖:
``` bash
sudo apt-get install python-pip
sudo pip3 install schedule 
sudo pip3 install argparse
```

``` python task.py
import schedule
import time
from urllib.parse import quote, unquote
import requests
import argparse

def job(User,Pass):
    base_url = 'http://auth.dlut.edu.cn'
    try:
        resp = requests.get(base_url)
        print("conneting...")
    except Exception:
        #print("Already connected...")
        return
    query = resp.text[resp.text.find('wlanuserip'):resp.text.find('</script>')]
    query_str = quote(quote(query))

    url = base_url + '/eportal/InterFace.do?method=login'
    data = {
        'userId': User,  # username 1
        'password': Pass,  # password
        'service': '',  # empty
        'queryString': query_str,
        'operatorPwd': '',  # empty
        'operatorUserId': '',  # empty
        'validcode': '',  # empty
    }
    headers = {'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8'}
    resp = requests.post(url, data, headers=headers)
    print(resp.status_code)
    print(resp.text)
parser = argparse.ArgumentParser(description='connect to dlut')
parser.add_argument('-u',type=str, help='user name')
parser.add_argument('-p',type=str, help='user name')
args = parser.parse_args()
User = args.u
Pass = args.p

job(User, Pass)
schedule.every(300).minutes.do(job,User,Pass)

while True:
    schedule.run_pending()
    time.sleep(300) 
```
其主要作用是进行认证链接,命令行调用格式为:
``` bash
python3 task.py -u username -p passwd
```
username是认证用户名，passwd是对应密码

# 二、编写bash脚本调用登陆脚本

``` bash connect.sh
#!/bin/bash
python3 /home/sie/netset/task.py -u username  -p passwd &
``` 
添加可执行权限:
``` bash
chmod +x connect.sh
```
将connect.sh和task.py都放到同一目录下 设为path_to_sp
# 三、在/etc/rc.local里添加开机启动脚本命令

* 配置rc-local.service服务使其能够执行开机启动脚本 [参考教程](https://www.cnblogs.com/defifind/p/9285456.html)

* root权限创建/etc/rc.local文件并写入开机启动的脚本
``` bash
#!/bin/sh -e
# 
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.
su -s /bin/sh sie -c"path_to_sp/connect.sh"
exit 0
```
其中path_to_sp为connect.sh和task.py的目录

至此开机启动配置结束。

以前一直想搞定开机启动脚本的事情，但是总不得其要点，特此记录，以后还要多多了解原理才是。




