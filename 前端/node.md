## node

启动步骤

```sh
上述安装好nodejs环境后，生成构建一下
$ npm install
运行ui服务器
$ npm run dev
```



## 相关报错

### 1、npm  Syntax Error: Error: Cannot find module 'node-sass'

```sh
npm install --save-dev node-sass --registry=https://registry.npm.taobao.org --disturl=https://npm.taobao.org/dist --sass-binary-site=http://npm.taobao.org/mirrors/node-sass
```

### 2、npm install的时候，有时候是因为没有目录权限

### 3、启动后无法打开页面

可能是代码中配置的ip不是本机ip