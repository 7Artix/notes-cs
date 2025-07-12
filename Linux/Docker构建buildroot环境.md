# 安装`docker desktop`

在本机建立`Docker Daemon`时, 需要安装`docker desktop`. 
如果远程连接其他`docker`主机, 可以仅使用`brew install docker`安装`client`.

# 创建`docker`镜像
新建目录用以创建`docker`镜像, 路径随意.
在目录下创建`Dockerfile.buildroot`, 其中`buildroot`即为镜像名.
`Dockerfile`可以参考如下:
```Dockerfile
# 文件名: Dockerfile.buildroot
FROM ubuntu:20.04
  
ENV DEBIAN_FRONTEND=noninteractive
  
# 安装基本构建依赖
RUN apt update && apt install -y \
build-essential \
tzdata \
gcc \
g++ \
git \
wget \
curl \
unzip \
python3 \
rsync \
bc \
libncurses-dev \
flex \
bison \
make \
file \
cpio \
sudo \
gawk \
xz-utils \
zstd \
libssl-dev \
locales \
vim \
ca-certificates \
&& apt clean
  
# 设置时区和 locale，避免 make 报错
RUN ln -fs /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
dpkg-reconfigure -f noninteractive tzdata && \
locale-gen en_US.UTF-8 && \
update-locale LANG=en_US.UTF-8
  
ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US:en
ENV LC_ALL=en_US.UTF-8
  
WORKDIR /buildroot
```

设置`WORKDIR`后, 进入`docker`容器之后会自动进入`WORKDIR`目录. 如果不存在会自动创建.

之后在`Dockerfile`目录下构建镜像:
```bash
docker build -f Dockerfile.buildroot -t buildroot .
```

`-f`参数指定`Dockerfile`文件名, `-t`参数指定构建出的镜像名.
`.`表示构建的上下文路径为当前目录, 此时`docker`构建仅能使用当前目录文件.

# 创建`docker`容器

创建一个`Host`上的目录, 用于挂载到`docker容器`的目录中.
例如创建一个`case_sensitive`分区用来做文件交互`/Volumes/case_sensitive/mnt/buildroot`.
目标容器内部也应当有一个对应的目录, 用来挂载`Host`的目录, 例如`/buildroot`.

运行如下命令查看已有的`docker镜像`:
```bash
docker images
```

运行如下命令创建`docker`容器:
```bash
docker run -it \
--name buildroot \
--hostname buildroot-docker \
--mount type=bind,source=/Volumes/case_sensitive/mnt/buildroot,target=/buildroot \
buildroot \
bash
```

`--name`指定容器名
`--hostname`指定容器的主机名
`--mount`指定挂载主机目录, 便于开发
- `type=bind`将主机目录原样映射到容器中, 适合共享文件, 其中`source`必须是主机存在的目录, `target`目录不存在时会在容器中自动创建.
- `type=volume`使用`docker`自己管理的卷, 可以持久存储, 存储在`/var/lib/docker/volumes/`中, 自动创建, 管理比较方便, `source=volume-name`是`volume`名称, `target`为容器内挂载目录.
- `type=tmpfs`挂载一个基于内存的文件管理系统, 关闭容器后自动删除, `target`挂载于容器内部的`/tmpfs`
`buildroot`为镜像名
`bash`表示启动容器后运行`bash`

# 再次启动容器

使用如下命令列出所有已经创建的容器:
```bash
docker ps -a
```

输出例如:
```bash
CONTAINER ID   IMAGE       COMMAND   CREATED        STATUS                       PORTS     NAMES

70b35305cd28   buildroot   "bash"    27 hours ago   Exited (255) 2 minutes ago             buildroot-dev
```

使用如下命令启动并进入:
```bash
docker start 70b35305cd28
docker exec -it 70b35305cd28 bash
```

# 保存&恢复构建好的镜像

使用如下命令查看现有的全部镜像:
```bash
docker images
```

使用如下命令保存构建好的镜像:
```bash
docker save -o imagename.tar imagename:tag
```

恢复镜像到`docker`中:
```bash
docker load -i imagename.tar
```