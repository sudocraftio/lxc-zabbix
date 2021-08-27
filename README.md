# lxc-zabbix
How To Install Zabbix In LXC Container

```
lxc launch ubuntu:20.04 zabbix
lxc launch ubuntu:20.04 proxy
```

```
lxc config device add proxy myport80 proxy listen=tcp:0.0.0.0:80 connect=tcp:127.0.0.1:80 proxy_protocol=true
lxc config device add proxy myport443 proxy listen=tcp:0.0.0.0:443 connect=tcp:127.0.0.1:443 proxy_protocol=true
lxc config device add zabbix myport10051 proxy listen=tcp:0.0.0.0:10051 connect=tcp:127.0.0.1:10051
```

# Nginx Proxy

```
sudo lxc exec proxy -- /bin/bash
sudo apt update
sudo apt install nginx -y
sudo systemctl enable nginx.service
```

```
sudo nano /etc/nginx/sites-available/zabbix
```

```
server {
        listen 80 proxy_protocol;
        listen [::]:80 proxy_protocol;
 
        server_name example.com; # Your URL
 
        location / {
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_pass http://zabbix.lxd;
        }
 
        real_ip_header proxy_protocol;
        set_real_ip_from 127.0.0.1;
}
```

```
sudo ln -s /etc/nginx/sites-available/zabbix /etc/nginx/sites-enabled/
sudo systemctl reload nginx
```

# Zabbix

```
sudo lxc exec zabbix -- /bin/bash
sudo apt update
sudo apt-get install mariadb-server mariadb-client -y
sudo systemctl start mariadb.service
sudo systemctl enable mariadb.service
```

```
sudo mysql
create database zabbix character set utf8 collate utf8_bin;
create user zabbix@localhost identified by 'password';
grant all privileges on zabbix.* to zabbix@localhost;
flush privileges;
exit;
```

```
wget https://repo.zabbix.com/zabbix/5.2/ubuntu/pool/main/z/zabbix-release/zabbix-release_5.2-1+ubuntu20.04_all.deb
dpkg -i zabbix-release_5.2-1+ubuntu20.04_all.deb
sudo apt update
```

```
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-nginx-conf zabbix-agent -y
sudo zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
```

```
sudo nano /etc/zabbix/zabbix_server.conf
```

```
DBPassword=password
```

```
sudo nano /etc/zabbix/nginx.conf
```

```
sudo systemctl restart zabbix-server zabbix-agent nginx php7.4-fpm
sudo systemctl enable zabbix-server zabbix-agent nginx php7.4-fpm
exit
```
