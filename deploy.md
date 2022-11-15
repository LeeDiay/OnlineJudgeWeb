# Deploy Server Online Judger
> Copy từ anh hypnguyen1209

## Step 1: Cài đặt docker

1 dòng duy nhất 😁

```sh
sudo curl -fsSL https://get.docker.com -o get-docker.sh && sh get-docker.sh && sudo usermod -aG docker $USER && sudo reboot
```

## Step 2: Cài đặt docker-compose

Install:

```sh
sudo curl -L "https://github.com/docker/compose/releases/download/2.0.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

Access role:

```sh
sudo chmod +x /usr/local/bin/docker-compose
```

## Step 3: Clone repo

```sh
git clone -b 2.0 https://github.com/QingdaoU/OnlineJudgeDeploy.git && cd OnlineJudgeDeploy
```

## Step 4: Compose up

```sh
sudo docker-compose up -d
```

## Cấu hình port 

Sửa port trong file `docker-compose.yml`

```yml
ports:
    - "0.0.0.0:80:8000"
    - "0.0.0.0:443:1443"
```

Tắt service: 

```sh
sudo docker-compose down
```

## Cấu hình Revese Proxy và Load Balancing

Setup Nginx:

```sh
sudo apt update
sudo apt install nginx
```

Check status: 

```sh
sudo serivce nginx status
```

Gỡ file `default`:

```sh
sudo ln -s /etc/nginx/sites-enabled/default
```

Tạo file `re-proxy.conf`

```sh
sudo vi /etc/nginx/sites-enabled/re-proxy.conf
```

Nội dung

```nginx
server {
	listen 80 default_server;
	listen [::]:80 default_server;
  	root /var/www/html;
	index index.html index.htm index.nginx-debian.html;
	client_max_body_size 125M; # Cấu hình to to 1 tí để đề phòng upload file testcase lớn :v
	server_name contest.vcspassport.com;
	location / {
		proxy_pass http://127.0.0.1:5000;
		proxy_set_header X-Forwarded-Proto $scheme;
		try_files $uri $uri/ /index.html =404;
	}
	location /api/(.*) {
                proxy_pass http://127.0.0.1:5000/api;
		proxy_set_header X-Forwarded-Proto $scheme;
                try_files $uri $uri/ /index.html =404;
        }
        # Caching and filter XSS
        gzip             on;
        gzip_comp_level  3;
        gzip_types       text/html text/plain text/css image/*;
        add_header    X-Content-Type-Options nosniff;
        add_header    X-Frame-Options SAMEORIGIN;
        add_header    X-XSS-Protection "1; mode=block";
}
```

Reload config:

```sh
sudo service nginx restart
```

## Step 5: Sửa frontend

Clone repo:

```sh
git clone https://github.com/QingdaoU/OnlineJudgeFE && cd OnlineJudgeFE
```

Cài Nodejs:

```sh
# Using Ubuntu
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs

# Using Debian, as root
curl -fsSL https://deb.nodesource.com/setup_lts.x | bash -
apt-get install -y nodejs
```

```sh
npm i
# Bind môi trường cho api backend
export TARGET=http://Your-backend
# Run serve dev
npm run dev
```

Sửa favicon:

`/src/pages/oj/index.html`

```html
<link rel="shortcut icon" href="/public/website/favicon.ico">
```

`/src/pages/admin/index.html`

```html
<link rel="shortcut icon" href="/public/website/favicon.ico">
```

Sửa path static file:

`/config/index.js`

```javascript
assetsPublicPath: '/__STATIC_CDN_HOST__/',
```

Thay bằng:

```javascript
assetsPublicPath: '/',
```

Tạo config build:

```sh
npm run build:dll
```

Build:

```sh
npm run build
```

Thư mục dist chức các file đã được build.

Copy thư mục `dist` vào container `oj-backend`

```sh
sudo docker cp dist oj-backend:/app
```
## Backup database

```sh
docker exec -it oj-postgres pg_dumpall -c -U onlinejudge > db_backup_`date +%Y_%m_%d"_"%H_%M_%S`.sql
```

Done!
