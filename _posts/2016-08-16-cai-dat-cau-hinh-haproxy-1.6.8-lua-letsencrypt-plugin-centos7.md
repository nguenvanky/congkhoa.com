---
layout: post
title:  "Cài đặt và cấu hình HAProxy 1.6.8 + Lua + Let's Encrypt plugin trên Centos 7"
date: 2016-08-16 13:00:05 +0700
categories: [ops, haproxy]
---

Để cài Let's Encrypt plugin cho HAProxy thì cần bản HAProxy hỗ trợ nhúng Lua script (1.5+) nên dùng bản mới nhất 1.6.8
Phiên bản HAProxy 1.6.8 chưa được hỗ trợ mặc định trong REHL7 nên phải build từ source. 
TIPS: Bạn có thể cài bản mặc định của REHL7 để tận dụng mấy cái cấu hình systemd, tạo user... đỡ phải làm. 

``` bash
$ yum install readline readline-devel wget gcc pcre-static pcre-devel haproxy -yum
```

**Bước 1. Cài đặt lua 5.3.3**

``` bash
$ cd ~
$ curl -R -O http://www.lua.org/ftp/lua-5.3.3.tar.gz
$ tar zxf lua-5.3.3.tar.gz
$ cd lua-5.3.3
$ sudo make linux
$ sudo make INSTALL_TOP=/usr/local/lua53 install
```

**Bước 2. Cài đặt HAProxy 1.6.8 (enable Lua)**

``` bash
$ wget http://www.haproxy.org/download/1.6/src/haproxy-1.6.8.tar.gz -O ~/haproxy.tar.gz
$ tar xzvf ~/haproxy.tar.gz -C ~/
$ cd ~/haproxy-1.6.8
$ make TARGET=linux2628 USE_OPENSSL=1 USE_PCRE=1 USE_LUA=1 \
    LUA_LIB=/usr/local/lua53/lib/ \
    LUA_INC=/usr/local/lua53/include/
$ make install
$ cp /usr/local/sbin/haproxy /usr/sbin/
```

có thể thêm `USE_PCRE_JIT=1` để hỗ trợ `USE_PCRE_JIT`.

**Bước 3. Cài đặt Let's Encrypt client `certbot`**

``` bash
$ yum install certbot -y
```

**Bước 4. Issue cetificate**
Bước này để đăng ký domain với Let's Encrypt và nhận lại key, cert đỡ phải tạo key thủ công nhiều bước.

``` bash
$ certbot certonly --text --webroot --webroot-path /var/lib/haproxy -d yourdomain.com \
        --renew-by-default --agree-tos --email name@company
```
* Lưu ý: `--webroot-path` trỏ về `/var/lib/haproxy` là thư mục chứa code để Let's Encrypt verify bạn có thực sự sở
hữu domain này không.
Bằng cách load file `http://yourdomain.com/.well-known/acme-challenge/<hash-file-name>` 
của thư mục `/var/lib/haproxy/.well-known/acme-challenge/<hash-file-name>` được tạo sau khi chạy bước 4

**Bước 5. Tạo file pem cho từng domain**

Sau khi chạy thành công bước trên bạn sẽ nhận được thông báo file *.pem đã được tạo và để ở thư mục `/etc/letsencrypt/live/yourdomain.com/`.
Lúc này cần gom 2 file này lại để dùng cho cấu hình SSL của Haproxy. Chạy lệnh:

``` bash
$ cat /etc/letsencrypt/live/yourdomain.com/privkey.pem \
    /etc/letsencrypt/live/yourdomain.com/fullchain.pem | \
    sudo tee /etc/letsencrypt/live/yourdomain.com/haproxy.pem >/dev/null`
```

**Bước 6. Cấu hình SSL cho HAProxy**

* Clone repo `haproxy-acme-validation-plugin` [ở đây](https://github.com/janeczku/haproxy-acme-validation-plugin)

* Copy file `acme-http01-webroot.lua` vào thư mục `/etc/haproxy/`

* Cấu hình HAProxy theo [mẫu sau](https://github.com/janeczku/haproxy-acme-validation-plugin/blob/master/haproxy.cfg.example)

* Lưu ý nếu chạy nhiều domain _(yourdomain2.com, yourdomain3.com)_ thì lặp lại bước 4 và 5 cho mỗi domain.

* Cấu hình SSL cho nhiều domain của HAProxy bằng cách thêm file crt như sau:

``` bash
bind *:443 ssl crt /etc/letsencrypt/live/yourdomain.com/haproxy.pem crt /etc/letsencrypt/live/yourdomain2.com/haproxy.pem /etc/letsencrypt/live/yourdomain3.com/haproxy.pem 
```

**Bước 7. Cấu hình cronjob để tự động renew Certificates**

Do Certificates sẽ hết hạn sau 3 tháng nên cần liên tục renew với server của Lets Encrypt.

* Mở file `cert-renewal-haproxy.sh` clone được ở bước 6, chỉnh sửa cấu hình phần `configuration` giống như sau:

``` python
EMAIL="name@company" #(email của bạn đăng ký ở bước trên, cần giống nhau đối với nhiều domain)

LE_CLIENT="/path/to/letsencrypt-client" #(ở đây là file /usr/bin/certbot)

HAPROXY_RELOAD_CMD="service haproxy reload"

WEBROOT="/var/lib/haproxy"
```
* Copy file bash vào thư mục tùy ý (nên là `/usr/local/bin`)

``` bash
$ cp /path/to/haproxy-acme-validation-plugin/cert-renewal-haproxy.sh /usr/local/bin/cert-renewal-haproxy.sh
$ chmod +x /usr/local/bin/cert-renewal-haproxy.sh
```

* Thêm dòng sau vào crontab:

```bash
$ crontab -e
``` 
`5 8 * * 6 /usr/bin/cert-renewal-haproxy.sh` 

**Bước 8. Cấu hình log cho HAProxy(option)**
Dùng syslog để ghi log cho Haproxy.

* cấu hình `/etc/haproxy/haproxy.cfg`:
Trong file cấu hình cần có dòng cấu hình như dưới: 

``` bash
global
log         127.0.0.1 local2
[...]
``` 

* Cấu hình syslog:

Thêm 3 dòng sau vào cấu hình của syslog `/etc/rsyslog.conf`:

``` bash
ModLoad imudp
UDPServerRun 514
UDPServerAddress 127.0.0.1
```
Tạo file `/etc/rsyslog.d/haproxy.conf` có nội dung như sau:

``` bash
local2.=info     /var/log/haproxy-info.log 
local2.notice    /var/log/haproxy-allbutinfo.log
```

* restart syslog, haproxy:

``` bash
$ service rsyslog restart
$ service haproxy restart
```

_PS: Bài viết sử dụng hướng dẫn của:_

_[https://github.com/janeczku/haproxy-acme-validation-plugin](https://github.com/janeczku/haproxy-acme-validation-plugin)_

_[http://blog.haproxy.com/2015/10/14/whats-new-in-haproxy-1-6/](http://blog.haproxy.com/2015/10/14/whats-new-in-haproxy-1-6/)_

_[https://www.digitalocean.com/community/tutorials/how-to-implement-ssl-termination-with-haproxy-on-ubuntu-14-04](https://www.digitalocean.com/community/tutorials/how-to-implement-ssl-termination-with-haproxy-on-ubuntu-14-04)_

_[http://serverfault.com/questions/560978/configure-multiple-ssl-certificates-in-haproxy](http://serverfault.com/questions/560978/configure-multiple-ssl-certificates-in-haproxy)_


