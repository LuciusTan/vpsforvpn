#### 准备工作：

##### 1. 新建 CentOS 8 系统的 VPS

##### 2. 注册域名，将域名解析到 vps 的 ip

---

#### 1. 安装宝塔面板

```
curl -sSO http://download.bt.cn/install/new_install.sh && bash new_install.sh
```

> 安装完记录下 Bt-Panel-URL 地址，账户，密码

#### 2. 进入宝塔面板，安装 Nginx 服务

#### 3. 网站菜单 - 添加站点按钮 - （输入域名）

#### 4. 安装 v2ray

```
bash <(curl -L -s https://install.direct/go.sh)
```

> 安装完成后，记录下 port - 12345 和 uuid - xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

#### 5. 配置 SSL

##### 网站菜单 - 域名（设置） - SSL - let's Encrypt - 开启强制 https 开关

##### 网站菜单 - 域名（设置） - 配置文件 - 在 #SSL-END 下面写入以下代码 - 保存

```
    location /vps
    {
        proxy_pass http://127.0.0.1:上面的端口号12345;
        proxy_redirect off;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;
        proxy_read_timeout 300s;
    }
```

> 其中第一行的 vps 是自己 ws 的 path，斜杠不要删除，可以自己修改,也可以不改

#### 6. 配置 v2ray 配置文件

##### 进入 vps 系统，文件路径 /etc/v2ray - 编辑 config.json 文件，替换代码：（修改注释项目，并删除注释）

```
{
  "policy": {
    "levels": {
      "0": {
        "uplinkOnly": 0,
        "downlinkOnly": 0,
        "connIdle": 150,
        "handshake": 4
      }
    }
  },
  "inbound": {
    "listen": "127.0.0.1",
    "port": 12345,    //填写上面的 v2ray 端口号 12345
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",    //填写上面的 v2ray UUID
          "level": 1,
          "alterId": 32
        }
      ]
    },
    "streamSettings": {
      "network": "ws",
      "security": "auto",
      "wsSettings": {
        "path": "/vps",   //填上面设置的 ws 的 path, 如果修改配置文件的时候没有修改过就是 /vps
        "headers": {
          "Host": "www.xxx.com"  //填写域名
        }
      }
    }
  },
  "outbound": {
    "protocol": "freedom",
    "settings": { }
  },
  "outboundDetour": [
    {
      "protocol": "blackhole",
      "settings": { },
      "tag": "blocked"
    }
  ],
  "routing": {
    "strategy": "rules",
    "settings": {
      "rules": [
        {
          "type": "field",
          "ip": [
            "0.0.0.0/8",
            "10.0.0.0/8",
            "100.64.0.0/10",
            "127.0.0.0/8",
            "169.254.0.0/16",
            "172.16.0.0/12",
            "192.0.0.0/24",
            "192.0.2.0/24",
            "192.168.0.0/16",
            "198.18.0.0/15",
            "198.51.100.0/24",
            "203.0.113.0/24",
            "::1/128",
            "fc00::/7",
            "fe80::/10"
          ],
          "outboundTag": "blocked"
        }
      ]
    }
  }
}
```

#### 7. 开启防火墙端口

##### 宝塔安全菜单 - 防火墙 - 放行端口 port - 12345

##### 服务器中启动 v2ray

```
systemctl start v2ray

#更多v2ray命令
service v2ray start|stop|status|reload|restart|force-reload
```

#### 8. 配置 CDN

https://cdn.bnxb.com/index.html

##### 域名接入

##### 修改域名解析 CNAME -> BNXB 提供的域名

#### 9. 配置客户端

```
地址 -> 域名
端口 -> 443
id -> uuid
额外id - 32
加密方式 - 128-gcm
传输协议 - ws
路径 -> 如未修改，即为 /vps
底层传输安全 - tls - true
```

#### 10. 可按需配置 PAC 等模式
