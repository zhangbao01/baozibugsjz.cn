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

创建一个目录，打开 `Git Bash Here`

### 安装全局Hexo

```bash
$ npm i hexo-cli -g
```

* `i` 是 `install`，`hexo-cli` 是 `hexoclient`，`-g` 是全局。

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

访问 `http://localhost:4000/` 可以看到本地结果

## 连接Github与本地

### 修改配置
打开博客根目录下的`_config.yml`文件，这是博客的配置文件，在这里你可以修改与博客相关的各种信息。
```bash
$ deploy:
  type: git
  repo: https://github.com/zhangbao01/zhangbao.github.io.git
  branch: main
```
### 安装hexo git插件
```bash
$ npm i hexo-deployer-git
```
### 上传github
```bash
$ hexo d
```
* 编写完markdown文件后，将文件保存到`\source\_posts`目录下，在根目录下输入`hexo g`生成静态网页，然后输入`hexo s`可以本地预览效果，最后输入`hexo d`上传到github上。然后访问`zhangbao01.github.io`，前缀必须和GitHub账户名一致，不然会报404。绑定域名在settings里。

