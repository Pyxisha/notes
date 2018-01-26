---
title: "2018-01-11 how to use zabbix"
date: 2018-01-12 16:06
---

##Zabbix使用

####添加源 

```
rpm -ivh http://repo.zabbix.com/zabbix/3.4/rhel/7/x86_64/zabbix-release-3.4-2.el7.noarch.rpm --或者可以換成其他版本
```

####Server端安裝

```
yum install zabbix-server-mysql zabbix-get
yum install httpd php php-mysql php-mbstring php-gd php-bcmath php-ldap php-xml
yum install zabbix-web zabbix-web-mysql
```

####server端部署

1. 導數據庫， 創建數據庫文件，sql文件在/usr/share/doc/zabbix-server-mysql-{version}下。
2. 修改配置，配置在/etc/zabbix/zabbix_server.conf,必須修改的項目有數據庫用戶名密碼等,listen地址端口。
3. 啓動server，systemctl start zabbix-server
4. 修改web配置，/etc/httpd/conf.d/zabbix.conf,必要修改的有時區,/etc/httpd/conf/httpd.conf,修改監聽地址等。
5. 啓動web，systemctl start httpd， 進入http://host/zabbix,初次會有配置，之後以用戶名admin，密碼zabbix可以登錄。

這里可能遇到的問題：

1. systemctl服務啓動失敗，很有可能是selinux的問題, 解決辦法： setenforce 0， 關閉selinux
2. curl host => no route to host

       1. 防火牆問題： killall firewalld
       2. Iptables問題： 關閉iptables或者清除iptables規則

####Agent端

```
yum install zabbix-agent zabbix-sender
```

配置文件 /etc/zabbix-agentd.conf，主要需要修改server端的地址。

啓動agent systemctl start zabbix-agent

也有可能有selinux的問題。

####配置使用腳本的流程

##### 添加主機：
 
Configuration => Hosts => Create host

##### 添加監控項目

Configuration => Hosts 看到主機列表

items => create item


Key => 選擇要監控的字段

Store value => 可以選擇是記錄差值還是原值

Applications 應用集也是指的是一個集合

##### 添加media types，這裏添加腳本處理的邏輯

Administration => Meida types => Create media type 

類型選擇Script

Parameters指定腳本的參數

執行腳本默認應該放在/usr/lib/zabbix/alertscripts當中

路徑可以在conf當中的AlertScriptPath這一項來配置

##### 創建觸發器

Configuration => Hosts => triggers => create trigger

比較重要的配置expression，定義來報警的觸發條件

##### 創建action

Configuration => actions => create action

Condition當中定義觸發的條件

Operations定義執行的行爲

##### monitoring => events 當中可以action觸發的情況

可能遇到的問題: 

1. 腳本沒有執行權限
2. No media defined for user： 到Administration=> user =>{user} =>media  當中配置啓用


####參考腳本

```python
 #!/usr/bin/python

import requests
import sys
import time

log_file = "/test/wechat.log"
logger = open(log_file,"a")
usage = """usage:
./wechat.py sendto subject message
"""
logger.write(str(time.time())+"\n")

if len(sys.argv) < 4:
    print(usage)
    logger.write(usage)
    exit()

access_token = "access_token=1IOiV7rGzMaxQD_nNl17GtHNu7Osz91vNlRBj7hEHrhZVpvwBF_qU4fMCfFacmu5QvFjlr3jXYJ6Mlybheq8$
url = "https://qyapi.weixin.qq.com/cgi-bin/message/send?"
text = """
sendto: %s
subject: %s
message: %s
""" % (sys.argv[1],sys.argv[2],sys.argv[3])

content = {
   "toparty": "4",
   "msgtype": "text",
   "agentid": 0,
   "text": {
       "content": text
   },
   "safe":0
}

r = requests.post(url+access_token,json=content)
logger.write(r.text)
```