

# 安装nodejs

## 使用EPEL安装

- ```
  yum info epel-release  先确认系统是否已经安装了epel-release包
  ```

- ```
  yum install epel-release    没有则安装
  ```

- ```
  yum install nodejs  安装nodejs
  ```

- ```
  node -V 查看版本
  ```

## 升级nodejs

- ```
  npm install -g n  n是nodejs管理工具
  ```

- ```
  n latest 安装最新版
  ```

- ```
  n 切换版本
  ```

## 切换失效的解决办法

- ```
   which node
  /usr/local/bin/node #举个例子
  ```

- ```
  vim ~/.bash_profile
  ```

- 将下面两行代码插入到文件末尾：

- ```
  export N_PREFIX=/usr/local #node实际安装位置
  export PATH=$N_PREFIX/bin:$PATH
  ```

- ```
  source ~/.bash_profile 执行source使修改生效
  ```

# 开始部署

## 全局安装pm2

- ```undefined
  npm install pm2 -g 全局安装pm2
  ```

- ```bash
  ln -s /home/bin/pm2 /usr/local/bin/pm2 创建pm2链接
  ```

## 项目打包

- 写一个node启动脚本

- ```jsx
  const fs = require('fs');
  const path = require('path');
  const express = require('express');
  const app = express();
  
  app.use(express.static(path.resolve(__dirname, './manage'))) ##这里你项目的名称
  
  app.get('*', function(req, res) {
      const html = fs.readFileSync(path.resolve(__dirname, './manage/index.html'), 'utf-8')
      res.send(html) #index.html的位置
  })
  
  app.listen(8002); ##在Web访问的端口
  ```

- 生成 package.json文件

- ```json
  {
    "name": "back_manage",
    "version": "1.0.0",
    "description": "",
    "main": "app.js",
    "scripts": {
      "test": "echo \"Error: no test specified\" && exit 1"
    },
    "author": "",
    "license": "ISC",
    "dependencies": {
      "express": "^4.15.3"
    }
  }
  ```

## 文件上传和启动

- 注意：将vue项目打包文件、app.js启动脚本、package.json配置文件，上传至该文件夹下，注意：三个文件是同级的

- ``` undefined
  npm install 在项目文件夹里输入
  ```

- 在项目文件夹里启动xxx.js

- ```undefined
  pm2 start xxx.js
  ```

- pm2的其他一些命令

- ```dart
  pm2 show 0（id）//查看某个启动的应用详情
  pm2 show list//查看当前启动的所有应用
  pm2 stop 0(id)//关闭某个应用
  ```

