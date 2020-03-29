本节主要通过Swarm, 介绍跨主机多子网等复杂的网络配置方案, 认识基于容器引擎的集群管理模式. 此外, 还会介绍一些著名的容器网络管理工具以及常用的监控手段, 并向Swarm集群提出一些高效实用的高可用部署案例.

Swarm在Docker1.12版本之前属于一个独立的项目, 在Docker1.12版本发布之后, 该项目合并到了Docker中, 成为Docker的一个子命令, 及Swarm集群模式. 目前Swarm是Docker社区提供的唯一的一个原生支持Docker集群管理的工具. 他可以把多个Docker主机组成的系统转换成为单一的虚拟Docker主机, 使得容器可以组成跨主机的子网网络.

### 1 Docker Swarm命令
可以使用--help参数查看命令

![图片](https://uploader.shimo.im/f/pQm8RgRDIx8MWhh7.png!thumbnail)

最新版本的Docker1.13在Swarm命令中一共包含七个子命令.

![图片](https://uploader.shimo.im/f/EFkBh0vZ6jsaCVuT.png!thumbnail)

我们常说的Swarm模式, 并不是指Docker Swarm命令, 它还包括诸如Docker Node, Docker Service, Docker Stack, Docker Deploy, 甚至包括Docker Network等命令在内一套完整的集群部署管理体系, Swarm模式是融于Docker中的集群管理方案. 

* Swarm初始化

先来看集群初始化的子命令: docker swarm init.

![图片](https://uploader.shimo.im/f/aXslFLPVnwkjkfB8.png!thumbnail)

Swarm的初始化过程并不复杂, Shell中的选项实际上是Docker通过Swarm API进来操作的.

* 加入集群

docker swarm join , 主要包含四个选项

![图片](https://uploader.shimo.im/f/E0sLMGhtdUcRRFOk.png!thumbnail)

加入集群的命令与初始化命令颇为相似, 但是较为简单, 因为口令只在初始化出现一次, 所以如果要查看口令就需要使用下面的子命令.

* 管理添加节点的口令

Swarm添加节点是需要管理节点生成一个口令, 待添加的子节点需要凭借这个口令才能加入集群, 这个子命令(docker swarm join-token )更加简单, 主要用于管理集群口令, 该子命令只能管理节点.

* 离开集群

docker swarm leave 用于退出当前集群, 只有一个-f选项, 意为强制离开,无视警告.

* 解锁集群

docker swarm unlock 没有选项可用, 这个功能是由于最新的Docker 1.13版才加入的新功能, 用于解除锁定的Swarm集群.

* 管理解锁秘钥

docker swarm unlock-key

* 更新集群

docker swarm update 用于更新节点中的服务, 更新集群选项的说明如下:

![图片](https://uploader.shimo.im/f/587hQNso3Q0o1wqw.png!thumbnail)

### 2 Docker Node命令
Docker Swarm命令只能从宏观的角度调度节点, 并不能针对节点做一些操作, 而Docker Node就是对集群节点管理的命令

![图片](https://uploader.shimo.im/f/0d4KjWtgf8sI5nta.png!thumbnail)

集群节点管理命令的说明:

![图片](https://uploader.shimo.im/f/CGLOEKERKLIz6EpK.png!thumbnail)

* 节点降级

docker node demote Node[Node....]

* 查看节点信息

docker node inspect [OPTIONS]  self | NODE[NODE...]

* 集群节点列shell

docker node ls [OPTIONS]

* 提升节点权限

docker node promote NODE [NODE ...]

* 查看任务

docker node ps [OPTIONS] [NODE...]

* 删除节点

docker node rm [OPTIONS] NODE [....]

* 更新节点

docker node update [OPTIONS] NODE

![图片](https://uploader.shimo.im/f/PY99Gm9soAIE0a5p.png!thumbnail)

### 3 Docker Stack命令
该命令用来管理Docker栈, 这个命令是由于最新的Docker1.13版才加入的功能. 他一共包含五个子命令, 

![图片](https://uploader.shimo.im/f/FUTpThhWVWszd48l.png!thumbnail)

![图片](https://uploader.shimo.im/f/iFQWbXw1jKQ1d4ty.png!thumbnail)

* 部署Docker栈

docker stack deploy [OPTIONS] STACK

* 查看所有栈

docker stack ls

* 查看栈内任务

docker stack ps [OPTIONS] STACK

* 删除栈

docker stack rm STACK

* 查看栈内服务

docker stack services [OPTIONS] STACK

### 4 Docker 集群网络
至此我们可以以Swarm为核心搭建一个Docker集群网络, 体验一下Docker集群部署的便捷.

为了方便演示跨主机网络, 我们需要用到一个工具 -- Docker Machine, 这个工具与Docker Compse, Docker Swarm一起成为Docker三剑客, 下面来安装Docker Machine

![图片](https://uploader.shimo.im/f/cv9xzDfBZbsSshu9.png!thumbnail)

![图片](https://uploader.shimo.im/f/rRsBLNp7NFANzpcW.png!thumbnail)

安装过程和Docker Compose非常类似. 现在Docker三剑客已经全部到齐, 可以开始实战了.

**1.4.1 建立跨主机网络**

首先使用Docker Machine创建一个虚拟机作为manager的节点

![图片](https://uploader.shimo.im/f/loBrmbL0NWwhtInq.png!thumbnail)

查看虚拟机的环境变量等信息, 包括虚拟机中的ip地址

![图片](https://uploader.shimo.im/f/rcoe2fA7a9E8rxwp.png!thumbnail)

然后再创建一个节点作为work节点

![图片](https://uploader.shimo.im/f/xzRK57PfBP41M0Gp.png!thumbnail)

现在我们有了两个虚拟机, 使用Machine命令可以查看(删改部分无关列)

![图片](https://uploader.shimo.im/f/rC9o0fRxwXkjJupr.png!thumbnail)

但是目前这两台虚拟机并没有什么关系, 为了把它们关系起来, 我们需要Swarm登场了. 

因为我们使用的是Docker Machine创建的虚拟机, 所以可以使用docker-machine ssh命令来操作虚拟机, 在实际生产环境中,并不需要像下面那样操作, 只需要执行Docker Swarm即可.

把manager1加入集群:

![图片](https://uploader.shimo.im/f/281dPfCFd1QeIYHi.png!thumbnail)

用--listen-addr指定监听的ip端口, 实际的Swarm命令格式如下, 本例只是使用Docker Machine来连接虚拟机而已:

![图片](https://uploader.shimo.im/f/8pddNISZEok5jI96.png!thumbnail)

接下来, 再把work1加入集群中:

![图片](https://uploader.shimo.im/f/C1he1KZcu4Q8gikw.png!thumbnail)

我们继续新建虚拟机manager2,work2, work3, 现在已经有5个虚拟机了, 使用docker-machine ls来查看虚拟机

![图片](https://uploader.shimo.im/f/m1r8tk1b88QuHHUo.png!thumbnail)

然后把剩余的虚拟机也加入到集群中.

将worker2添加到集群中

![图片](https://uploader.shimo.im/f/K1oTSqvW2NUnrIM0.png!thumbnail)

将worker3添加到集群中

![图片](https://uploader.shimo.im/f/JO2c5aLrG3MG73PH.png!thumbnail)

将manager2添加到集群中, 先将Manager1中获取manager的token.

![图片](https://uploader.shimo.im/f/yuoiFnd8uq0CmlLq.png!thumbnail)

然后将manager2添加到集群中

![图片](https://uploader.shimo.im/f/KdAGDKBMUEgXTNS3.png!thumbnail)

现在再看集群信息

![图片](https://uploader.shimo.im/f/a3KmCjTOvMUuguJL.png!thumbnail)

为了演示的更清晰, 下面我们把宿主机也加入到集群中, 这样我们使用Docker命令操作会清晰很多. 

直接在本地执行加入集群命令

![图片](https://uploader.shimo.im/f/ExxEBlQouHIWxNHz.png!thumbnail)

现在我们三台manager, 三台worker. 其中一台是宿主机, 五台是虚拟机.

![图片](https://uploader.shimo.im/f/sZK11S4D0RAbsGYo.png!thumbnail)

查看网络状态

![图片](https://uploader.shimo.im/f/5mXNUHsPHKYUcT9d.png!thumbnail)

可以看到: 在Swarm上默认已有一个名为ingress的overlay网络, 默认在Swarm里使用, 笨栗子中会创建一个新的overlay网络.

![图片](https://uploader.shimo.im/f/YXYFLiJFlJsktnm1.png!thumbnail)

这样, 一个跨主机网络就搭建好了, 但是现在这个网络只是处在待机状态, 接下来我们会在这个网络上部署应用.

**1.4.2 在跨主机网络上部署应用**

需要说明的是: 我们上面创建的界定啊都是没有镜像的, 因此我们要逐一pull镜像到节点中, 这里我们使用前面大家的私有仓库.

![图片](https://uploader.shimo.im/f/1kwQZrHPEj4Omn4Y.png!thumbnail)

上面使用docker pull分别在五个虚拟机虚拟机节点拉取nginx:alpine镜像. 接下来我们要在这五个节点都部署一组Nginx服务.

部署Nginx服务使用swarm_test跨主机网络

![图片](https://uploader.shimo.im/f/NQoZw6WQG345Acqp.png!thumbnail)

查看服务状态

![图片](https://uploader.shimo.im/f/ISLxXLxSrBMZ3iK6.png!thumbnail)

查看helloworld服务详情

![图片](https://uploader.shimo.im/f/jkDjJgDt5d43b7GX.png!thumbnail)

可以看到: 两个实例分别运行在两个节点上

进入两个节点, 查看服务状态

![图片](https://uploader.shimo.im/f/b1aZdh5nbgwhzoLz.png!thumbnail)

上面的输出做了调整, 实际的NAMES值为:

![图片](https://uploader.shimo.im/f/YRwBWY0Veg4x1I2k.png!thumbnail)

记住上面的两个名字, 接下来我们来看这两个跨主机的容器是否能够互通.

首先使用Machine进入manager1节点, 然后使用docker exec -i命令进入helloworld.1容器中,ping运行在woker2节点的helloworld.2容器中.

![图片](https://uploader.shimo.im/f/GyaReJ3YhI8pFLVb.png!thumbnail)

然后使用Machine进入worker2节点, 再然后使用docker exec -i 命令进入helloworld.2容器中, ping运行在manager1节点的helloworld.1容器中.

![图片](https://uploader.shimo.im/f/3RF1xbWe05ArEji4.png!thumbnail)

至此, 这两个跨主机的服务集群里各个容器是可以互相连接的.

![图片](https://uploader.shimo.im/f/UymbT3BuXlYXrhUE.png!thumbnail)

![图片](https://uploader.shimo.im/f/CvncsN1beT8AnsHf.png!thumbnail)



