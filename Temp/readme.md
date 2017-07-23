# 电影投票系统

> Java课程设计，想试试前后端完全分离的开发(原来在Class+时曾体验过)，于是有了下面4天2人的开发历程。先上图吧！Demo：[http://movie.ycjcl.cc](http://movie.ycjcl.cc)

## 安装

> 确保安装了[NodeJS](https://nodejs.org/en/)

下载项目，进入项目目录
```bash
$ git clone https://github.com/ycjcl868/VoteApp.git && cd VoteApp
```
安装项目依赖
```bash
$ npm install
```

使用 `bower` 安装前端依赖库  
```bash
// 若没有安装 `bower` ，使用 `npm install -g bower` 安装
$ bower install
```

编译打包项目
```bash
// 命令对项目进行打包，会生成 `dist` 打包好的目录  
$ npm build
``` 

运行项目 , [http://localhost:3008](http://localhost:3008)
```bash
// 开启项目 
$ npm start
``` 

就 ok 了，enjoy!~

## 配置
后台地址：[http://localhost:3008/admin](http://localhost:3008/admin)  
用户名： kylin  
密码：123456  

前台投票页：

* [x] 查看电影列表
* [x] 执行投票操作
* [x] 查看一小时内的日志及日志数 
* [x] 数据统计图
* [x] 搜索
* [ ] 投票成功分享

![](https://ycjcl868.github.io/VoteApp/img/1.gif)

后台管理：

* [x] 新增电影
* [x] 删除电影 
* [x] 查看日志
* [ ] 修改电影 
* [ ] 查看统计图

![](https://ycjcl868.github.io/VoteApp/img/2.gif)

## 系统架构

### 概述
共用了两台腾讯云服务器(CentOS 7)，一台前端Node服务器和一台Java SpringMVC服务器，之前的交互通过`JSON`。通过前后分离，将`MVC`中的`View`和`Controller`层分给前端，后台`SpringMVC`负责`Model`层，只提供`JSON`数据，交互如图：

![](https://ycjcl868.github.io/VoteApp/img/架构图.png)

### 前端服务器

前端在数据层采用`Express`+`Vue`，UI 层前台使用腾讯手 Q的[Frozenui](http://frozenui.github.io)  + [Zepto](http://zeptojs.com) + [LayerMobile](http://layer.layui.com/mobile/) + [CanvasJS(这个我极力推荐,个人贡献了中文文档)](http://canvasjs.com)，后台管理使用`老油条组合` [jQuery]() + [BootStrap](http://v3.bootcss.com) 。 整个前端使用`Gulp`进行自动化处理，配置见[Express+Gulp配置高效率开发环境](https://www.ycjcl.cc/2017/01/08/express-gulppei-zhi-gao-xiao-lu-kai-fa-huan-jing/)。

#### 首页：

![](https://ycjcl868.github.io/VoteApp/img/3.jpg)

![](https://ycjcl868.github.io/VoteApp/img/1.png)

#### 日志页：

![](https://ycjcl868.github.io/VoteApp/img/2.png)

#### 数据统计页：

![](https://ycjcl868.github.io/VoteApp/img/0.png)

#### 后台页面

![](https://ycjcl868.github.io/VoteApp/img/5.png)

![](https://ycjcl868.github.io/VoteApp/img/10.png)


#### 信息安全
对传输数据进行了加密处理：

```js
/**
 * 执行投票操作(POST)
 * @param userIP 用户ip
 * @param token  用户唯一标识加密(ip+'xxxx').md5()
 * @param cineId 电影id
 * @param province  省份
 *
 * 返回：state // 状态或票数
 *      cineId // 电影id
 */
router.post('/doVote', function(req, res, next) {
    var params = {};
    var send = res;
    params.userIP = req.session.ip;
    params.token = api.cryptoToken(req.session.ip);
    params.cineId = req.body.cineId;
    // 通过百度接口用 ip 获得省份
    api.getProvince(req.session.ip,function (res) {
        params.province = res;
        console.log(params);
        
        // 执行投票操作
        api.doVote(params,function (res) {
            console.log(res);
            send.send(res);
        })
    });
});
```

然后后台Java拿到`ip`后，对`ip`进行加密然后判断是否等于前端传来的`token`来执行投票操作。

![](https://ycjcl868.github.io/VoteApp/img/11.png)

#### 本地存储
首次在项目中使用`localStorage`的新特性，存储了是否投过票`voteId`和投票时间`voteTime`。来前端控制一个小时内只能投票一次(当然后台 Java 也得控制呢~)。当`Vue`到`ready`生命周期时就判断：

```js
var currentTime = new Date().getTime();
var voteTime = localStorage.getItem('voteTime');
// console.log('currentTime'+currentTime);
// console.log('voteTime'+voteTime);
// console.log(currentTime - voteTime >= 3600000);
if(currentTime - voteTime >= 3600000){
	// 一个小时后，就可以投票了(实际上是可以向 Java 发请求了)
	localStorage.setItem('voteId','-1');
}
```
这样，再次刷新时，如果投过`button`就成灰色，效果如图

![](https://ycjcl868.github.io/VoteApp/img/8.png)

当然，另一舍友今天也问我，为什么不用`cookie`，`cookie`还可以设置过期时间呢，区别是什么?，见文解析[详说 Cookie, LocalStorage 与 SessionStorage](http://jerryzou.com/posts/cookie-and-web-storage/)。不过，我认为`cookie`也是可以的，`localStorage`是键值对，而`cookie`只是字符串，没有封装`cookie`方法时操作`cookie`就比较麻烦。

#### 前端自动化工具Gulp
本想试下`Webpack`，发现不好配，还是配下稍微熟悉点的`Gulp`，发现真是方便，自动编译`Less`，自动检查语法错误，自动压缩，自动浏览器刷新，前端开发神器。

![](https://ycjcl868.github.io/VoteApp/img/13.png)



#### NodeJS图片异步上传
这一块原来用`Express`框架一直没上传过，这次看了`multer`官方原版文档 + 原来前端原生JS使用的`Formdata`对象，完美实现异步上传预览。 **下次发博时写个详细的解决Express异步上传图片预览**(因为现在用国内搜索引擎搜出来的基本都用不了，不是让你用jquery插件，就是让你npm install xxx模块，都不能很好的解决。)

### 后台服务器
只负责提供`JSON`数据，基本架构`Springmvc+Spring+MyBatis`是我舍友写的。数据库采用 `Mysql`(本来打算用`Oracle`，不过实在是太重了，不易管理，所以换Mysql了)

![](https://ycjcl868.github.io/VoteApp/img/7.png)

## 遇到的问题

###### 1. 首次和舍友采用**前后分离**的架构，没有文档驱动，接口就基本上通过 QQ 沟通。因为接口较少。

###### 2. 后台英文要稍微好点，不然**{"statue":1}**和**{"status":1}**是很容易写错了。(自我反省了好久，我有没有错，才发现不是我的错。)

![](https://ycjcl868.github.io/VoteApp/img/12.png)

![](https://ycjcl868.github.io/VoteApp/img/9.png)

###### 3. **json**问题，有时一粗心，就忘通过**JSON.parse(jsonStr)**将**json字符串**转成**json对象**了。

###### 4. 前端完全分离的调试问题，舍友的电脑能连的Wifi我连不上，我能连的他连不上。然后就买了腾讯云的`CentOS 7` Java服务器。

###### 5. 舍友对**Linux**不是太了解，在**大小写方面**和**目录权限**问题上没处理好！

![](https://ycjcl868.github.io/VoteApp/img/14.png)

###### 6. 后台对编码的处理没统一，先传过去的中文全是乱码，我一看数据库格式不是**utf8_general_ci**就明白了，结果还是乱码，舍友一看肯定是**SpringMVC**没有设置过滤器**UTF-8**格式，一加就**OK**了



###### 7. 最后一个问题是部署在云上时才发现的，因为用了**Nginx**服务器代理到了**NodeJS**，所以才开始获取到的用户ip全是`127.0.0.1`，最后**Node**要获取`ip`的话，只能通过`X-Forwarded-For` 的头`(起初我个人觉得不太安全，不过一测试，结果不能伪造)`。

![](https://ycjcl868.github.io/VoteApp/img/FFA456B0-26BE-45A6-A0B9-D8627275D7CA.png)

###### 8. 正式投票时，发现投了6个左右，后台Java Spring后台就挂掉了，显示的错误信息大概的意思是阻止请求信息，一查发现是没有配置连接数，所以就爆掉了，临时写了一个反馈的弹窗。最后改了之后，就能解决小量高并发了。

![](https://ycjcl868.github.io/VoteApp/img/D9AEEFE8-17CB-474A-829A-AD06D14B9424.png)
