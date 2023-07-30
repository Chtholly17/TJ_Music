# TJ_Music
Group project for Software Engineering course, Tongji University.
# 项目部署
### 后端部署
1. clone本项目，进入backend submodule所在的文件夹
2. 在根目录执行
  ```bash
  $ mvn clean package -Dmaven.test.skip=true
  ```
  或者您也可以使用IDEA打开项目，并且按照如下步骤操作
  ![image](https://github.com/Chtholly17/TJ_Music/assets/82920537/3b17d0af-c6af-4bb7-bbae-66dbd6abd936)
3. 之后，在执行上述命令后在`./Tj_music_work/target`中，可以看到打包好的项目包
    在根目录执行以下命令，将包传到指定的服务器路径处
  ```bash
  $ scp ./target/tj_music-0.0.1-SNAPSHOT.jar root@120.46.60.40:/root/TJ_music
  ```
4. 使用ssh登陆服务器，执行下列命令安装jdk
  ```bash
  $ yum install java-1.8.0-openjdk* -y
  ```
5. 在服务器`/root/TJ_music`下执行命令
  ```bash
  $ nohup java -cp static -jar tj_music-0.0.1-SNAPSHOT.jar > output.log &
  ```
  后端服务器部署完成。
### Clash部署
由于我们的项目需要使用open AI的api，因此可能需要一些技术手段，此处以clash为例：
首先需要下载clash：
```bash
wget https://github.com/Dreamacro/clash/releases/download/v1.11.4/clash-linux-amd64-v1.11.4.gz
```
解压缩：
```bash
gzip -d ./clash-linux-amd64-v1.11.4.gz
```
赋权重：
```bash
chmod u+x clash
```
下载country db:
```bash
cd /root/.config/clash
wget https://github.com/Dreamacro/maxmind-
geoip/releases/download/20220612/Country.mmdb
```
上传配置文件：
```bash
https://api.meaqua.fun/sub?target=clash&url=你的订阅链接或vmess
```
启动clash，并将其挂在后台 在服务器中访问网页对clash进行配置：
```bash
https://clash.razord.top/
```
选择新加坡节点，开启tun模式 即可正常访问open AI了
### python环境部署
1. 在服务器上安装Miniconda
2. 将`music.yml`和`requirements.txt`上传到服务器，并分别执行命令
  ```bash
  $ conda env create -f music.yml 
  $ conda activate music 
  $ pip install -r requirements.txt
  ```
3. 将gitcode和gpt_call两个文件夹上传至服务器
4. 最后的服务器结构如下：
![image](https://github.com/Chtholly17/TJ_Music/assets/82920537/d1f16161-961a-468d-8625-49c261f734d3)
### 前端环境配置
1. 在服务器上安装nginx，注意centos和ubuntu安装方法不同，这里只说明centos的安装方法
```bash
yum install -y nginx
```
如果未配置yum源的，可参考网上如何使用yum安装nginx。 
2. 进入/etc/nginx文件夹，修改nginx.conf文件
```bash
cd /etc/nginx vi nginx.conf
```
参考配置文件如下所示
```bash
# For more information on configuration, see:
# * Official English Documentation: http://nginx.org/en/docs/ # * Official Russian Documentation: http://nginx.org/ru/docs/ user root; worker_processes auto; error_log /var/log/nginx/error.log; pid /run/nginx.pid;
# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic. include /usr/share/nginx/modules/*.conf;
events { worker_connections 1024; }
http { log_format
access_log
main
'$remote_addr - $remote_user [$time_local] "$request" ' '$status $body_bytes_sent "$http_referer" ' '"$http_user_agent" "$http_x_forwarded_for"';
/var/log/nginx/access.log
main;
sendfile on; tcp_nopush on; tcp_nodelay on; keepalive_timeout 65; types_hash_max_size 2048;
include /etc/nginx/mime.types; default_type application/octet-stream; client_max_body_size 200M;
# Load modular configuration files from the /etc/nginx/conf.d directory. # See http://nginx.org/en/docs/ngx_core_module.html#include # for more information.
include /etc/nginx/conf.d/*.conf;
server {
listen listen server_name root
8080 default_server; [::]:8080 default_server; _; /root/TJ_music/dist;
# Load configuration files for the default server block. include /etc/nginx/default.d/*.conf;
location / { try_files $uri $uri/ /index.html; }
#加入下面代码 location @405 {
proxy_set_header Host $host; proxy_set_header X-Real-IP $remote_addr; proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; #ip为后端服务地址 proxy_pass http://120.46.60.40:8888$request_uri ;
} # 配置与后端服务器地址的映射 location ^~ /api/ { proxy_pass http://120.46.60.40:8888/;
error_page 405 =200 @405; #405页面处理
}
error_page 404 /404.html; location = /40x.html { }
error_page 500 502 503 504 /50x.html; location = /50x.html { }
}
# Settings for a TLS enabled server.
# # server { # listen 443 ssl http2 default_server; # listen [::]:443 ssl http2 default_server; # server_name _; # root /usr/share/nginx/html; # # ssl_certificate "/etc/pki/nginx/server.crt"; # ssl_certificate_key "/etc/pki/nginx/private/server.key"; # ssl_session_cache shared:SSL:1m; # ssl_session_timeout 10m; # ssl_ciphers PROFILE=SYSTEM; # ssl_prefer_server_ciphers on; # # # Load configuration files for the default server block. # include /etc/nginx/default.d/*.conf; # # location / { # } # # error_page 404 /404.html; # location = /40x.html { # } # # error_page 500 502 503 504 /50x.html; # location = /50x.html { # } # }
}
```
4. 本地打包前端文件，将dist文件夹通过scp指令传输到/root/TJ_music/dist文件夹中
```bash
npm run build scp -r ./dist/ root@120.46.60.40:/root/TJ_music
```
5. 设置安全组，前端端口是8080，后端是8888
![image](https://github.com/Chtholly17/TJ_Music/assets/82920537/c63dc7ba-d2b2-48ac-99d7-50317d80c5f9)
![image](https://github.com/Chtholly17/TJ_Music/assets/82920537/fe5297cb-f8d7-4aaf-99c2-51dab863775f)
6. 可以在`http://120.46.60.40:8080`查看部署好的网站

JL
