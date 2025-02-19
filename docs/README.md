# Java笔记

#### 介绍
Java笔记

#### 软件架构
软件架构说明
本博客使用 Docsify 进行搭建
Docsify 官网：https://docsify.js.org/

#### Docsify 快速搭建开源项目文档

1. 安装 Node.js

2.  将 npm 改为国内的镜像
    
    ```sh
    npm install -g cnpm --registry=https://registry.npm.taobao.org
    ```
    如果切换为镜像后仍出现报错，可能是淘宝镜像证书问题，切换为以下淘宝新镜像可解决
    ```sh
    npm config set registry https://registry.npmmirror.com
    ```
    
3. 安装 docsify

   `docsify` 官方快速开始文档：https://docsify.js.org/#/quickstart
   
   安装命令
   ```sh
   npm i docsify-cli -g
   ```

4. 部署到服务器

   `docsify` 部署文档：[https://docsify.js.org/#/deploy](https://link.zhihu.com/?target=https%3A//gitee.com/link%3Ftarget%3Dhttps%3A%2F%2Fdocsify.js.org%2F%23%2Fdeploy)

#### 使用说明

1. 启动命令

   ```sh
   docsify serve docs
   ```

   或者使用`python`命令启动

   ```bash
   cd docs && python -m http.server 3000
   **或者在docs目录下直接执行
   python -m http.server 3000
   ```

   

2. 默认访问地址 http://localhost:3000 

3. 在线地址

   `github` [Java Notes (liuxin-up.github.io)](https://liuxin-up.github.io/java-notes/#/)

   `gitee`&ensp;  [Java Notes (gitee.io)](https://cn_up.gitee.io/java-notes/#/)
