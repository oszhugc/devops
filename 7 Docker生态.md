Docker作为时下最流行的容器化软件, 围绕着Docker容器的生态系统也在不断完善, 比如容器化, 服务发现, 全局配置存储, 网络工具, 调度, 集群管理, 编排, 安全等.

## 1 宿主机管理工具 Machine
Docker Machine是Docker官方提供的一个工具, 他可以帮助我们在远程的机器上安装Docker, 或者在虚拟机上直接安装虚拟机并在虚拟机中运行Docker. 我们还可以通过docker-machine命令来管理这些虚拟机和Docker集群.

### 1.1 Machine的安装
在安装Docker Machine前需要现在本地安装Docker,

Docker Machine的安装非常简单, 在Ubuntu中直接把可执行文件下载到本地就可以了.

![图片](https://uploader.shimo.im/f/Rs9drwGPLQ4f1BWB.png!thumbnail)

Docker Machine也是个开源项目, 你可以选择安装不同的版本, 或者自行编译.

### 1.2 宿主环境管理
**1 远程管理机器**

如果你已经有多台服务器, 但是还没有安装Docker, 可以使用Docker Machine远程控制安装Docker, 方便快捷.

准备工作, 首先把ssh public key放到远程服务器中, 以便Machine登陆时不需要输入密码, 同时允许登陆用户直接使用docker命令, 例如docker用户组或者root用户组都可以

![图片](https://uploader.shimo.im/f/0219LztcePQtYyWs.png!thumbnail)

在上面的命令中,-d表示用什么驱动程序来创建目标主机, 他是--driver的简写. 驱动有很多种. --generic-ip-address表示要连接的主机IP地址, 也可以填写域名; --generic-ssh-user的值必须是一个可以免密操作Docker的用户, 在Docker安装之后有一个提示, 你可以选择是否把当前用户加入Docker用户组; --generic-ssh-key是一个本地的ssh私钥, 公钥需要事先放上去才能免密登陆.

安装成功之后执行 eval $(docker-machine env ops-node3)即可把远程主机的环境变量输出到当前的shell会话中(临时), 然后本地使用docker命令就可以直接操作远程的Docker机器了.

![图片](https://uploader.shimo.im/f/LWsuA1d8pmgb2517.png!thumbnail)

查看版本, 可以看到Docker的Client和Server不一样, 这是因为Server是来自远程主机的Docker进程, 而Client是本地的一个命令客户端.

**2 本地模拟Docker集群**

这种用法是Docker Machine中最常见的, 因为在学习Docker过程中, 在本地操作模拟集群的成本最低. 通过使用Docker Machine可以快速部署一个基于本地的虚拟机建立的Docker集群环境.

直接执行以下程序:

![图片](https://uploader.shimo.im/f/2eizDbZwV2c1Ml5J.png!thumbnail)

上面使用virtaulbox这个驱动程序, 是因为宿主机安装了VirtualBox虚拟机. 如果你使用Vmware, 可以替换--driver参数值为vmwarefusion, default表示虚拟机的名称.

使用docker-machine ls查看虚拟机列表. 使用docker-machine ip可以查看创建的虚拟机ip地址. 

![图片](https://uploader.shimo.im/f/dCohHT66GsYdCndH.png!thumbnail)

## 2 容器编排调度
Docker集群容器环境有一个必备的组件就是调度器. 调度器负责在相关的Docker主机上加载容器, 启动容器, 停止容器和管理这个进程的生命周期. 流行的调度器很多, 例如Marathon+Mesos(大规模集群首选), Docker门下的Swarm+Compse(发展迅速), Google门下的K8s等.

### 2.1 Rancher 集群管理面板
Rancher是一个开放源码的软件平台, 可以在生产中调度, 编排和管理容器(使用Docker和k8s). 使用Rancher提供晚上的开箱即用的管理平台, 不再需要运维开发使用不同的开源技术从头开始构建容器服务平台. Rancher提供了管理生产中所需要的整个软件堆栈. 

1. 安装Rancher 

![图片](https://uploader.shimo.im/f/ZXHm7ZAsqzo30NA4.png!thumbnail)

这个命令会下载一个比较大的镜像, 访问本地8080端口, 看到一个后台管理界面, Racher提供了强大的UI操作界面(命令行也有), 几乎所有的操作都可以通过这个界面完成.

因为Racher使用MySQL驱动(内置MySQL), 所以上面的命令只是启动一个容器, 并不具备生产环境要求, 要实现数据持久化则需要保证数据库的数据保留下来, 不随容器生命周期的结束而消失. 有两种方式保存Rancher的数据, 一种是连接已经存在的数据库.

![图片](https://uploader.shimo.im/f/GYedbui2pIseBFBL.png!thumbnail)

另一种是把数据库文件保存在数据卷中:

![图片](https://uploader.shimo.im/f/88RuQshB3coGxZGV.png!thumbnail)

Rancher界面自带简体中文

![图片](https://uploader.shimo.im/f/Ld7qNrV8FGEtHqFu.png!thumbnail)

1. 添加主机

单击"添加主机"按钮, 进入主机添加界面; 添加宿主机为一个节点, 只需要执行下面给出的agent容器运行命令

![图片](https://uploader.shimo.im/f/ZsoJtXIvOgMFVsGT.png!thumbnail)

这是一个运行结束后自动删除的容器, 他通过申请--privileged权限吧agent留在了/var/lib/rancher目录中

### 2.2 Nomad 行业领先的调度系统
Nomand是一个分布式, 高可用的数据中心感知型(跨域多数据中心)调度平台, 使用它可以轻松地部署任何规模的应用程序. Nomad可以在5分钟内部署100万个容器到5000台主机上, 其强大的调度能力和跨域感知能力是最大的特点. 

### 2.3 服务发现
服务发现无论在那种调度平台上都是其策略的一个重要组成部分, 在容器集群中,服务发现可以让容器在没有管理员敢于的情况下了解运行状态, 可以自行发现并注册各种组件的运行状态, 一旦注册成功便可以在服务发现系统上找到连接该组件的方法, 以便各个组件之间的交互.

服务返现通常是全局分布式配置存储服务, 可以存储基础设施中任意的服务配置信息. 

一些流行的服务发现项目如下:

* etcd
* consoul
* zookeeper
* crypt
* confd
## 3 Docker插件
Docker插件尽管很年轻, 但是发展迅速, 他的插件生态也在逐渐起步. 虽然目前可用的插件还比较少, 但是对已经有一些出色的插件被开发出来.

### 3.1 授权插件
* Casbin AuthZ Plugin

基于Casbin(一个高效开源访问控制库, 支持基于各种访问控制模型实施授权)的授权插件, 支持ACL,RBAC,ABAC等访问控制模型. 访问控制模型可以定制, 该策略可以持久存入文件或数据库.

* HBM Plugin

HBM Plugin是一个授权插件, 他可以阻止执行具有某些参数的命令,例如Docker用户组中需要更加细分权限, 如果不想让A用户使用--privalileged参数启动容器, 那么使用这个插件就可以组织A启动启动privileged容器.

* Twistlock AuthZ Broker

Twistlock AuthZ Broker是一个简单的可扩展Docker授权插件, 直接在主机上或容器内运行. 使用--tlsverify标志(用户名从证书通用名称中提取)启动Docker守护程序是, 将提供基本授权.

### 3.2 Flocker存储插件
容器除了使用宿主机本地目录, 还有一些Docker volumn plugins支持使用诸如iSCSI, NFS等方式的共享存储. 使用网络共享数据卷的好处在于它是独立于宿主机之外的. 只要容器可以访问共享存储的后端并已安装相关插件(volume plugin), 就可以跨宿主机访问数据.

安装Flocker 

![图片](https://uploader.shimo.im/f/Dd1VK76rJw0tgv5k.png!thumbnail)

安装全部套件, 包括Flocker的容器套件后, 用户可以使用Flocker来管理数据卷了. 创建一个volume driver类型的Flocker, 名为volume_test的数据卷.

![图片](https://uploader.shimo.im/f/5ZFFhtcVlmYYnX3I.png!thumbnail)

然后启动一个容器使用这个数据卷

![图片](https://uploader.shimo.im/f/obgBWfkB2EAE01na.png!thumbnail)

### 3.3 网络驱动插件
* Contiv Networking

也叫Netplugin, 是一个开源网络插件, 专门为多用户的微服务部署提供网络安全策略, 意在处理集群多主机系统中的联网问题.

* Kuryr Network Plugin

这个插件作为OpenStack Kuryr项目的一部分, 通过使用OpenStack的网络服务Neutron来实现Docker网络(libnetwork)远程驱动程序API. 它还包括一个IPAM驱动程序.

* Weave Network Plugin

这大概是Docker下最著名的网络插件, 早在Swarm还未问世的时候, Weave就承担了一部分Swarm的工作. Weave可以创建一个虚拟网络连接Docker容器--即使是跨主机网络, Weave实现了应用程序的自动发现. Weave网络具有可伸缩,分区容错性等特点.

## 4 Docker安全工具
### 4.1 Docker Slim

Docker的Logo是一条鲸鱼, 但是你的Docker容器可不需要鲸鱼这么大的体积. Docker Slim是"容器的神奇减肥药", 它允许分析容器镜像并删减多余的东西.

容器中不需要的依赖和模块会增加容器的体积, 使用Docker Slim可以把一个Python容器样本大小从433MB减少到约15.97MB, 把一个Java应用样本大小从743MB变为100.3MB. 该分析会展示出去实际运行必须的软件包.

### 4.2 Clair 

Clair 是一个开源项目, 用于静态分析APPC和Docker容器中的漏洞. 

## 5 监控与日志
### 5.1 cAdvisor 原生集群监控
来自Google的cAdvisor可以说是监控Docker集群的不二选择, 目前Kubernetes已经集成了cAdvisor组件, 部署K8s集群后可以直接使用cAdvisor监控集群状态. 由于cAdvisor是将数据缓存在内存中, 数据展示能力有限. 顾一般我们选择把cAdvisor+Influxdb+Grafana三者结合起来实现一个集监控, 收集, 显示于一体的监控平台.

### 5.2 Logspout  日志处理
Logspout是一个运行在Docker容器里面的Docker容器日志路由器. 他可以附件到主机上的所有容器, 然后将他们的日志转发到任何你想要存储的地方, 还具有可扩展的模块系统. 

### 5.3 Grafana 数据可视化
准确说, Grafana并不是一个日志工具,而是一个控制台, 一般配合各种日志工具实现一个完整的日志监控平台. 例如Docker Swarm集群中一般选择cAdvisor+InfluxDB+Grafana或者K8s集群中的heapster+influxdb+frafana. 

## 6 基于Docker的Paas平台
Docker的前身是Paas(平台即服务), Docker最初被设计就是要作为PasS平台后端管理引擎的. 所以使用Docker搭建一个PaaS平台并不是什么难事. 社区中以及有很多类型的基于Docker容器技术的PaaS产品. 

### 6.1 Deis - 轻量级PaaS平台
Deis有两个版本, 一个版本是V1, 基于CoreOS和Fleet, 现在不再开发. 另一个新版本采用k8s调度, 命名为Deis Workflow. 

Deis Workflow是一个开源的平台即服务(PasS), 他处在k8s集群和开发人员之间, 为开发人员提供一个友好的操作层, 使开发人员易于部署和管理应用程序.

### 6.2 Tsuru 可扩展Pass平台
Tsuru是一个可扩展和开源的PaaS平台, 同样Go语言开发.

### 6.3 Flynn 模块化PaaS平台
Flynn是一个开源的PaaS平台, Flynn意在运行可以在Linux上运行任何东西, 而不是无状态的Web应用. 









