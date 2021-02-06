# K8S教程02-安装Docker私有仓库Harbor

Harbor是VMware公司开源的企业级公司开源的企业级DockerRegistry项目， 项目地址为<https://github.com/vmware/harbor>。
其目标是帮助用户迅速搭建一个企业级的Docker镜像服务。它以Docker公司开源的registry为基础，提供了管理UI，
基于角色的访问控制(Role Based Access Control)，AD/LDAP集成、以及审计日志(Auditlogging) 
等企业用户需求的功能，同时还原生支持中文。Harbor的每个组件都是以Docker容器的形式构建，
使用Docker Compose来对它进行部署。用于部署Harbor的Compose模板位于`/Deployer/docker-compose.yml`，
由5个容器组成，这几个容器通过Docker link的形式连接在一起，在容器之间通过容器名字互相访问。
对终端用户而言，只需要暴露 Proxy（即Nginx）的服务端口。

* Proxy：由Nginx 服务器构成的反向代理。
* Registry：由Docker官方的开源 registry镜像构成的容器实例。
* UI：即架构中的 core services，构成此容器的代码是 Harbor 项目的主体。
* MySQL：由官方 MySQL 镜像构成的数据库容器。
* Log：运行着 rsyslogd 的容器，通过 log-driver 的形式收集其他容器的日志。

