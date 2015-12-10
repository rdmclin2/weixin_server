Nodejs微信公众平台服务器接入指南
---


> 最近申请了一个阿里云服务器,尝试着在上面用Nodejs搞个微信公众号机器人玩玩，申请了个人公众账号【技术栈】，你可以加关注然后尝试机器人的效果,虽然它现在还只会回“你来我家接我吧”,而且你看到的时候也不知道这服务器还在不在=。=.

<!-- more -->


# 搭建环境 
首先在阿里云服务器上搭建Nodejs开发环境，参考[在阿里云上搭建Nodejs开发环境](http://mclspace.com/2015/12/09/aliyun-build-nodejs-environment/)

# 接入过程
首先如果你没有[微信公众平台](https://mp.weixin.qq.com)账号，先去注册一个。

然后进入开发 -> 基本配置,这里会显示你的应用ID和应用密钥,记下这里你的AppID。

在服务器配置一栏中有3个参数:
```
URL是你的服务器的响应微信请求的地址
Token是你和微信通信的凭证，可以任意设置(3-32字符),例如设置为123456
EncodingAESKey 用于消息加密的密钥,我们点击后面的随机生成然后记下来就好。
```

Ok,现在我们有3个参数:AppID,Token和EncodingAESKey，然后我们在本地新建Nodejs服务器。

我们使用Express-generator快捷地生成一个可用的服务器,nodemon用来启动服务器:
```
$ express -e weixin_server
$ cd weixin_server && npm install
$ npm install --save nodemon
```
在weixin_server中新建文件Makefile和config.js,分别用来自动化构建服务器以及配置服务器:
```
$ touch Makefile config.js
```

建好的工程的目录如下:
```
.
├── app.js
├── bin
├── config.js
├── makefile
├── node_modules
├── package.json
├── public
├── routes
└── views
```

在config.js中写入刚刚我们获得的3个参数:
```
var config = {
    token: 'xxxxxxx',
    appid: 'xxxxxxxxxxx',
    encodingAESKey: 'xxxxxxxxxxxxxxxxxxxxxxxx'
};
module.exports = config;
```

在Makefile中写入运行命令,这里我们设置运行端口为80，目前微信只支持80:
```
run :
	@PORT=80 ./node_modules/.bin/nodemon ./bin/www
.PHONY: run
```

在routes文件夹中新建我们用来处理微信消息的路由wechat.js
```
$ cd routes && touch wechat.js
```

在app.js中的routes和users变量下添加我们处理微信消息的路由wechat,并配置使用该路由
```
var routes = require('./routes/index');
var users = require('./routes/users');
var wechat = require('./routes/wechat');

...

app.use('/', routes);
app.use('/users', users);
app.use('/wechat',wechat);
```

下面我们安装wechat模块，这是卜灵大叔写的用于和微信服务器进行通信的Nodejs模块
```
$ npm install --save wechat 
```

然后在我们的wechat.js中填入以下代码:
```
var express = require('express');
var router = express.Router();

var wechat = require('wechat');
var config = require('../config.js');

router.use('/', wechat(config, function (req, res, next) {
    // 微信输入信息都在req.weixin上
    var message = req.weixin;
    if (message.FromUserName === 'diaosi') {
        // 回复屌丝(普通回复)
        res.reply('hehe');
    } else if (message.FromUserName === 'text') {
        //你也可以这样回复text类型的信息
        res.reply({
            content: 'text object',
            type: 'text'
        });
    } else if (message.FromUserName === 'hehe') {
        // 回复一段音乐
        res.reply({
            type: "music",
            content: {
                title: "来段音乐吧",
                description: "一无所有",
                musicUrl: "http://mp3.com/xx.mp3",
                hqMusicUrl: "http://mp3.com/xx.mp3",
                thumbMediaId: "thisThumbMediaId"
            }
        });
    } else {
        // 回复高富帅(图文回复)
        res.reply([
            {
                title: '你来我家接我吧',
                description: '这是女神与高富帅之间的对话',
                picurl: 'http://nodeapi.cloudfoundry.com/qrcode.jpg',
                url: 'http://nodeapi.cloudfoundry.com/'
            }
        ]);
    }
}));

module.exports = router;
```
这是wechat模块给的示例代码，我们暂时先用着，下面我们部署服务器,首先将该服务器上传到github或者任何能够从远程部署代码的代码托管服务商上。以github为例新增.gitignore防止将node_modules和.DS_Store上传到服务器:
```
node_modules
example
.DS_Store
coverage
```
然后在github新建工程上传，不赘述。

在阿里云上clone我们刚刚上传的工程, 然后用make命令运行
```
$ git clone git@github.com:rdmclin2/weixin_server.git
$ cd wexin_server && npm install
$ make run
```

ok,我们继续到微信公众平台 -> 开发 -> 基本配置,填入
```
URL : <your ip>/wechat
Token : <your token>
EncodingAESKey: <your EncodingAESKey>
消息加解密方式 : 安全模式
```
点击提交,完成接入。