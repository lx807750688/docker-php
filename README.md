## 安装

  **1. docker安装**
     yum install -y docker  //配置好yum源可以选择阿里云yum源
  
  **2. 测试docker是否安装成功**
      
       [root@localhost nginx]# docker version
        Client:
        Version:         1.13.1
        API version:     1.26
        Package version: docker-1.13.1-96.gitb2f74b2.el7.centos.x86_64
        Go version:      go1.10.3
        Git commit:      b2f74b2/1.13.1
        Built:           Wed May  1 14:55:20 2019
        OS/Arch:         linux/amd64

        Server:
        Version:         1.13.1
        API version:     1.26 (minimum version 1.12)
        Package version: docker-1.13.1-96.gitb2f74b2.el7.centos.x86_64
        Go version:      go1.10.3
        Git commit:      b2f74b2/1.13.1
        Built:           Wed May  1 14:55:20 2019
        OS/Arch:         linux/amd64
        Experimental:    false
        
     
## 下拉镜像

   **1. 运行docker**

      systemctl start docker
       
      
   **2. 下拉php环境的docker镜像**
   
      docker pull nginx
      docker pull php:7.2-fpm
      docker pull mysql
      
   **3. 查看镜像docker镜像**
   
       [root@localhost nginx]# docker images
       REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
       docker.io/php       7.3-fpm             ff76bea6e276        35 hours ago        371 MB
       docker.io/nginx     latest              f68d6e55e065        9 days ago          109 MB
       docker.io/mysql     latest              c7109f74d339        4 weeks ago         443 MB
       
## 部署php环境

   **1. 部署nginx**
   - 创建nginx所需的映射文件夹
   
      ```  
      mkdir -p /docker-php/nginx/www /docker-php/nginx/logs /docker-php/nginx/conf/nginx.conf
      ```   
      
   - 启动nginx容器
   
     ```
     docker run -d -p 8080:80 --name nginx-web -v /docker-php/nginx/www/:/usr/share/nginx/html -v /docker-php/nginx/conf/nginx.conf/:/etc/nginx/nginx.conf -v /docker-php/nginx/logs/:/var/log/nginx --privileged=true docker.io/nginx
     ``` 
         
   - 命令说明
       
      ```
      -p 8080:80： 将容器的 80 端口映射到主机的 8080 端口

      --name nginx-web ：将容器命名为 nginx-web 

      -v /docker-php/nginx/www/:/usr/share/nginx/html：将我们自己创建的 www 目录挂载到容器的 /usr/share/nginx/html。

      --privileged=true 给容器加上执行权限
      ```
          
   - 写一个基本的nginx.conf
     
     ```
     user  nginx;
     worker_processes  1;
     error_log  /var/log/nginx/error.log warn;
     pid        /var/run/nginx.pid;
     events {
         worker_connections  1024;
     }
     http {
         include       /etc/nginx/mime.types;
         default_type  application/octet-stream;
         log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                           '$status $body_bytes_sent "$http_referer" '
                           '"$http_user_agent" "$http_x_forwarded_for"';
         access_log  /var/log/nginx/access.log  main;
         sendfile        on;
         #tcp_nopush     on;
         keepalive_timeout  65;
         #gzip  on;
         include /etc/nginx/conf.d/*.conf;
     }
     ```
            
   - 在/docker-php/nginx/www/目录下新建一个基本的index.html页面
   
     ```
     <!DOCTYPE html>
     <html>
     <head>
     <meta charset="utf-8">
     <title>test</title>
     </head>
     <body>
         <h1>TEST</h1>
     </body>
     </html>
     ```
         
   - 测试能否运行成功
      
     ```
     [root@localhost nginx]# curl 127.0.0.1:8080
     <!DOCTYPE html>
     <html>
     <head>
     <meta charset="utf-8">
     <title>test</title>
     </head>
     <body>
         <h1>TEST</h1>
     </body>
     </html>
     ```
         
   **2. 部署php-fpm容器**

   - 启动容器
   
      ```
      docker run -d  -p 9000:9000 --name php-fpm -v /docker-php/nginx/www:/www --privileged=true docker.io/php:7.2-fpm  
      ```   
    
   - 添加php-fpm容器和nginx容器映射
    
     先查看俩个容器的ip地址之后利用 echo 'IP+容器id' >> /etc/hoots
    
      ```
      docker inspect 72433982d5c3 |grep IPAddress
      echo "172.17.0.3 72433982d5c3" >> /etc/hosts 
      ```
    
  - 在nginx配置文件中添加
   
     ```
     server {
     listen       80;
     server_name  localhost;
     root   /usr/share/nginx/html;
     index  index.html index.htm index.php;

     location / {
         root   /usr/share/nginx/html;
         index  index.html index.htm index.php;
     }

     error_page   500 502 503 504  /50x.html;
     location = /50x.html {
         root   /usr/share/nginx/html;
     }

     location ~ \.php$ {
         fastcgi_pass   172.17.0.3:9000; //为php-fpm容器的IP地址
         fastcgi_index  index.php;
         fastcgi_param  SCRIPT_FILENAME  /www/$fastcgi_script_name;
         include        fastcgi_params;
     }
     }
    ```
    
   - 重启nginx容器并新建/docker-php/nginx/www/index.php
      
      ```
      [root@localhost nginx]# cat /docker-php/nginx/www/index.php 
      <?php
      echo phpinfo();
      ?>
      ```
   - 访问index.php
     ```
     [root@localhost nginx]# curl  127.0.0.1:8080/index.php
     出现大量html代码成功
     ```
     
     
   
    
    
        
