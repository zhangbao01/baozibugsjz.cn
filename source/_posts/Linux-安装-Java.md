---
title: Linux 安装 Java
---
## CentOS
### 上传安装包或自行下载
地址: https://www.oracle.com/cn/java/technologies/javase/javase-jdk8-downloads.html
### 创建安装文件夹
```bash
$ mkdir /usr/local/java/
```
### 解压安装
```bash
$ tar -zxvf jdk-8u291-linux-x64.tar.gz -C /usr/local/java/
```
### 编辑配置环境文件
```bash
$ vim /etc/profile
```
### 文件末尾添加
```bash
$ export JAVA_HOME=/usr/local/java/jdk1.8.0_291
$ export JRE_HOME=${JAVA_HOME}/jre
$ export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
$ export PATH=${JAVA_HOME}/bin:$PATH
```
### 生效修改
```bash
$ source /etc/profile
```
### 校验
```bash
$ java -version
```
### 直接运行jar
```bash
$ nohup java -jar company-0.0.1-SNAPSHOT.jar >log.txt &
```
`nohup`挂起运行
`>`输出文件
`&`后台运行

### 挂起`nohup`和后台`&`区别