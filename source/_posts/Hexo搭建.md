---
title: Hexo搭建
---

## 安装node.js

### 添加国内镜像

```bash
$ npm config set registry https://registry.npm.taobao.org
```

## 安装Git

## 安装Hexo

创建一个目录，打开 Git Bash Here 

### 安装全局Hexo

```bash
$ npm i hexo-cli -g
```

* i 是 install，hexo-cli 是 hexoclient，-g 是全局。

### 初始化文件夹

```bash
$ hexo init
```

### 生成静态页面

```bash
$ hexo g
```

### 打开本地服务器

```bash
$ hexo s
```

访问 http://localhost:4000/ 可以看到本地结果

## 连接Github与本地