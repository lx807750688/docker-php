## 安装

  1. docker安装
  yum install -y docker  //配置好yum源可以选择阿里云yum源
  
  2. 测试docker是否成功安装并且运行docker
     测试docker version 
     ```
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
     运行systemctl start docker 成功后分别下拉nginx镜像php-fpm镜像和mysql镜像
     docker pull nginx
     docker pull php:7.3-fpm
     docker pull mysql

     查看镜像docker镜像 docker images
     [root@localhost nginx]# docker images
     REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
     docker.io/php       7.3-fpm             ff76bea6e276        35 hours ago        371 MB
     docker.io/nginx     latest              f68d6e55e065        9 days ago          109 MB
     docker.io/mysql     latest              c7109f74d339        4 weeks ago         443 MB
## 部署php环境
  1. 部署nginx 
