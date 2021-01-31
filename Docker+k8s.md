# Docker入门到实践

## Vagrant安装Centos虚拟机

### 安装virtualbox

下载链接：https://www.virtualbox.org/wiki/Downloads 

根据自己需要下载适合自己的操作系统的版本，这里我们安装Mac版本，安装过程略过

### 安装Vagrant

下载链接：https://www.vagrantup.com/downloads

根据自己需要下载适合自己的操作系统的版本，这里我们安装Mac版本，安装过程略过

### 安装centos虚拟机

由于通过vagrant安装centos的过程中，下载会比较慢，所以提前从官方下载centos镜像

下载地址：https://cloud.centos.org/centos/7/vagrant/x86_64/images/CentOS-7-x86_64-Vagrant-2004_01.VirtualBox.box

在Mac终端

```bash
$ mkdir centos
$ cd centos
```

添加本地镜像源

```bash
$ vagrant box add centos7 /Users/xxx/Downloads/CentOS-7-x86_64-Vagrant-2004_01.VirtualBox.box
```

上面的centos7 和路径名根据自己实际需要修改

```bash
$ vagrant init
```

当前路径下会产生一个Vagrantfile文件，vi进行修改

```bash
config.vm.box = "centos7"
```

保存退出后，运行命令进行安装

```bash
$ vagrant up
```

安装完成，变可以在virtualbox看到已经安装好的虚拟机

可以用vagran连接虚拟机

```bash
$ vagrant ssh
```



## 安装Docker

### 删除旧版本

```bash
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

### 安装依赖包

```bash
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2 vim
```

### 添加安装源

```bash
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

或者手工修改安装源

```bash
$	sudo vim /etc/yum.repos.d/docker.repo
```

```bash
[docker]
name=docker
baseurl=https://mirrors.cloud.tencent.com/docker-ce/linux/centos/7/x86_64/stable/
enabled=1
gpgcheck=0
```

### 安装Docker CE

```bash
$ sudo yum install -y docker-ce docker-ce-cli containerd.io
```

### 启动Docker

```bash
$ sudo systemctl start docker
$	sudo systemctl enable docker
```

### 验证Docker

```bash
$ sudo docker version
$ sudo docker run hello-world
```

### 添加国内的registry源

```bash
$	sudo mkdir -p /etc/docker
$	sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://u5w57oct.mirror.aliyuncs.com"]
}
EOF
```

## FQ

### Q1：vagrant up 文件夹挂载的时候，报错 unknown filesystem type 'vboxsf' 处理办法

```bash
$	vagrant plugin install vagrant-vbguest
$	vagrant destroy && vagrant up
```

如果安装特别慢，或者没有响应，是安装源不可访问的缘故，可以做以下操作

```bash
$	gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
```

然后再进行vagrant up命令 或者下面命令直接安装

```bash
$	vagrant plugin install --plugin-clean-sources --plugin-source https://gems.ruby-china.com/ $	vagrant-vbguest
```

安装完vagrant-vbguest，最好虚拟机进行一下update

```bash
$	sudo yum -y update
```

这样用vagrant重新启动虚拟机，加载目录就不会报错了

### Q2：去掉sudo,直接运行docker命令

```bash
$	sudo groupadd docker
$	sudo gpasswd -a vagrant docker
```

然后重启docker服务，重新登录终端即可不用sudo直接运行docker命令

## 获取Docker镜像

### 官方获取

```bash
docker pull hello-world
```

### 自己制作

编写dockfile，然后用docker命理自定义镜像

```bash
docker build -t zhangweibo413/hello-world .
```

镜像基本操作命令

快速删除所有容器

```bash
docker rm $(docker container ls -aq)
docker rm $(docker container ls -f "status=exited" -q)
```

常用命令

```bash
docker commit centos zhangweibo413/centos-vim
docker build -t  zhangweibo413/centos-vim .
docker push zhangweibo413/centos-vim:latest
```

DockerFile语法

FROM scratch

尽量使用官方的镜像

LABEL：Metadata不可少

RUN

反xian'gang

## Docker网络

```bash
brctl show
```

命令需要安装

## 数据持久化存储

### Data Volume

比如mysql数据库的数据存储

1：DockerFile定义VOLUME

```dockerfile
VOLUME ["/var/lib/mysql"]
```

2：然后在运行容易的时加载Volume

```bash
$	sudo docker run -d --name mysql -v mysql:/var/lib/mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql
```

### Bind Mounting

直接挂载宿主机的路径，比如Nginx,容器web主目录加载宿主机的/var/www/html目录

```bash
$	sudo docker run -d -p:80:80 -v /var/www/html:/usr/share/nginx/html --name web nginx
```



## 多容器部署

### 部署wordpress

部署Mysql

```bash
$	sudo docker run -d --name mysql -v $pwd/mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=wordpress mysql
```

无需对外暴露数据库的3306端口

部署wordpress

```bash
$	sudo docker run -d -e WORDPRESS_DB_HOST=mysql:3306 --link mysql -p 8080:80 wordpress
```

### Docker Compose

批处理的角色

docker-compose.yml

Service

一个Service代表一个container,这个容器从dockerhub的image来创建，或者从本地的DockFile build出来的image来创建

可以给service提供network和volume

```yaml
services：
	db：
		image:postgres:9.4
		volumes：
			- "db-data:/var/lib/postgresql/data"
		networks:
			- back-tier
```

​		相当于以下命令

```bash
$	sudo docker run -d --network back-tier -v db-data:/var/lib/postgresql/data postgres:9.4
```

```yaml
version: '3'
services:
  wordpress:
    image: wordpress
    ports:
      - 8080:80
    depends_on:
      - mysql
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_PASSWORD: root
    networks:
      - my-bridge

  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: wordpress
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - my-bridge

volumes:
  mysql-data:

networks:
  my-bridge:
    driver: bridge
```

Network

Volume

Docker Compose安装

```bash
$	sudo curl -L "https://github.com/docker/compose/releases/download/1.28.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$	sudo curl -L https://get.daocloud.io/docker/compose/releases/download/1.28.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
$	sudo chmod +x /usr/local/bin/docker-compose
```

```bash
$	docker-compose up -d
$	docker-compose -f ./docker-compose.yml up -d
```

默认会去寻找docker-compose.yml文件

```bash
$	sudo docker-compose ps
$	sudo docker-compose stop/down
```

### 水平扩展和负载均衡

```bash
$	docker-compose up scale web=3 -d
```

要注意端口重复

haproxy+web+redis负载均衡的例子

docker-compose.yml文件

```yaml
version: "3"
services:
  redis:
    image: redis
  web:
    build:
      context: .
      dockerfile: Dockerfile
      REDIS_HOST: redis
  lb:
    image: dockercloud/haproxy
    links:
      - web
    ports:
      - 8080:80
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock 
```

DockerFile文件

```dockerfile
FROM python:2.7
LABEL maintaner="Zhang Weibo zhangweibo@gmail.com"
COPY . /app
WORKDIR /app
RUN pip install flask redis
EXPOSE 80
CMD [ "python", "app.py" ]
```

app.py文件(web程序)

```python
from flask import Flask
from redis import Redis
import os
import socket
app = Flask(__name__)
redis = Redis(host=os.environ.get('REDIS_HOST', '127.0.0.1'), port=6379)
@app.route('/')
def hello():
    redis.incr('hits')
    return 'Hello Container World! I have been seen %s times and my hostname is %s.\n' % (redis.get('hits'),socket.gethostname())
if __name__ == "__main__":
    app.run(host="0.0.0.0", port=80, debug=True)
```

```bash
docker-compose build
```

提前先build镜像文件，后面就直接up就行

## 容器编排Swarm

### Swarm基本概念

Manager角色

Worker角色

Service

Replicas

### 创建三节点swarm集群

1：先通过vagrant创建3台虚拟机

分别是manager work1 work2

```bash
$	vagrant ssh manager
$	sudo docker swarm init --advertise-addr=192.168.205.10
```

work1节点

```bash
$	vagrant ssh work1
$	sudo docker swarm join --token SWMTKN-1-5sdu2cmyse9knkltem7xaplqbf7g3a5yvjz6gztt821x8jpzl3-f0i4rt7iqabr5ne8dd669frlg 192.168.205.10:2377
```

work2节点

```bash
$	vagrant ssh work2
$	sudo docker swarm join --token SWMTKN-1-5sdu2cmyse9knkltem7xaplqbf7g3a5yvjz6gztt821x8jpzl3-f0i4rt7iqabr5ne8dd669frlg 192.168.205.10:2377
```

在manager节点

```bash
$	vagrant ssh manager
$	sudo docker node ls
```

### Service的创建维护和水平扩展

创建容器

```bash
$	sudo docker service creat --name demo busybox sh -c "while true;do sleep 3600;done"
```


水平扩展

```bash
$	sudo docker service scale demo=5
```

### 通过service部署wordpress

在manager机器上安装overlay网络

```bash
$	sudo docker network create -d overlay demo
```

然后部署Mysql

```bash
$	sudo docker service create --name mysql --env MYSQL_ROOT_PASSWORD=root --env MYSQL_DATABASE=wordpress --network demo --mount type=volume,source=mysql-data,destination=/var/lib/mysql mysql
```

然后部署wordpress

```bash
$	sudo docker service create --name wordpress -p 80:80 --env WORDPRESS_DB_PASSWORD=root --env WORDPRESS_DB_HOST=mysql --network demo wordpress
```

集群服务间通道之Routing Mesh

```bash
$	sudo docker service scale wordpress=3
```

### Ingress Network

### Docker stack 部署wordpress

```yaml
version: '3'

services:

 web:
  image: wordpress
  ports:
   - 8080:80
  depends_on:
   - mysql
  environment:
   WORDPRESS_DB_HOST: mysql
   WORDPRESS_DB_PASSWORD: root
  networks:
   - my-bridge
  deploy:
      mode: replicated
      replicas: 3
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
      update_config:
        parallelism: 1
        delay: 10s
        
 mysql:
  image: mysql:5.7
  environment:
   MYSQL_ROOT_PASSWORD: root
   MYSQL_DATABASE: wordpress
  volumes:
   - mysql-data:/var/lib/mysql
  networks:
   - my-bridge
 	deploy:
      mode: global
      placement:
        constraints:
          - node.role == manager
          
volumes:
 mysql-data:
networks:
 my-bridge:
  driver: overlay
```

```bash
docker stack deploy wordpress --compose-file=docker-compose.yml
docker stack ls
docker stack ps wordpress
docker stack services wordpress
docker stack rm wordpress
```

### Docker Secret管理及使用

创建一个secret

创建一个password文件，就写“admin123”这个密码

```bash
$	sudo docker secret create my-pw password
$	sodu rm -rf password	#建议删除密码文件
```

或者

```bash
$	sodu echo "admin123" | docker secret my-pw -
```

Mysql应用secret

```bash
$	sudo docker service create --name db --secret my-pw -e MYSQL_ROOT_PASSWORD_FILE=/run/secrets/my-pw mysql
```

 我们可以通过参看mysql在哪个work端机器，验证一下mysql的root密码是不是admin123

### Service更新

更新images

```bash
$	sudo docker network create -d overlay demo
$	sudo docker service create --name web --publish 8080:5000 --network demo zhangweibo413/python-flask-demo:1.0
$	sudo docker service scale web=2	#必须2个节点以上
$	sudo docker service ps web	#查看在哪个节点
$	sudo docker service update --image zhangweibo413/python-flask-demo:2.0 web	#更新到2.0
$	sh -c "while true;do curl 127.0.0.1:8080&&sleep 1; done""	#能看出过一段时间后从1.0变成2.0 
```

更新网络端口

```bash
$	sudo udo docker service update --publish rm 8080:5000 --publish add 8088:5000 web
```

## Docker Cloud与Devops

### Docker Cloudzi之自动build

### Docker Cloudzi之持续集成和持续部署

```
master不允许直接修改，Fork到自己账户下
git clone到本地
本地修改
本地测试
上传到github
pull request 测试通过
merge pull request到master分支
docker cloud联动build镜像
docker 联动自动depoly
```

## Kubernetes

### minikube快速搭建k8s单节点环境

常用搭建工具

minikue（单节点）， kubeadmin（集群）， kops，tectonic（Coreos）

安装minikube

```shell
$	curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-latest.x86_64.rpm
$	sudo rpm -ivh minikube-latest.x86_64.rpm
```

安装kubectl

```bash
$	sudo tee /etc/yum.repos.d/kubernetes.repo <<-'EOF'
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
$	sudo yum install -y kubectl
```

```bash
minikube start	#系统要求最小2核心cpu，2G内存
```



### 最小单位POD

共享一个namespace

### RedplicaSet和ReplicationController

### Deployment

### 使用Tectonic在本地单间多节点K8S集群

shell命令补全



### k8s网络基础

### Service

### NodePort类型Service和Label的应用




