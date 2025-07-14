# Debian/Ubuntu通过certbot安装SSL证书（以Debian 12为例）

## 1. 安装certbot
```shell
apt update && apt install -y certbot python3-certbot-nginx
```

## 2. 网站根目录(/usr/share/nginx/html)创建challenge目录
```shell
mkdir -p /usr/local/nginx/html/.well-known/acme-challenge/ && echo "test" > /usr/local/nginx/html/.well-known/acme-challenge/test.txt
```

## 3. 配置Nginx

在配置文件server块中添加challenge端点

```nginx
server {
    listen 80 default_server;
    server_name localhost;

    # 添加如下内容
    location ^~ /.well-known/acme-challenge/ {
        default_type "text/plain";
        root /usr/local/nginx/html;  # 确保路径正确
    }
}
```

然后重载Nginx配置
```shell
/usr/local/nginx/sbin/nginx -s reload
```

## 4. 申请证书
比如批量申请ssl的域名是`x.com`、`y.com`、`z.com`，邮箱是`mail@x.com'
```shell
certbot certonly --webroot -w /usr/share/nginx/html \
 -d x.com \
 -d y.com \
 -d z.com \
 -m mail@x.com
```
按照提示同意协议，选择是否分享邮箱，然后等待申请完成。申请成功后会显示类似以下信息：
```
Certificate is saved at: /etc/letsencrypt/live/x.com/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/x.com/privkey.pem
```
⚠️注意：申请SSL的域名必须已经解析到服务器IP地址，否则会申请失败。

## 5. 配置Nginx使用SSL证书

在Nginx配置文件中添加SSL证书路径，确保`server_name`包含所有域名。

```nginx
server {
    listen 443 ssl;
    server_name x.com y.com z.com;

    ssl_certificate /etc/letsencrypt/live/x.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/x.com/privkey.pem;

    # 其他配置...
    
    # 保留该端点
    location ^~ /.well-known/acme-challenge/ {
        default_type "text/plain";
        root /usr/share/nginx/html;
    }
}
```

重载Nginx配置
```shell
/usr/local/nginx/sbin/nginx -s reload
```

## 6. 自动续期 （可选）

打开编辑器
```shell
crontab -e
```
添加如下内容
```shell
0 0,12 * * * /usr/bin/certbot renew --quiet
```

