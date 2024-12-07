---
title: CentOS停服后如何配置国内yum源镜像
draft: false
tags:
  - linux
---
1.  备份系统原始yum源配置文件
```
# 创建备份文件夹
sudo mkdir -p /etc/yum.repos.d/bak
# 将源配置移动至备份文件夹
sudo mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/bak
```
2. 下载镜像站配置
* 阿里云 [地址]([https://mirrors.aliyun.com/repo/](https://mirrors.aliyun.com/repo/)) 
```
# Centos7
sudo wget -O /etc/yum.repos.d/Centos-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

# 下载epel源配置文件
sudo wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
```
* 腾讯 [地址]([https://mirrors.cloud.tencent.com/repo/](https://mirrors.cloud.tencent.com/repo/))
```
# Centos7
sudo wget -O /etc/yum.repos.d/Centos-Base.repo http://mirrors.cloud.tencent.com/repo/centos7_base.repo

# 下载epel源配置文件
sudo wget -O /etc/yum.repos.d/epel.repo http://mirrors.cloud.tencent.com/repo/epel-7.repo
```

3. 重建本地缓存
```
# 运行以下两行命令重建本地缓存即可生效
sudo yum clean all && yum makecache

# 测试安装软件包是否正常
sudo yum install curl wget
```