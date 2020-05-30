# 1、安装Node.js

node / Node.js

下载安装Node.js地址：
```
https://nodejs.org/en/
```
安装成功：
```
node -v
```

# 2、安装npm

npm / node package manager

Node.js的包管理器，是集成在Node.js中的，安装了Node.js也就安装了node package manager：
```
npm -v
```

# 3、安装cnpm

cnpm / 淘宝npm镜像

淘宝npm镜像地址：
```
https://npm.taobao.org/
```
安装cnpm：
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```
安装成功：
```
cnpm -v
```
# 4、安装webpack

webpack - JavaScript应用程序的静态模块打包器 - module bundler
```
cnpm install webpack -g
```
# 5、安装vue-cli

vue-cli / Vue Command Line Interface / Vue命令行接口 / 脚手架 / 自动生成项目代码

安装npm-cli：
```
cnpm install -g vue-cli
```
安装成功：
```
vue --version
```
# 5、创建Vue项目

选择好将要创建的项目的所在位置，在终端中进入该目录。
```
cd 项目预选目录
```
使用脚手架初始化项目，并且项目是基于webpack的： 
```
vue init webpack 项目目录名称
```
完善项目信息：
```
Project name（工程名）:回车
Project description（工程介绍）：回车
Author：作者名
Vue build（是否安装编译器）:回车
Install vue-router（是否安装Vue路由）：回车
Use ESLint to lint your code（是否使用ESLint检查js代码）：n
Set up unit tests（安装单元测试工具）：n
Setup e2e tests with Nightwatch（是否安装端到端测试工具）：n
Should we run npm install for you after the project has been created? (recommended)：回车。
```

# 7、编译运行Vue项目

进入项目目录：
```
cd 项目目录名称
```
安装项目所需要的依赖：
```
npm install
```
或
```
cnpm install
```
启动项目：
```
npm run dev
```
浏览器打开项目运行结果：
```
http://localhost:8080/
```

