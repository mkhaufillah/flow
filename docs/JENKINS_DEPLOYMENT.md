# JENKINS DEPLOYMENT

### Change Server to Root Mode for Simplify the Command

Apply this command.

```sh
sudo su
cd
apt update
apt upgrade
```

### Download JDK and Jenkins (latest version)

Apply this command.

```sh
wget https://aka.ms/download-jdk/microsoft-jdk-21.0.3-linux-x64.tar.gz
wget https://get.jenkins.io/war-stable/2.440.3/jenkins.war
```

### Install JDK and Jenkins (latest version)

Apply this command.

```sh
tar -xvzf microsoft-jdk-21.0.3-linux-x64.tar.gz
rm -rf microsoft-jdk-21.0.3-linux-x64.tar.gz
mv jdk-21.0.3+9/ jdk21
echo 'export JAVA_HOME="/root/jdk21"' >> .bashrc
echo 'export PATH="$PATH:$JAVA_HOME/bin"' >> .bashrc
source .bashrc
```

### Setup Jenkins

Create Jenkins server configuration file with `vim /lib/systemd/system/jenkins-java.service` with this.

```
[Unit]
Description=Jenkins
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/root
ExecStart=/root/jdk21/bin/java -jar jenkins.war --httpPort=8080
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Then apply this command

```sh
systemctl daemon-reload
systemctl start jenkins-java
systemctl enable jenkins-java
systemctl status jenkins-java
apt install xvfb
apt install fontconfig
```

### Setup NGINX

```sh
apt install nginx
rm -rf /etc/nginx/sites-enabled/default
rm -rf /etc/nginx/sites-available/default
rm -rf /var/www/html/index.nginx-debian.html
```

Create Jenkins NGINX configuration file with `vim /etc/nginx/sites-available/jenkins.filla.id` with this.

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    server_name jenkins.filla.id;

    location / {
        limit_req zone=ip burst=12 delay=8;
        limit_req_status 444;
        limit_req_log_level warn;
        proxy_pass http://localhost:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host:$server_port;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Port $server_port;
    }
}
```

Create Jenkins NGINX configuration file with `vim /etc/nginx/sites-available/flow.filla.id` with this.

```
server {
    listen 80;
    listen [::]:80;

    server_name flow.filla.id;

    location / {
        limit_req zone=ip burst=12 delay=8;
        limit_req_status 444;
        limit_req_log_level warn;
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host:$server_port;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Port $server_port;
    }
}
```

Then apply this command

```sh
ln -s /etc/nginx/sites-available/jenkins.filla.id /etc/nginx/sites-enabled/jenkins.filla.id
ln -s /etc/nginx/sites-available/flow.filla.id /etc/nginx/sites-enabled/flow.filla.id
nginx -t
systemctl restart nginx
systemctl reload nginx
systemctl enable nginx
```

### Setup SSL

Apply this command

```sh
snap install --classic certbot
ln -s /snap/bin/certbot /usr/bin/certbot
certbot --nginx -d jenkins.filla.id -d flow.filla.id
systemctl status snap.certbot.renew.service
nginx -t
systemctl restart nginx
systemctl reload nginx
systemctl restart jenkins-java
```

### Setup Jenkins UI

1. Pointing the domain jenkins.filla.id and flow.filla.id dns to server jenkins
2. Access https://jenkins.filla.id
3. Follow the instructions
4. Enable proxy compability in https://jenkins.filla.id/manage/configureSecurity/

### Generate SSH

Apply this command

```sh
ssh-keygen -t ed25519 -C "khaufillahmohammad@gmail.com"
cat .ssh/id_ed25519.pub
```

This SSH usefull for access repository in jenkins, please store it in github/gitlab/etc