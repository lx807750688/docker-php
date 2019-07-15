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
      docker pull mysql:5.7
      
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
    
   - 查看php-fpm的扩展情况缺少可以直接进入容器安装
   
     ```
     docker exec -it 484b8c4a3203 /bin/bash
     cd  /usr/local/bin/
     ./docker-php-ext-install pdo_mysql
     ./docker-php-ext-install mysqli
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
   **3. 部署mysql容器**
   - 在docker-php目录下新建mysql日志文件配置文件和数据文件
     ```
     mkdir -p  mysql/logs mysql/data mysql/conf
     ```
   - 运行mysql容器
     ```
     docker run -p 3307:3306 --name mysql -v /docker-php/mysql/conf:/etc/mysql/conf.d -v /docker-php/mysql/logs:/logs -v /docker-php/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=000000 -d --privileged=true  docker.io/mysql:5.7
     ```
   - 新建index.php测试能否连通数据库
     ```
     先进入mysql容器创建测试数据库 test
     [root@localhost docker-php]# vi nginx/www/index.php
     <?php
     //die(phpinfo());
     $mysqli = mysqli_connect('172.17.0.4','root','000000','test');
     $conn = new PDO('mysql:host=172.17.0.4;dbname=test;port=3306','root','000000');
     var_dump($conn,$mysqli);
     ?>
     ```
   - 测试成功
     ```
     object(PDO)#2 (0) { } object(mysqli)#1 (19) { ["affected_rows"]=> int(0) ["client_info"]=> string(79) "mysqlnd 5.0.12-dev - 20150407 - $Id: 3591daad22de08524295e1bd073aceeff11e6579 $" ["client_version"]=> int(50012) ["connect_errno"]=> int(0) ["connect_error"]=> NULL ["errno"]=> int(0) ["error"]=> string(0) "" ["error_list"]=> array(0) { } ["field_count"]=> int(0) ["host_info"]=> string(21) "172.17.0.2 via TCP/IP" ["info"]=> NULL ["insert_id"]=> int(0) ["server_info"]=> string(6) "5.7.26" ["server_version"]=> int(50726) ["stat"]=> string(132) "Uptime: 2512 Threads: 2 Questions: 4 Slow queries: 0 Opens: 105 Flush tables: 1 Open tables: 98 Queries per second avg: 0.001" ["sqlstate"]=> string(5) "00000" ["protocol_version"]=> int(10) ["thread_id"]=> int(3) ["warning_count"]=> int(0) }
     ```
        
