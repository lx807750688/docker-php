## 安装

  1. docker安装
     yum install -y docker  //配置好yum源可以选择阿里云yum源
  
  2. 测试docker是否安装成功
     ```
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
      ```
     
## 下拉镜像

   1. 运行docker
      ```
      systemctl start docker
      ``` 
   2. 下拉php环境的docker镜像
      ```
      docker pull nginx
      docker pull php:7.3-fpm
      docker pull mysql
      ``` 
   3. 查看镜像docker镜像
       ```
       [root@localhost nginx]# docker images
       REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
       docker.io/php       7.3-fpm             ff76bea6e276        35 hours ago        371 MB
       docker.io/nginx     latest              f68d6e55e065        9 days ago          109 MB
       docker.io/mysql     latest              c7109f74d339        4 weeks ago         443 MB
       ```
## 部署php环境
   1. 部署nginx
   
      创建nginx所需的映射文件夹
      ```
       mkdir -p /docker-php/nginx/www /docker-php/nginx/logs /docker-php/nginx/conf/nginx.conf
      ```
      
      启动nginx容器
      ```
      docker run -d -p 8080:80 --name nginx-web -v /docker-php/nginx/www/:/usr/share/nginx/html -v /docker-php/nginx/conf/nginx.conf/:/etc/nginx/nginx.conf -v /docker-php/nginx/logs/:/var/log/nginx --privileged=true docker.io/nginx
      ```
      命令说明
      
      -p 8080:80： 将容器的 80 端口映射到主机的 8080 端口
      
      --name nginx-web ：将容器命名为 nginx-web 
      
      -v /docker-php/nginx/www/:/usr/share/nginx/html：将我们自己创建的 www 目录挂载到容器的 /usr/share/nginx/html。
     
      --privileged=true 给容器加上执行权限
      
      
      
