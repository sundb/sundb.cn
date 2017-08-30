---
title: 如何优雅解决egret请求http的跨域问题
---

## 碰到问题
使用nginx可以很容易解决egret请求http的跨域问题
但是在开发阶段,如果需要调试或者断点,就会出现跨域的问题

## 查找问题
* egret使用node的http server来处理h5的加载,熟悉node同学都知道如何在node中对消息进行转发,只要能解决egret的消息转发,就解决了跨域问题

## 解决问题
egret可以在命令行和gui两种模式下进行服务的启动,但最终调用的命令都是egret startserver

### 命令行模式
* 使用npm安装http-proxy

```javascript
npm install -g http-proxy
```

* 修改egret的命令行启动服务脚本

mac上脚本路径: /Users/YOUR NAME/Library/Application\ Support/Egret/engine/5.0.6/tools/server/server.js

在顶部加入

```javascript
var httpProxy = require('http-proxy');
```

* 增加proxy

```javascript
var proxy = httpProxy.createProxyServer({
    target: 'http://192.168.10.38:8180/'
});
```

* 修改转发

```javascript
var server = http.createServer(function (request, response) {
  // 在些处对/api/的地址进行转发,修改成你需转换的地址验证
  if (/\/api\/.*$/.test(request.url)) {
    console.log(123);
    proxy.web(request, response);
    return;
  }

  response.setHeader("Access-Control-Allow-Origin", "*");
  m(request, response).then(function () {
    response.end();
  }).catch(function (e) {
    console.error(e);
    response.end();
  });
});
```

### gui模式
* 修改启动脚本
mac上脚本路径

```
/Applications/Egret\ Wing\ 3.app/Contents/Resources/app/extensions/http-server/server.js
```

在顶部加入

```javascript
var httpProxy = require('http-proxy');
```


* 在http-server这级目录执行

```
npm install http-proxy --save
```

* 增加proxy

```javascript
var proxy = httpProxy.createProxyServer({
    target: 'http://192.168.10.38:8180/'
});
```

* 修改server启动脚本
该原有的代码

```
var server = connect()
  .use(serveStatic(config.root))
  .listen(config.port, '0.0.0.0', function (err) {
    config.error = undefined;
    ss(server);
  }
);

```

修改为

```
var server = connect()
  .use(serveStatic(config.root))
  .use(function(req, res){
    proxy.web(req, res);
  })
  .listen(config.port, '0.0.0.0', function (err) {
    config.error = undefined;
    ss(server);
  }
);


```

重启egret wing

### 吐槽egret
明明可以一个app解决的事,你要分成10几个app,好玩吗?
通过打log研究了下egret的launcher和wing,两个app耦合度如此之高,互相调,瞎了我的狗眼,什么傻逼玩意
