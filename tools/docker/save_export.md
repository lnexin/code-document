    有时我们需要将一台电脑上的镜像复制到另一台电脑上使用，除了可以借助仓库外，还可以直接将镜像保存成一个文件，再拷贝到另一台电脑上导入使用。

    对于镜像的导出和导入，Docker 提供了两种方案，下面分别进行介绍。

## 一、使用 export 和 import

### 1，查看本机的容器

这两个命令是通过容器来导入、导出镜像。首先我们使用 docker ps -a 命令查看本机所有的容器。

[![原文:Docker - 实现本地镜像的导出、导入（export、import、save、load）](https://www.hangge.com/blog_uploads/201905/2019050717392713580.png)](#)

  

### 2，导出镜像

（1）使用 docker export 命令根据容器 ID 将镜像导出成一个文件。

<table><tbody><tr><td><p>1</p></td><td><div><p><code>docker export f299f501774c &gt; hangger_server.tar</code></p></div></td></tr></tbody></table>

（2）上面命令执行后，可以看到文件已经保存到当前的 docker 终端目录下。

[![原文:Docker - 实现本地镜像的导出、导入（export、import、save、load）](https://www.hangge.com/blog_uploads/201905/2019050717450053020.png)](#)

  

### 3，导入镜像

（1）使用 docker import 命令则可将这个镜像文件导入进来。

<table><tbody><tr><td><p>1</p></td><td><div><p><code>docker import - new_hangger_server &lt; hangger_server.tar</code></p></div></td></tr></tbody></table>

（2）执行 docker images 命令可以看到镜像确实已经导入进来了。

[![原文:Docker - 实现本地镜像的导出、导入（export、import、save、load）](https://www.hangge.com/blog_uploads/201905/201905071749348504.png)](#)

  

## 二、使用 save 和 load

### 1，查看本机的容器

这两个命令是通过镜像来保存、加载镜像文件的。首先我们使用 docker images 命令查看本机所有的镜像。

[![原文:Docker - 实现本地镜像的导出、导入（export、import、save、load）](https://www.hangge.com/blog_uploads/201905/2019050717553854896.png)](#)

  

### 2，保存镜像

（1）下面使用 docker save 命令根据 ID 将镜像保存成一个文件。

<table><tbody><tr><td><p>1</p></td><td><div><p><code>docker save 0fdf2b4c26d3 &gt; hangge_server.tar</code></p></div></td></tr></tbody></table>


使用docker save命令导出对应镜像（保留tag信息，极空间仅支持有tag信息的tar文件导入）
```bash
docker save xhofe/alist:latest>D:\soft\alist.tar
```

（2）我们还可以同时将多个 image 打包成一个文件，比如下面将镜像库中的 postgres 和 mongo 打包：

<table><tbody><tr><td><p>1</p></td><td><div><p><code>docker save -o images.tar postgres:9.6 mongo:3.4</code></p></div></td></tr></tbody></table>

  

### 3，载入镜像

使用 docker load 命令则可将这个镜像文件载入进来。

<table><tbody><tr><td><p>1</p></td><td><div><p><code>docker load &lt; hangge_server.tar</code></p></div></td></tr></tbody></table>

  

## 附：两种方案的差别

特别注意：两种方法不可混用。  
如果使用 import 导入 save 产生的文件，虽然导入不提示错误，但是启动容器时会提示失败，会出现类似"docker: Error response from daemon: Container command not found or does not exist"的错误。

  

### 1，文件大小不同

export 导出的镜像文件体积小于 save 保存的镜像

### 2，是否可以对镜像重命名

* docker import 可以为镜像指定新名称
* docker load 不能对载入的镜像重命名

  

### 3，是否可以同时将多个镜像打包到一个文件中

* docker export 不支持
* docker save 支持

  

### 4，是否包含镜像历史

* export 导出（import 导入）是根据容器拿到的镜像，再导入时会丢失镜像所有的历史记录和元数据信息（即仅保存容器当时的快照状态），所以无法进行回滚操作。
* 而 save 保存（load 加载）的镜像，没有丢失镜像的历史，可以回滚到之前的层（layer）。

### 5，应用场景不同

* docker export 的应用场景：主要用来制作基础镜像，比如我们从一个 ubuntu 镜像启动一个容器，然后安装一些软件和进行一些设置后，使用 docker export 保存为一个基础镜像。然后，把这个镜像分发给其他人使用，比如作为基础的开发环境。
* docker save 的应用场景：如果我们的应用是使用 docker-compose.yml 编排的多个镜像组合，但我们要部署的客户服务器并不能连外网。这时就可以使用 docker save 将用到的镜像打个包，然后拷贝到客户服务器上使用 docker load 载入。




## 一、前言

随着容器技术的发展，现在很多的应用程序系统都会选择使用docker容器进行部署，但是有时候使用docker容器进行部署的时候会遇到问题，比如说我们的应用程序里面需要依赖其他第三方的镜像，如果这时候服务器是在内网不能连接外网的情况下，那么就无法部署了。基于这种情况，docker官方支持docker镜像和容器的导入和导出，我们可以在一台能够联网的机器上面编译镜像，然后导出镜像或者容器，最后把导出的镜像或者容器上传到内网服务器，然后在导入镜像或者容器，这样就可以了。

镜像和容器的导入、导出操作主要涉及到下面的几个命令：save、load、export、import。

演示过程中我们是在本地生成镜像或者容器，然后把镜像或者容器导出，最后上传到阿里云服务器演示导入功能。

我们使用VS 2019创建一个ASP.NET Core MVC的项目，添加Dockerfile文件：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

# 使用运行时镜像
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1-buster-slim
# 设置工作目录
WORKDIR /app
# 把目录下的内容都复制到当前目录下
COPY . .
# 暴露80端口
EXPOSE 80
# 运行镜像入口命令和可执行文件名称
ENTRYPOINT ["dotnet", "DockerDemo.dll"]

![复制代码](https://common.cnblogs.com/images/copycode.gif)

然后发布项目。我们查看现有的docker镜像

![](https://img2020.cnblogs.com/blog/1033738/202007/1033738-20200711092705882-134263669.png)可以看到：现在只有两个.net core的镜像。我们生成镜像：

 docker build -t dockerdemo .

如下图所示：

![](https://img2020.cnblogs.com/blog/1033738/202007/1033738-20200711094333233-849977170.png)

查看生成后的镜像

![](https://img2020.cnblogs.com/blog/1033738/202007/1033738-20200711094420022-770278941.png)

然后我们根据生成的镜像来运行容器，首先查看现有的容器：

![](https://img2020.cnblogs.com/blog/1033738/202007/1033738-20200711104807259-1928059914.png)

可以看到这时没有任何容器。我们运行容器：

![](https://img2020.cnblogs.com/blog/1033738/202007/1033738-20200711104851577-1700620918.png)

可以看到：容器已经运行成功了。 

## 二、docker镜像的导入和导出

## 1、docker镜像的导出

涉及到的命令：

docker save [options]  images [images...]

我们使用上面的镜像来演示镜像的导出：

docker save -o dockerdemo.tar  dockerdemo

如下图所示：

![](https://img2020.cnblogs.com/blog/1033738/202007/1033738-20200711094638060-1131008266.png)

这里导出的时候指定了导出后文件的路径，如果不指定路径，默认是当前文件夹。 

或者也可以使用下面的命令导出：

docker save > dockerdemo.tar dockerdemo

其中-o和>表示输出到文件，dockerdemo.tar为导出的目标文件，dockerdemo为源镜像名。

我们查看本地是否有了导出后的文件：

![](https://img2020.cnblogs.com/blog/1033738/202007/1033738-20200711094726629-2070930358.png)可以看到目录下面已经有了刚才导出的文件。 

## 2、docker镜像的导入

我们首先使用XFtp把上面导出的镜像文件上传到阿里云服务器

![](https://img2020.cnblogs.com/blog/1033738/202007/1033738-20200711095726168-1948077380.png)

然后进入文件所在的目录 

![](https://img2020.cnblogs.com/blog/1033738/202007/1033738-20200711095806564-2081745524.png)

我们查看阿里云服务器上面有哪些镜像：

![](https://img2020.cnblogs.com/blog/1033738/202007/1033738-20200711095930157-1668451765.png)

从上图中看出：现在阿里云服务器上面没有任何的镜像。

涉及到的导入命令load

接下来我们导入刚才上传的镜像。

docker load -i dockerdemo.tar

如下图所示：

![](https://img2020.cnblogs.com/blog/1033738/202007/1033738-20200711100235705-1874550895.png)

或者也可以使用下面的命令

docker load < dockerdemo.tar

其中-i(i即imput)和<表示从文件输入。上面的两个命令都会成功导入镜像以及相关元数据，包括tag信息。

导入后查看镜像：

![](https://img2020.cnblogs.com/blog/1033738/202007/1033738-20200711100334382-894588292.png)

可以看到有了我们刚才导入的镜像。导入了镜像以后就可以根据镜像运行容器，最后运行应用程序。

## 三、docker容器的导入和导出

接下来我们演示容器的导入和导出。

## 1、docker容器的导出

涉及到的命令export。

docker export [options]  container

我们把上面生成的容器导出：

docker export -o D:\containers\dockerdemocontainer.tar dockerdemo

如下图所示：

![](https://img2020.cnblogs.com/blog/1033738/202007/1033738-20200711105133346-2668524.png)

其中，-o表示输出的文件，这里指定了输出的路径，如果没有指定路径，则默认生成到当前文件夹。dockerdemocontainer.tar为目标文件，dockerdemo为源容器名。

我们查看目录下面是否生成了导出的容器：

![](https://img2020.cnblogs.com/blog/1033738/202007/1033738-20200711105206808-1016611633.png)

## 2、docker容器的导入

我们首先把导出的容器使用XFTP上传到阿里云服务器。

![](https://img2020.cnblogs.com/blog/1033738/202007/1033738-20200711105631044-1012399942.png)

涉及到的导入命令import。

docker import [options] file|URL|- [REPOSITORY[:TAG]]

如下图所示

![](https://img2020.cnblogs.com/blog/1033738/202007/1033738-20200711110136298-803225350.png)我们导入刚才上传的容器

docker import dockerdemocontainer.tar dockerdemo:imp

dockerdemocontainer.tar表示要导入的容器，dockerdemo:imp表示导入后的镜像名称，imp表示给导入的镜像打tag。

如下图所示

![](https://img2020.cnblogs.com/blog/1033738/202007/1033738-20200711110450315-1842734558.png)

然后我们查看镜像：

![](https://img2020.cnblogs.com/blog/1033738/202007/1033738-20200711111127816-338382367.png)

可以看到这时有我们刚才导入的镜像了，导入的镜像tag为imp。 

## 四、总结

下面我们来总结一下镜像和容器导入导出的区别：

1. 镜像导入是一个复制的过程，容器导入是将当前容器变成一个新的镜像。
2. docker save命令保存的是镜像（image），docker export命令保存的是容器（container）。
3. export命令导出的tar文件略小于save命令导出的。
4. 因为export导出的是容器，export导出的文件在import导入时，无法保留镜像所有的历史（即每一层layer信息），不能进行回滚操作。而save是根据镜像来的，所以导入时可以完整保留下每一层layer信息。如下图所示：dockerdemo:latest是save导出load导入的，dockerdemo:imp是export导出import导入的。
5. ![](https://img2020.cnblogs.com/blog/1033738/202007/1033738-20200711115830169-1262686556.png)
6. docker load不能对导入的镜像重命名，而docker import导入可以为镜像指定新名称。例如，上面导入的时候指定dockerdeom:imp。
    

对于是使用镜像导入导出还是使用容器导入导出该如何选择呢？有下面两点建议：

1. 若是只想备份image，使用save和load。
2. 若是在启动容器后，容器内容有变化，需要备份，则使用export和import。



### 1.镜像制作

使用Dockerfile制作一个[docker](https://so.csdn.net/so/search?q=docker&spm=1001.2101.3001.7020)镜像

#### 1.1 编辑Dockerfile文件

下面是一个制作[openssh](https://so.csdn.net/so/search?q=openssh&spm=1001.2101.3001.7020)的Dockerfile文件：

[root@docker]# vim Dockerfile

```
FROM centos:7LABEL  demo demo@gmail.comRUN yum -y install openssh-server \&& useradd natash \&& echo "redhat"|passwd --stdin natash \&& echo "redhat"|passwd --stdin root   \&& ssh-keygen -q -t rsa -b 2048 -f /etc/ssh/ssh_host_rsa_key -N ''\&& ssh-keygen -q -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key -N '' \&& ssh-keygen -t dsa -f /etc/ssh/ssh_host_ed25519_key -N ''ADD ssh_host_ecdsa_key   /tmp/ssh_host_ecdsa_keyADD ssh_host_ed25519_key /tmp/ssh_host_ed25519_keyADD ssh_host_rsa_key     /tmp/ssh_host_rsa_keyCMD  ["/usr/sbin/sshd", "-D"]```

说明：

FROM表示下载基本镜像

LABEL作者信息

RUN 表示要执行的动作，相当于执行脚本，执行的是/bin/sh -c ***的动作

ADD表示复制文件

CMD表示执行一个命令

#### **1.2 FROM 和 RUN 指令的作用**

**FROM**：定制的镜像都是基于 FROM 的镜像，这里的 centos就是定制需要的基础镜像。后续的操作都是基于 centos。

**RUN**：用于执行后面跟着的命令行命令。有以下俩种格式：

shell 格式：

```
RUN <命令行命令># <命令行命令> 等同于，在终端操作的 shell 命令。```

exec 格式：

```
RUN ["可执行文件", "参数1", "参数2"]# 例如：# RUN ["./test.php", "dev", "offline"] 等价于 RUN ./test.php dev offline```

**注意**：Dockerfile 的指令每执行一次都会在 docker 上新建一层。所以过多无意义的层，会造成镜像膨胀过大。例如：

```
FROM centosRUN yum install wgetRUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"RUN tar -xvf redis.tar.gz以上执行会创建 3 层镜像。可简化为以下格式：FROM centosRUN yum install wget \&& wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \&& tar -xvf redis.tar.gz```

如上，以 && 符号连接命令，这样执行后，只会创建 1 层镜像。

#### 1.3 镜像构建

在 Dockerfile 文件的存放目录下，执行构建动作。

以下示例，通过目录下的 Dockerfile 构建一个 openssl:demo（镜像名称:镜像标签）。

```
$ docker build -t openssl:demo .```

**注**：最后的 . 代表本次执行的上下文路径。

上下文路径，是指 docker 在构建镜像，有时候想要使用到本机的文件（比如复制），docker build 命令得知这个路径后，会将路径下的所有内容打包。

### 2. 导入导出命令介绍

涉及的命令有export、import、save、load

#### 2.1 save

* 命令  
    `docker save [options] images [images...]`
* 示例  
    `docker save -o nginx.tar nginx:latest`  
    或  
    `docker save > nginx.tar nginx:latest`  
    其中-o和>表示输出到文件，`nginx.tar`为目标文件，`nginx:latest`是源镜像名（name:tag）

#### 2.2 load

* 命令  
    `docker load [options]`
* 示例  
    `docker load -i nginx.tar`  
    或  
    `docker load < nginx.tar`  
    其中-i和<表示从文件输入。会成功导入镜像及相关元数据，包括tag信息

#### 2.3 export

* 命令  
    `docker export [options] container`
* 示例  
    `docker export -o nginx-test.tar nginx-test`  
    其中-o表示输出到文件，`nginx-test.tar`为目标文件，`nginx-test`是源容器名（name）

#### 2.4 import

* 命令  
    `docker import [options] file|URL|- [REPOSITORY[:TAG]]`
* 示例  
    `docker import nginx-test.tar nginx:imp`  
    或  
    `cat nginx-test.tar | docker import - nginx:imp`

### 2.5 区别

* export命令导出的tar文件略小于save命令导出的
* export命令是从容器（container）中导出tar文件，而save命令则是从镜像（images）中导出  
    基于第二点，export导出的文件再import回去时，无法保留镜像所有历史（即每一层layer信息，不熟悉的可以去看Dockerfile），不能进行回滚操作；而save是依据镜像来的，所以导入时可以完整保留下每一层layer信息。如下图所示，nginx:latest是save导出load导入的，nginx:imp是export导出import导入的。

### 2.6 建议

* 可以依据具体使用场景来选择命令
    
* 若是只想备份images，使用save、load即可
* 若是在启动容器后，容器内容有变化，需要备份，则使用export、import


https://www.hangge.com/blog/cache/detail_2411.html

https://www.cnblogs.com/dotnet261010/p/13283176.html


https://duckcloud.cn/archives/1690598317463

https://blog.csdn.net/welcome66/article/details/105640242