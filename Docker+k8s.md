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

### 安装k8s

常用搭建工具

minikue（单节点）， kubeadmin（集群）， kops，tectonic（Coreos）

### minikube快速搭建k8s单节点环境

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

也可以指定国内镜像启动

```bash
minikube start --registry-mirror=https://u5w57oct.mirror.aliyuncs.com
```

```bash
minikube dashboard
```

### 使用kubeadmin搭建三个节点的k8s集群

准备三台机器

masrer	192.168.205.10

node1	192.168.205.11

node2	192.168.205.12

要求CPU至少2核以上，内存至少2G以上

允许 iptables 检查桥接流量

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
```

安装 kubeadm、kubelet 和 kubectl

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF
```

将 SELinux 设置为 permissive 模式（相当于将其禁用）

```bash
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

安装

```bash
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable --now kubelet
```

```bash
sudo systemctl stop firewalld
sudo systemctl disable firewalld
sudo swapoff -a
```

```bash
sudo systemctl enable docker.service
sudo systemctl enable kubelet.service
```

master安装成功

```bash
sudo kubeadm init --pod-network-cidr 172.100.0.0/16 --apiserver-advertise-address 192.168.205.10
```


master安装成功后的提示

```bash
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.205.10:6443 --token s1kd51.1ueds2t342qmv0m6 \
    --discovery-token-ca-cert-hash sha256:20df1826136cb3c8b2bcfbd502f3b140b69d9ba157b4821d17d0b762465969c5
```

然后在master节点上运行

```bash
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

检查pod

```bash
[vagrant@k8s-master ~]$ kubectl get pod --all-namespaces
NAMESPACE     NAME                                 READY   STATUS    RESTARTS   AGE
kube-system   coredns-5c98db65d4-f4kjf             0/1     Pending   0          58m
kube-system   coredns-5c98db65d4-xqpwd             0/1     Pending   0          58m
kube-system   etcd-k8s-master                      1/1     Running   0          57m
kube-system   kube-apiserver-k8s-master            1/1     Running   0          57m
kube-system   kube-controller-manager-k8s-master   1/1     Running   0          57m
kube-system   kube-proxy-9l9vr                     1/1     Running   0          58m
kube-system   kube-scheduler-k8s-master            1/1     Running   0          57m
[vagrant@k8s-master ~]$
```

安装网络插件

```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

### kubectl基本使用

命令自动补全

```bash
source <(kubectl completion zsh)	#tab键，或者字母自动补全
source <(kubectl completion bash)	#linux系统
```

配置文件在

~/.kube/config

可以通过把其他集群的配置文件复制过来，这样能切换不同的集群环境

```bash
kubectl config current-context
kubectl config get-context	#如果配置文件有复制过来，这里就会有选择了
kubectl config use-context kubeadm	#切换，假如新复制过来的context是kubeadm
```

### k8s的节点和标签

节点常用命令

```bash
kubectl get node
kubectl describe nodes minikube
kubectl get node -o wide
kubectl get node -o yaml	#yaml格式输出
```

标签常用命令

```bash
kubectl get node --show-labels
kubectl label node k8s-master env=test	#添加env=test的label
kubectl label node k8s-master env-			#删除label
```

设置节点的roles，特殊的label

```bash
kubectl label node k8s-node1 node-role.kubernetes.io/worker=	#设置节点为work
```

### 最小调度单位POD

- 共享相同命名空间
- pod是k8s中最小的调度单位

pod的定义

```yaml
apiVersion:
kind:
metadata:
	name:
spec:
	containers:
		- name:nginx						#容器1
			image:nginx
		- name:busybox					#容器2
			image:busybox
```

创建及删除pod

```bash
kubectl create -f nginx_busybox.yml
kubectl delete -f nginx_busybox.yml
```

pod基本命令

```bash
kubectl get pod
kubectl describe pod nginx_busybox
kubectl get pods nginx_busybox -o wide
kubectl exec nginx_busybox -it sh		#默认进入第1个容器
kubectl exec nginx_busybox -it -c busybox sh		#指定容器进行命令操作
```

### NameSpace

不同team，不同项目之间的隔离

基本操作命令

```bash
kubectl get namespace
kubectl create namespace demo
kubectl get pod --namespace kube-system
```

在特定namespace创建pod（默认是defaut的namespace）

```yaml
metadata:
	name:nginx
	namespace:demo
```

```
kubectl create pod -f nginx.yml		#先要建立demo的namespace
```

### 创建自己的context

修改默认的namespace为自己定义的namespace

通过修改contexts来实现

```bash
kubectl config set-context demo --user=minikube --cluster=minikube --namespace=demo
kubectl config use-context demo
kubectl get pod			#默认就搜索namespace=demo下的pod
kubectl config delete-context demo	#删除demo的context
```

### Controller和Deployment

Controller实现的目的监控当前的实际状态，当实际状态不满足预定的状态，就会想办法去改变,实现标签选择，容器升级，容器平滑扩容，缩容

举例：默认的nginx的部署文件，nginx_deployment.yml

```yaml
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template: # create pods using pod definition in this template
    metadata:
      # unlike pod-nginx.yaml, the name is not included in the meta data as a unique name is
      # generated from the deployment name
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

deployment能维护容器的规模，哪怕是删除，也会自动补充上

nginx从1.7.9**升级**到1.8版本  nginx_deployment_update.yml

```yaml
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.8 # Update the version of nginx from 1.7.9 to 1.8
        ports:
        - containerPort: 80
```

但不能用**create**参数来升级,用**apply**命令来实现

```bash
kubectl apply -f nginx_deployment_update.yml
```

也可以使用下面办法实现

```bash
kubectl set image deployment/nginx_deployment nginx=nginx:1.8
```

**扩容**pod数量从2到4，nginx_deployment_scale.yml

```yaml
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 4 # Update the replicas from 2 to 4
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.8
        ports:
        - containerPort: 80
```

可以通过以下方式来实现，方法很灵活

```bash
kubectl apply -f nginx_deployment_scale,yml
kubectl eit deployment nginx_deployment	#可以通过这个命令进deployment修改replicas参数,保存退出
kubectl scale --current-replicas=3 --replicas=4 deployment/nginx_deployment
```

### RedplicaSet和ReplicationController

```bash
kubectl get replicaset
kubectl scale --current-replicas=3 --replicas=4 deployment/nginx_deployment
```

### 升级回滚

```bash
kubectl rollout history deployment nginx_deployment
kubectl rollout history deployment nginx_deployment --revision 1	#查看第一个版本
kubectl rollout history deployment nginx_deployment --revision 2	#查看第二个版本
kubectl rollout undo deployment nginx_deployment									#退回到上一个版本
kubectl rollout undo deployment nginx_deployment --to-revison 2		#回滚到指定版本

```

### 使用Tectonic在本地单间多节点K8S集群

安装完以后修改配置文件，让minikube和tectonic并存，运行下列命理进行切换

```bash
$	kubectl config user-context minikube
$	kubectl get nodes	#可以验证切换过去的集群是否节点正确
```

shell命令补全

```bash
$	kubectl completion zsh/bash	#根据自己的shell环境选择
```

### k8s网络基础

### Service

ClusterIP	集群内部可以访问

```bash
$	kubectl expose pods nginx-pod
$	kubectl get svc		#能显示Cluster-IP和端口
```

原nginx-pod.yml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx-container
    image: nginx
    ports:
    - name: nginx-port
      containerPort: 80
```

NodePort	外界可以访问node节点

```bash
$	kubectl expose pods nginx-pod --type=NodePort
$	kubectl get svc		#能显示NodePort和端口，（默认暴露的端口在30000-32767之间随机）
```

LoadBalancer	运服务商可以提供

### NodePort类型Service和Label的应用

```bash
$	kubectl label node w1.techonicsanbox.com hardware=good
```

假如service的配置文件里label标签选择hardware=good，而node没有定义这个label，则pod就不会创建，会一直处在pending状态

### 容器的监控及weavescope

```bash
docker top 容器名
docker stats			#实时系统状态，cpu，内存占用
```

安装步骤

```bash
$ sudo curl -L git.io/scope -o /usr/local/bin/scope
$	sudo chmod +x /usr/local/bin/scope
$ scope launch 192.168.205.13
Unable to find image 'weaveworks/scope:1.13.1' locally
1.13.1: Pulling from weaveworks/scope
c9b1b535fdd9: Pull complete
.
Weave Scope is listening at the following URL(s):
  * http://10.0.2.15:4040/
  * http://192.168.205.13:4040/
  * http://192.168.49.1:4040/
```

或者同时监控多台k8s主机

```bash
$ scope launch 192.168.205.13 192.168.205.14
$ scope stop	#停止
```

### K8s资源监控heapster+Grafana+InfluxDB

安装

```bash
$ sudo docker pull k8s.gcr.io/heapster-grafana-amd64:v4.4.3
$ sudo docker pull k8s.gcr.io/heapster-amd64:v1.4.2
$ sudo docker pull k8s.gcr.io/heapster-influxdb-amd64:v1.3.3
```

3个deployment文件

查询apiVersion的方法

```bash
kubectl api-resources  |grep Deployment
kubectl api-resources  |grep Service
kubectl api-resources  |grep ServiceAccount
```

grafana.yaml

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: monitoring-grafana
  namespace: kube-system
spec:
  replicas: 1
  selector:
   matchLabels:
     app: monitoring-grafana
  template:
    metadata:
      labels:
        task: monitoring
        app: grafana
    spec:
      containers:
      - name: grafana
        image: k8s.gcr.io/heapster-grafana-amd64:v4.4.3
        ports:
        - containerPort: 3000
          protocol: TCP
        volumeMounts:
        - mountPath: /etc/ssl/certs
          name: ca-certificates
          readOnly: true
        - mountPath: /var
          name: grafana-storage
        env:
        - name: INFLUXDB_HOST
          value: monitoring-influxdb
        - name: GF_SERVER_HTTP_PORT
          value: "3000"
          # The following env variables are required to make Grafana accessible via
          # the kubernetes api-server proxy. On production clusters, we recommend
          # removing these env variables, setup auth for grafana, and expose the grafana
          # service using a LoadBalancer or a public IP.
        - name: GF_AUTH_BASIC_ENABLED
          value: "false"
        - name: GF_AUTH_ANONYMOUS_ENABLED
          value: "true"
        - name: GF_AUTH_ANONYMOUS_ORG_ROLE
          value: Admin
        - name: GF_SERVER_ROOT_URL
          # If you're only using the API Server proxy, set this value instead:
          # value: /api/v1/namespaces/kube-system/services/monitoring-grafana/proxy
          value: /
      volumes:
      - name: ca-certificates
        hostPath:
          path: /etc/ssl/certs
      - name: grafana-storage
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  labels:
    # For use as a Cluster add-on (https://github.com/kubernetes/kubernetes/tree/master/cluster/addons)
    # If you are NOT using this as an addon, you should comment out this line.
    #kubernetes.io/cluster-service: 'true'
    kubernetes.io/name: monitoring-grafana
  name: monitoring-grafana
  namespace: kube-system
spec:
  # In a production setup, we recommend accessing Grafana through an external Loadbalancer
  # or through a public IP.
  # type: LoadBalancer
  # You could also use NodePort to expose the service at a randomly-generated port
  # type: NodePort
  ports:
  - port: 80
    targetPort: 3000
  selector:
    k8s-app: grafana
  type: NodePort
```

heapster.yaml

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: heapster
  namespace: kube-system
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: heapster
  namespace: kube-system
spec:
  replicas: 1
  template:
    metadata:
      labels:
        task: monitoring
        k8s-app: heapster
    spec:
      serviceAccountName: heapster
      containers:
      - name: heapster
        image: k8s.gcr.io/heapster-amd64:v1.4.2
        imagePullPolicy: IfNotPresent
        command:
        - /heapster
        - --source=kubernetes:https://kubernetes.default
        - --sink=influxdb:http://monitoring-influxdb.kube-system.svc:8086
---
apiVersion: v1
kind: Service
metadata:
  labels:
    task: monitoring
    # For use as a Cluster add-on (https://github.com/kubernetes/kubernetes/tree/master/cluster/addons)
    # If you are NOT using this as an addon, you should comment out this line.
    #kubernetes.io/cluster-service: 'true'
    kubernetes.io/name: Heapster
  name: heapster
  namespace: kube-system
spec:
  ports:
  - port: 80
    targetPort: 8082
  selector:
    k8s-app: heapster
```

influxdb.yaml

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: monitoring-influxdb
  namespace: kube-system
spec:
  replicas: 1
  template:
    metadata:
      labels:
        task: monitoring
        k8s-app: influxdb
    spec:
      containers:
      - name: influxdb
        image: k8s.gcr.io/heapster-influxdb-amd64:v1.3.3
        volumeMounts:
        - mountPath: /data
          name: influxdb-storage
      volumes:
      - name: influxdb-storage
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  labels:
    task: monitoring
    # For use as a Cluster add-on (https://github.com/kubernetes/kubernetes/tree/master/cluster/addons)
    # If you are NOT using this as an addon, you should comment out this line.
    # kubernetes.io/cluster-service: 'true'
    kubernetes.io/name: monitoring-influxdb
  name: monitoring-influxdb
  namespace: kube-system
spec:
  ports:
  - port: 8086
    targetPort: 8086
  selector:
    k8s-app: influxdb
```

### 自动横向伸缩

```bash
kubectl run php-apache --image=k8s.gcr.io/hpa-example --requests=cpu=200m expose --port=80
kubectl autoscale deployment php-apache --cpu-precent=50 --min=1 --max=10
kubectl get horizontalpodautoscaler	#查看cpu负载
```

### k8s的log采集

Syslog-----ELK(elasticsearch+logstash+kibana)

hosted log服务

Fluentd

ElasticSearch(Log Index)

Kibana(Log可视化)

LogTrail(Log UI查看)

```bash
kubectl label node --all beta.kuberneters.io/fluentd-ds-ready=true	#给所有节点打标签
```

### k8s集群监控prometheus

### CICD（开源模式）

GitLab

Jenkins或者Travis CI

### GitLab Server搭建

以Centos7为例，准备一台至少内存为4G的机器。

#### 安装依赖文件

```bash
sudo yum install -y git vim gcc glibc-static telnet
sudo yum install -y curl policycoreutils-python openssh-server
sudo systemctl enable sshd
sudo systemctl start sshd

sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix
```

#### 设置gitlab安装源

如果在国内的话，可以尝试使用清华大学的源。

新建 /etc/yum.repos.d/gitlab-ce.repo，内容为

```bash
[gitlab-ce]
name=Gitlab CE Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/
gpgcheck=0
enabled=1
```

如果在国外的话，可以使用

```bash
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash
```

#### 安装GitLab

关于域名，如果要是设置域名，则如下，这个域名可以是真实购买的域名，如果您要把gitlab安装到公网比如阿里云上的话。

如果只是想本地测试，则可以像下面一样，设置一个example的域名，然后记得在本地你的笔记本设置host，如果是MAC就在 /etc/hosts里添加 一行 `192.168.205.10 gitlab.example.com`

```bash
sudo EXTERNAL_URL="http://gitlab.example.com" yum install -y gitlab-ce
```

如果不想设置域名，或者想将来再考虑，可以直接

```bash
sudo yum install -y gitlab-ce
```

安装完成以后，运行下面的命令进行配置

```bash
sudo gitlab-ctl reconfigure
```

#### 登录和修改密码

打开http://gitlab.example.com/ 修改root用户密码，然后使用root和新密码登陆。

### GitLab CI服务器的搭建

不使用Jenkins（因为gitlab ci 跟gitlab的搭配更加方便，且功能上也不输Jenkins）

GitLab CI服务器最好是单独与gitlab服务器的一台Linux机器。

#### 安装Docker

```bash
curl -sSL https://get.docker.com/ | sh
```

#### 安装gitlab ci runner

```bash
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.rpm.sh | sudo bash
sudo yum install gitlab-ci-multi-runner -y
```

查看是否运行正常

```bash
[vagrant@gitlab-ci ~]$ sudo gitlab-ci-multi-runner status
gitlab-runner: Service is running!
[vagrant@gitlab-ci ~]$
```

#### 设置Docker权限

为了能让gitlab-runner能正确的执行docker命令，需要把gitlab-runner用户添加到docker group里, 然后重启docker和gitlab ci runner

```bash
[vagrant@gitlab-ci ~]$ sudo usermod -aG docker gitlab-runner
[vagrant@gitlab-ci ~]$ sudo service docker restart
Redirecting to /bin/systemctl restart docker.service
[vagrant@gitlab-ci ~]$ sudo gitlab-ci-multi-runner restart
```

#### 注册配置GitLab CI

```
sudo gitlab-ci-multi-runner register
#域名输入GitLab的域名http://gitlab.example.com/
#令牌Token从GitLab的CI/CD里去寻找
#tags要输入，方便后期Pipeline构建过程选择Runner,比如demo，下面例子会用到
```

#### 演示Pipeline

举例.gitlab-ci.yml

```yaml
#定义stages
stages:
	- build
	- test
	- deploy
#定义job
job1:
	stages: build
	tags:					#注册时候定义的tags
		- demo
	script:
		- echo "I am job1"
		- echo "I am in build stages"
#定义job
job2:
	stages: test
	tags:					#注册时候定义的tags
		- demo
	script:
		- echo "I am job2"
		- echo "I am in test stages"
#定义job
job3:
	stages: deploy
	tags:					#注册时候定义的tags
		- demo
	script:
		- echo "I am job3"
		- echo "I am in deploy stages"
```

