# JConnect Installation
>
>
>
# Install NodeJs
## สำหรับไว้ Run NodeRed

> sudo apt update

> curl -sL https://deb.nodesource.com/setup_16.x | sudo bash -

> sudo apt install -y nodejs


#### Command Optional
- *Check Version*
> node -v

- *Uninstall*
> sudo apt-get remove nodejs
> 
> sudo apt-get purge nodejs
> 
> sudo apt-get autoremove

***
# Install Node Red
## สำหรับไว้เชื่อมต่ออุปกรณ์ต่างๆ ผ่าน Modbus,SNMP, ETC.

> sudo apt install npm
> 
> sudo npm install -g --unsafe-perm node-red
> 
> npm install node-red/node-red-auth-github
>
> sudo ufw allow 1880
>
- *ตั้งรหัสผ่าน NodeRed*
> 
> sudo node-red admin hash-pw
>
> sudo nano ~/.node-red/settings.js
>
- *Auto Start NodeRed หลังจาก Boot เครื่อง Server*
>
> sudo npm install -g pm2
>
> sudo pm2 start node-red -- -v
>
> sudo pm2 startup
>
> sudo pm2 startup systemd
> 
> sudo pm2 save



#### Command Optional
- *Check Version*
> sudo pm2 info node-red
>
> sudo pm2 logs node-red
> 

***
# Install Nginx
## Web Server, Reverse Proxy
> sudo apt install nginx
>
> sudo systemctl enable nginx
> 
#### Command Optional
- *Operation*
> sudo systemctl start nginx
>
> sudo systemctl stop nginx
>
> sudo systemctl reload nginx
>
> sudo systemctl status nginx
> 

***
# Install PHP

> sudo apt update
>
> sudo apt -y install software-properties-common
>
> sudo add-apt-repository ppa:ondrej/php
>
> sudo apt-get update
>
> sudo apt install php7.4 libapache2-mod-php7.4
>
> sudo apt install php7.4-{cli,common,curl,zip,gd,mysql,xml,mbstring,json,intl,fpm,bcmath,gmp,xmlrpc,dev,imap,opcache,readline,soap}
>
> sudo apt install php8.2 libapache2-mod-php8.2
>
> sudo apt install php8.2-{cli,common,curl,zip,gd,mysql,xml,mbstring,intl,fpm,bcmath,gmp,xmlrpc,dev,imap,opcache,readline,soap}

***
# Install Mariadb
## ฐานข้อมูล
> sudo apt install mariadb-server mariadb-client
>
> sudo systemctl enable mariadb
>
> sudo systemctl start mariadb
>
> sudo systemctl status mariadb
>
> sudo mysql_secure_installation

### apache change port
### เปลี่ยน Port จาก 80 เป็น 8080 และ จาก 443 เป็น 8443
> sudo nano /etc/apache2/ports.conf
> 
> sudo nano /etc/apache2/sites-available/000-default.conf
> 
> sudo service apache2 restart

***
# Install PHPMyadmin
## โปรแกรมจัดการฐานข้อมูล
> sudo add-apt-repository ppa:phpmyadmin/ppa
>
> sudo apt update
>
> sudo apt upgrade -y
>
> sudo apt -y install phpmyadmin
>
> sudo ln -s /usr/share/phpmyadmin/ /var/www/html/phpmyadmin
>
> sudo nano /etc/nginx/snippets/phpmyadmin.conf

# --------------------------
*#Coppy Past This Code*
``````
location /phpmyadmin {
    root /usr/share/;
    index index.php index.html index.htm;
    location ~ ^/phpmyadmin/(.+\.php)$ {
        try_files $uri =404;
        root /usr/share/;
        fastcgi_pass unix:/run/php/php8.2-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include /etc/nginx/fastcgi_params;
    }

    location ~* ^/phpmyadmin/(.+\.(jpg|jpeg|gif|css|png|js|ico|html|xml|txt))$ {
        root /usr/share/;
    }
}
``````
# --------------------------
### กำหนด Limit Concurrent ขอ Data Base
#### กำหนดแบบชั่วคราว
> mysql -u root -p
>> show variables like "max_connections";
>>
>> SET GLOBAL max_connections = 2000;
>>
>> exit;

> sudo nano /etc/my.cnf
>
*#Coppy Past This Code*
### กำหนด Limit Concurrent ขอ Data Base
#### กำหนดแบบถาวร
``````
[mysqld]
max_connections = 2000
``````
# --------------------------

> sudo service mysql restart

# Install Samba

> sudo apt update
>
> sudo apt install samba
>
> sudo service smbd restart
>
> sudo ufw allow samba
>
> sudo adduser connect
>
> sudo smbpasswd -a ubuntu
>
> sudo smbpasswd -a connect
>
> sudo usermod -aG www-data connect
>
> sudo chown www-data:www-data -R /var/www/html
>
> sudo chown root:www-data -R /var/www/html
>
> sudo chmod -R 775 /var/www/html
>
> sudo nano /etc/samba/smb.conf

# --------------------------
*#Coppy Past This Code*
#### ใส่เพิ่มในไฟล์
``````
[www]
path = /var/www/html
available = yes
valid users = @www-data
read only = no
browsable = yes
public = no
writable = yes
create mask = 0775
force user = www-data
force group = www-data
``````
# --------------------------


## NginX Config
sudo nano /etc/nginx/sites-available/default

# --------------------------
*#Coppy Past This Code*
``````
server {
        listen 80 default_server;
        listen [::]:80 default_server;
        root /var/www/html;
        index index.html index.php index.htm index.nginx-debian.html;
        client_max_body_size 100M;
        client_body_buffer_size 100M;

        server_name _;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                #try_files $uri $uri/ =404;
                try_files $uri $uri/ /index.php?$args;
        }
        include snippets/phpmyadmin.conf;
        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/run/php/php7.4-fpm.sock;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                include /etc/nginx/fastcgi_params;
                fastcgi_param   SCRIPT_FILENAME $request_filename;
        }       
        location /node/ {
                 proxy_http_version 1.1;
                 proxy_set_header Upgrade $http_upgrade;
                 proxy_set_header Connection 'upgrade';
                 proxy_set_header Host $host;
                 proxy_cache_bypass $http_upgrade;
                 proxy_pass http://127.0.0.1:1880/;
        }
        location ~ /\.ht {
                deny all;
        }
}
``````
# --------------------------

> sudo nginx -t
>
> sudo systemctl reload nginx
>
> sudo systemctl reload nginx
>
> sudo systemctl status nginx



## SSL

sudo mkdir -p /opt/certs
cd /opt/certs

sudo openssl genrsa -out https.jconnect.local.key 2048
sudo openssl req -new -key https.jconnect.local.key -out https.jconnect.local.csr

sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout https.jconnect.local.key -out https.jconnect.local.crt

sudo nano /etc/nginx/sites-available/default

######
listen 443 ssl default_server;
ssl_certificate /opt/certs/https.jconnect.local.crt;
ssl_certificate_key /opt/certs/https.jconnect.local.key;
######
sudo nginx -t
sudo service nginx reload

