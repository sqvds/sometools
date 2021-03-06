# Record how to build a basic image for nginx + php-fpm


## 0x01 php-fpm
pull images from hub
```
docker pull php:7.1-fpm
```
change the tag to get different version


## 0x02 nginx
pull nginx from 163hub
```
docker pull hub.c.163.com/library/nginx:latest
```

## 0x03 mysql(Not use this times)
pull mysql from 163hub
```
docker pull hub.c.163.com/library/mysql:5.6.36
```

## 0x04 Start a container for php-fpm
```
docker run --name php-fpm-7.1 -d -v ~/study/dockerfiledir:/var/www/html:ro php:7.1-fpm
```
1. --name 指定容器的名字
2. -d 后台运行
3. -v 把主机的~/study/dockerfiledir目录映射到容器的/var/www/html目录下 ro表示只读 
4. php:7.1-fpm表示来源镜像
ps: ~/study/dockerfiledir/是放在主机的web目录 

## 0x05 Start a container for nginx
```
docker run --name nginx_server -d -p 80:80 -v ~/study/dockerfiledir:/usr/share/nginx/html:ro -v ~/study/containerconfig/nginx/conf.d:/etc/nginx/conf.d:ro --link php-fpm-7.1:php hub.c.163.com/library/nginx
```
1. -p 80:80 端口映射，把所创建的容器的80端口绑定到宿主机80端口
2. -d 后台运行
3. -v 目录映射
4. --link 建立容器间连接关系，把php-fpm-7.1容器的信息绑定到所创建容器的环境变量php中，则通过php这个别名即可访问到容器php-fpm-7.1
ps:~/study/containerconfig/nginx/conf.d/是放在本机的nginx配置文件

## 0x06 link mysql
如果是再加一个mysql容器该怎么操作？
在创建php-fpm容器时需要与mysql容器建立连接
```
docker run --name server_mysql -d hub.c.163.com/library/mysql:5.6.36
docker run --name php-fpm-7.1 -d -v ~/study/dockerfiledir:/var/www/html:ro --link server_mysql php:7.1-fpm
```
ps: mysql镜像默认没有密码，但是限制非本机的连接，所以要自己付一下权限
GRANT ALL PRIVILEGES ON *.* TO 'root'@'172.16.0.2' IDENTIFIED BY 'password' WITH OPTION;

## 0x07 官方镜像没有php一些必备扩展怎么办？
官方的php-fpm docker镜像中在
/usr/local/bin目录下有几个脚本用于管理扩展
如docker-php-ext-install即可用来安装php扩展
docker-php-ext-enable程序可以用来控制扩展的启动和禁用
至此完成了nginx + php-fpm容器的创建，在docker官方文档中建议一个容器只运行一个应用，但是docker并没有限制对于一个容器运行多个应用的情况，之前一直在同一个容器中安装多个应用的情况，此次进行一下基本开发镜像的配置，方便以后操作

为了方便对于多个容器互连的管理，可以使用开源docker项目管理工具docker-compose



## 0x08 补充
补充了compose文件，docker-compose非常方便，yel文件的也很直观
在补充一些小细节
php-fpm默认没有安装一些扩展，上面提到过，众所周知，安装扩展后需要重启服务，
php 5.6后， service systemctl找不到php-fpm服务了，是因为缺少pid文件导致，手动建立太麻烦
其实php-fpm有信号机制，可以通过信号控制
kill -INT pid 关闭
kill -USR2 pid 平滑重启并重新载入配置和二进制文件
kill -USR1 重新打开日志文件

