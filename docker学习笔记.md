# docker学习笔记

## centos7环境下安装docker

### 卸载旧版本（如果之前安装过）

```bash
yum remove docker  docker-common docker-selinux docker-engine
```

### 安装docker

**手动安装docker**

```bash
yum install -y yum-utils device-mapper-persistent-data lvm2  ##安装需要的软件包依赖
```

```bash
yum-config-manager --add-repo http://download.docker.com/linux/centos/docker-ce.repo ##设置中央仓库
```

```bash
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo ##设置阿里仓库
```

```bash
yum list docker-ce --showduplicates | sort -r  ##查看可以选择的版本
```

```bash
yum -y install docker-ce-18.03.1.ce  ## yum install docker-ce-版本号格式进行安装
```

**使用官方安装脚本全自动安装**

```bash
#第一种方式
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
#第二种方式（国内通道）
curl -sSL https://get.daocloud.io/docker | sh
#两种全自动安装方式任选一种进行安装
```

### docker配置国内源

docker的默认源是：https://hub.docker.com(国外网站，访问速度堪忧)

创建 /etc/docker/daemon.json 文件（如果不存在），然后vim /etc/docker/daemon.json 打开文件，输入：

```bash
{
    "registry-mirrors": [
        "http://hub-mirror.c.163.com", #网易镜像
        "https://docker.mirrors.ustc.edu.cn",#中国科技大学镜像
        "https://registry.docker-cn.com",#docker中国镜像
        "https://reg-mirror.qiniu.com"#七牛云镜像
    ]
}

###！！！该配置文件内容不能有空格字符（可以有回车字符），否则会报错，镜像可以配置多个###
```

加载重启docker

```bash
docker restart docker
```

## docker常用命令

### docker服务管理

#### 启动docker

```bash
systemctl start docker
```

#### 关闭docker

```bash
systemctl stop docker
```

#### 设置开机自动启动

```bash
systemctl enable docker
```

#### 重启docker

```bash
# 注意：一般在调整了防火墙机制之后，需要重新启动docker服务，不然会报iptables failed错误
systemctl restart docker
```

#### 查看docker运行状态

```bash
systemctl status docker
```

### docker基本命令

#### 查看docker版本

```bash
docker version
```

#### 查看docker帮助

```bash
docker --help
docker 命令 --help
```



### docker镜像管理

#### 拉取镜像

```bash
docker pull mysql:5.7 ## docker pull 镜像:版本号 不指定版本号默认为最新版本，从https://hub.docker.com上查找镜像版本
```

快速链接：[官方镜像](https://hub.docker.com)

#### 查看本地镜像

```bash
docker images    ## 查看镜像
docker images -a ## 加参数 -a 则为列出所有镜像
#具体列的含义
REPOSITORY  #镜像仓库源                
TAG         #镜像的标签                 
IMAGE ID    #镜像id            
CREATED     #创建时间             
SIZE        #大小
```

#### 删除镜像

```bash
docker rmi    镜像名/镜像ID
docker rmi -f 镜像名/镜像ID ##强制删除镜像
docker rmi -f 镜像1 镜像2 镜像3 ##同时删除多个镜像
```

#### 打包镜像

```bash
#第一种方式
docker save 镜像名:tag > 路径/文件名.tar
#第二种方式
docker save -o 路径/文件名.tar 镜像名:tag
```

#### 导入镜像

```bash
#第一种方式
docker load < 路径/文件名.tar
#第二种方式
docker load -i 路径/文件名.tar
```

### docker容器管理

#### 查看容器

```bash
#查看正在运行的容器
docker ps
#查看所有容器
docker ps -a
```

#### 查看容器详细信息

```bash
docker inspect 容器名
```

#### 将镜像运行成容器(创建容器)

```bash
docker run -i -t --name 容器名 镜像名称/镜像ID  /bin/bash

#相关参数如下
-t 参数让Docker分配一个伪终端并绑定到容器的标准输入上,/bin/bash
-i 参数则让容器以交互模式打开，通常与-t一起用
-d 创建一个后台运行容器
-v 参数用于挂载一个目录，可以用多个-v参数同时挂载多个目录，格式：宿主机目录:容器目录
--name 容器名称
--restart=always 设置容器随docker服务启动而自动启动
-p 参数用于将容器的端口暴露给宿主机端口 格式：宿主机端口:容器端口
-c 参数用于给运行的容器分配cpu的shares值
-m 参数用于限制为容器的内存信息，以 B、K、M、G 为单位
--net 容器使用的网络
```

####  将容器创建成镜像

精简版本的linux可能没有vim、ifconfig这些常用命令，这时候我们可以在我们的精简版本上运行yum -y install vim 指令安装好vim后，再将这个精简版本的linux创建成镜像，那么这个新的镜像里面就包含有vim命令了。

```bash
docker commit 容器名/容器ID 新镜像:新镜像的tag

#参数如下
-a 表示镜像的作者
-c 表示使用Dockerfile指令创建镜像   
-m 提交时的说明文字
-p 在commit时，将容器暂停
```

#### 启动已停止的容器

```bash
docker start 容器名称/容器ID
```

#### 重启容器

```bash
docker restart 容器名称/容器ID
```

#### 停止容器

```bash
docker stop 容器名称/容器ID
docker kill 容器名称/容器ID
```

#### 进入运行中的容器

```bash
docker exec -it 容器名称/容器ID /bin/bash

#参数说明如下
-d 参数表示后台运行
-i 参数让容器以交互模式打开
-t 分配一个伪终端tty给容器

docker attach 容器名/容器ID  #使用这个命令进入容器，exit时，容器会退出运行，而exec命令启动则不会
```

#### 从容器中退出

```bash
exit      attach方式进入有效，exec方式进入无效
CTRL + D
CTRL + P CTRL + Q
```

#### 查看容器内运行的进程

```bash
docker top 容器名称/容器ID
```

#### 从容器拷贝数据到宿主机

```bash
docker cp 容器名:容器内文件路径  宿主机路径
```

#### 从宿主机拷贝数据到容器

```bash
docker cp 宿主机路径  容器名:容器内文件路径
```

#### 查看容器日志

```bash
docker logs 容器名/容器ID 
#参数说明如下
-f : 跟踪日志输出
--since :显示某个开始时间的所有日志
-t : 显示时间戳
--tail :仅列出最新N条容器日志

例如：
#查看redis容器日志，参数：-f  跟踪日志输出；-t   显示时间戳；--tail  仅列出最新N条容器日志
docker logs -f -t --tail=20 redis  
#查看容器redis从2020年06月01日后的最新10条日志
docker logs --since="2020-06-01" --tail=10 redis
```

## docker安装软件

### docker安装tomcat

```bash
#拉取tomcat镜像
docker pull tomcat:8.0.52
#运行tomcat镜像(未考虑数据挂载)
docker run -it -p 8080:8080 --name 容器名 tomcat:8.0.52
#交互模式进入tomcat容器
docker exec -it 容器名 /bin/bash 
#如果是高版本的tomcat，容器的/usr/local/tomcat目录下的webapps是空的，将webapps.dist修改为webapps替换即可
```

### docker安装mysql

```bash
#拉取mysql镜像
docker pull mysql:5.7
#运行mysql镜像(考虑端口映射，初始密码，数据挂载(数据、日志、配置文件))
docker run -d -p 3306:3306 --privileged=true -e MYSQL_ROOT_PASSWORD=数据库root用户密码 -v /mysqldata/mysql/log:/var/log/mysql  -v /mysqldata/mysql/data:/var/lib/mysql -v /mysqldata/mysql/conf:/etc/mysql/conf.d --name 容器名 mysql:5.7
#交互模式进入mysql容器
docker exec -it 容器名 /bin/bash
#用root用户登录数据库
mysql -uroot -p
#查看字符编码发现编码格式为latin1，会导致我们插入或查询数据出错或乱码
show variables like 'character%';
#设置编码统一
#在宿主机目录下（或者容器内）增加配置文件 vim /mysqldata/mysql/conf/my.cnf，然后文件中输入：
[client]
default_character_set=utf8
[mysqld]
collation_server=utf8_general_ci
character_set_server=utf8
#运行sql文件(注意要设置好编码后再运行sql文件)
source 数据库文件.sql
#mysql允许外部访问授权
grant all privileges on *.* to root@'%' identified by 'mysql用户密码' with grant option;
flush privileges;
```

### docker安装nginx

```bash
#拉取nginx镜像
docker pull nginx
#宿主机创建挂载目录（配置文件、html、日志文件、配置）
mkdir -p /opt/docker/nginx/conf.d 
mkdir -p /opt/docker/nginx/html
mkdir -p /opt/docker/nginx/logs
mkdir -p /opt/docker/nginx/conf/nginx.conf
#运行nginx镜像
docker run -d -p 80:80 --name 容器名 --restart=always 
-v /opt/docker/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /opt/docker/nginx/conf.d:/etc/nginx/conf.d \
-v /opt/docker/nginx/html:/usr/share/nginx/html \ 
-v /opt/docker/nginx/logs:/var/log/nginx \ 
nginx
```

### docker安装redis

```bash
#拉取redis镜像
docker pull redis
#运行redis镜像
docker run -it -p 6379:6379 --name 容器名 redis
```

### docker安装elasticsearch

```bash
#拉取elasticsearch镜像
docker pull elasticsearch:7.17.6
#运行elasticsearch镜像(ES_JAVA_OPTS指定elasticsearch内存最大使用512M)
docker run --name elasticsearch -d -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -e "discovery.type=single-node" -p 9200:9200 -p 9300:9300 elasticsearch:7.17.6
```

## Dockerfile

Dockerfile 是一个文本文件，包含了一些命令和参数，用来构建(制作)一个镜像，每一条指令构建一层镜像，用来描述该层镜像应当如何构建。Dockerfile 让我们可以构建自己的镜像，满足我们生产需要，比如我们可以通过Dockerfile把我们的Java Web项目构建成一个镜像然后运行在docker环境中。

### Dockerfile常用指令

```bash
#指定基础镜像（理解为母版），每一个Dockerfile文件第一行必须是FROM指令
FROM <image>:<tag> 
#指定镜像维护者的名字
MAINTAINER <作者名字>
#镜像在构建时执行的shell命令，如RUN yum -y install vim
RUN  <command>
#将宿主机上的文件复制到镜像
ADD  <源文件> <目标文件>
#COPY 作用与ADD一样
COPY <源文件> <目标文件>
#设置环境变量,如ENV JAVA_HOME /usr/local/jre
ENV  <变量> <变量值>
#为镜像运行后的容器指定一个默认落脚点，如WORKDIR /usr/local，进入容器后直接进入这个目录
WORKDIR 容器路径
# 用来创建挂载数据卷
VOLUME  容器路径
#对外暴露一个端口
EXPOSE <端口号>
#容器运行时执行的命令，即执行docker run 命令后容器要执行的命令(与ENTRYPOINT作用一致)，如果docker run 后面传递了参数，则会覆盖CMD的命令
CMD ["可执行命令","参数1","参数2"]
#与CMD作用类似，但是当CMD和ENTRYPOINT都存在时，CMD里面的值将作为ENTRYPOINT的参数，ENTRYPOINT级别更高一点
ENTRYPOINT ["可执行命令","参数1","参数2"]
```

### Dockerfile构建镜像命令

```bash
#其中的 . 是Dockerfile这个文件的路径，.表示Dockerfile在当前路径下
docker build -t 镜像名:tag .
```

## Docker-compose

### docker-compose安装

```bash
# 1、下载docker-compose
curl -L https://github.com/docker/compose/releases/download/1.16.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
# 2、为安装脚本添加执行权限
chmod +x /usr/local/bin/docker-compose
# 3、查看docker-compose版本
docker-compose --version
```

### docker-compose.yml配置文件编写

yml文件解析

```yaml
version: "3" #定义docker-compose的版本

services:    #定义服务集合
 
 bbs: #定义服务
  build: . #image来源，build . 表示使用当前目录的Dockerfile文件进行构建image
  ports: #定义该容器的端口映射，可以有多个
   - 8080:8080
  networks: #定义容器的网络
   - my_network
  container_name: my-bbs #定义容器的名字
  restart: always #定义docker启动时，容器自动启动
  depends_on: #定义当前服务依赖的服务，需要下面的先启动后才能启动
   - mysql 
  command: echo "hello world" #容器启动后覆盖默认执行命令
  
 mysql: #定义服务
  image: mysql:5.7 #定义该服务容器的镜像
  ports: #定义该容器的端口映射，可以有多个
   - 3306:3306
  networks: #定义容器的网络
   - my_network
  volumes:  #定义数据卷映射
   - /mysqldata/mysql/log:/var/log/mysql
   - /mysqldata/mysql/data:/var/lib/mysql
   - /mysqldata/mysql/conf:/etc/mysql/conf.d
  container_name: mysql57 #定义容器的名字
  restart: always #定义docker启动时，容器自动启动
  environment:  #定义容器启动环境配置
   MYSQL_ROOT_PASSWORD: 123456
   MYSQL_DATABASE: test
   MYSQL_USER: zhang
   
networks: #定义docker网络,默认为bridge模式
 my_network: #定义网络名称
  driver: bridge #指定网络类型为桥接模式
  
volumes:
```

### 用docker-compose一键部署多个容器

在docker-compose.yml配置文件同一目录下，执行docker-compose启动命令

```bash
docker-compose -d up
```

### docker-compose常用命令

```bash
#以下命令需要进入docker-compose.yml配置文件所在目录
#启动容器并后台运行
docker-compose -d up
#停止并删除容器、网络、卷、镜像
docker-compose down
#重启容器
docker-compose restart
#查看日志
docker-compose logs
#查看运行的容器
docker-compose ps
```

