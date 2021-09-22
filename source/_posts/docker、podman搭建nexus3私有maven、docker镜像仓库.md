---
title: Podman、Docker搭建Nexus3私有Maven、Docker镜像仓库
reward: true
copyright: true
date: 2021-09-15 18:09:58
categories:
- Linux
- Docker
- Podman
- Maven
- 容器化
tags:
- Linux
- Docker
- Podman
- Maven
- 容器化
keywords:
- Linux
- Docker
- Podman
- Maven
- 容器化
---

在日常开发中,为了更简单的使用一些自制的工具,因此有时会把工具集打成 jar 包方便调用  
虽然可以存放在 Maven 本地仓库,但是万物皆可云,为什么不搭建一个云端私有仓库呢  
众所周知,良心云套路云各大云平台都提供了免费的私有 Maven 仓库服务,用就好啦
# 此帖终结

哈哈逗你的,众所周知前期免费中后期就可能收费,作为一个DIY(qiong bi)爱好者,当然是自己动手搭建一个,经过 Google 最终选型是 Nexus  
Nexus 的全称是 Nexus Repository Manager,是 Sonatype 公司的一个产品。它是一个强大的仓库管理器，极大地简化了内部仓库的维护和外部仓库的访问(抄的)

# 进入教程
1. Podman、Docker安装  
   推荐官方文档: [Podman安装](https://podman.io/getting-started/installation)、[Docker安装](https://docs.docker.com/engine/install/)  
   
   `PS`:使用 Arch 系 Linux 安装 Podman 时会遇到权限问题, ArchWiki 上就有解决方案[(地址)](https://wiki.archlinux.org/title/Podman),Podman 基本兼容 Docker 命令,下文使用 Podman 进行操作,如使用 Docker ,需 root 权限  

2. 部署  
   此处使用最简单的 Podman 容器化部署 
   * 创建文件夹用于后面挂载容器数据  
     出于方便赋予 ``777`` 权限(重度洁癖者可自行授予其他权限)   
     ```shell
     mkdir -p ~/nexus/data
     chmod 777 -R ~/nexus  
     ```  
   * 创建容器并挂载对应数据卷  
     ```shell
     podman run --restart always -d \ 
                -v ~/nexus/data:/nexus-data \ 
                -p 8081-8089:8081-8089 \ 
                --name nexus \ 
                klo2k/nexus3
     ```  
     大约等待两分钟后通过浏览器访问 ``http://IP地址:8081`` 即可进入 Nexus 的管理界面  
     默认帐号为admin,默认密码可通过挂载的数据卷中的admin.password查看,登陆后可修改密码  
     端口映射了 8081-8089 九个端口作为冗余  

# 部署完成
接下来进入本文的第二部分,恰巧仓库搭建完,我需要一个 Docker 私有镜像仓库,又开始了漫漫选型路  
现如今比较常见的仓库有以下几个:
* Registry: Docker 官方提供的原生仓库
* Portus：由 SuSE 团队推出
* Harbor: VMWare 中国团队推出的企业级仓库  

企业级,多么诱人的名字,我理所当然地选择了 Harbor ,看了一圈下来果断劝退,理由是作为一个坚定的 Podman 使用者, Harbor 官方的部署方式是使用 docker-compose , Podman 目前并没有官方成熟的替代产品(痛心疾首),而手动部署的方式有过于麻烦也不便管理  
最终,在机缘巧合之下发现 Nexus3 的版本支持作为 Docker 镜像仓库(狂喜)  

# Nexus3 Docker 镜像仓库配置
1. 登陆 Nexus 后,点击设置
   ![Podman、Docker搭建Nexus3私有Maven、Docker镜像仓库-1](https://blog-1257162717.cos.ap-shanghai.myqcloud.com/Podman%E3%80%81Docker%E6%90%AD%E5%BB%BANexus3%E7%A7%81%E6%9C%89Maven%E3%80%81Docker%E9%95%9C%E5%83%8F%E4%BB%93%E5%BA%93/Podman%E3%80%81Docker%E6%90%AD%E5%BB%BANexus3%E7%A7%81%E6%9C%89Maven%E3%80%81Docker%E9%95%9C%E5%83%8F%E4%BB%93%E5%BA%93-1.png)  

2. 点击仓库并点击 Create repository 创建仓库
   ![Podman、Docker搭建Nexus3私有Maven、Docker镜像仓库-2](https://blog-1257162717.cos.ap-shanghai.myqcloud.com/Podman%E3%80%81Docker%E6%90%AD%E5%BB%BANexus3%E7%A7%81%E6%9C%89Maven%E3%80%81Docker%E9%95%9C%E5%83%8F%E4%BB%93%E5%BA%93/Podman%E3%80%81Docker%E6%90%AD%E5%BB%BANexus3%E7%A7%81%E6%9C%89Maven%E3%80%81Docker%E9%95%9C%E5%83%8F%E4%BB%93%E5%BA%93-2.png)  

3. Docker 镜像仓库分别有 group 、hosted 、proxy 三个选项  
   * group : 组类型，实质作用是组合多个仓库为一个地址
   * hosted : 本地存储，即同 docker 官方仓库一样提供本地私服功能
   * proxy : 提供代理其他仓库的类型，如 docker 中央仓库  

    此处选择 hosted 类型,填写上名称后,可勾选 HTTP 选项并填入 8082-8089 间任意一个端口作为镜像仓库的推拉端口  
    勾选 ``Enable Docker V1 API`` 允许 v1 版本 API  
    其他选项默认即可,点击 ``Create repository``即可创建仓库  
    结果如下:  
    ![Podman、Docker搭建Nexus3私有Maven、Docker镜像仓库-3](https://blog-1257162717.cos.ap-shanghai.myqcloud.com/Podman%E3%80%81Docker%E6%90%AD%E5%BB%BANexus3%E7%A7%81%E6%9C%89Maven%E3%80%81Docker%E9%95%9C%E5%83%8F%E4%BB%93%E5%BA%93/Podman%E3%80%81Docker%E6%90%AD%E5%BB%BAnexus3%E7%A7%81%E6%9C%89maven%E3%80%81docker%E9%95%9C%E5%83%8F%E4%BB%93%E5%BA%93-3.png)
    至此,大功告成

# 拓展  
   最后还有一些个人的思路,云服务器上部署私有镜像仓库听起来是很不错,但是实际上还是存在着一些问题,比如存储问题, Nexus 本身是支持使用 AWS 的对象存储作为 Blob Stores 的但是奈何我不想再注册一个 AWS 的帐号,而对象存储本身又是一笔支出,还是从免费入手,一番查证后,腾讯云的对象存储也是使用 AWS 的 S3 协议,但是一番操作过后,由于设置中的 Region 只能在已有列表中选择,宣告GG,于是又衍生出了另一个思路, 使用对象存储工具 COSFS 将腾讯云的存储桶挂载为服务器磁盘,然后在 Blob Stores 中进行添加,达到同样的效果,后续如果有时间去操作了,再出一个教程吧,而关于 Docker 和 Podman 如何使用私有镜像仓库,实际上还是有一些小坑的,也是等下个教程再说吧