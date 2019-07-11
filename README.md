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
3. 下拉镜像   
   运行systemctl start docker 成功后分别下拉nginx镜像php-fpm镜像和mysql镜像
   
   docker pull nginx
   
   docker pull php-fpm
   
   docker pull mysql
   
