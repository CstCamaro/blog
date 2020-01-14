# 网页版微信DOM-XSS
### 漏洞代码
打开网页版的微信，有一段代码是如下
```
https://res.wx.qq.com/a/wx_fed/webwx/res/static/js/index_c7d281c.js
```

```
_postMessage: function(url, data, msg){
	            data.FromUserName = msg.FromUserName;
	            data.ToUserName = msg.ToUserName;
	            data.LocalID = msg.LocalID;
	            data.ClientMsgId = msg.ClientMsgId;
	            data = angular.extend(accountFactory.getBaseRequest(), {Msg: data});
	
	            data.Scene = msg.Scene || 0;
	
	            // 调用angularjs的$http发请求会经过JSON.stringify来处理data，而IE8下中文会变成unicode，所以这里先做字符串转换
	            if(utilFactory.browser.msie && parseInt(utilFactory.browser.version) < 9 && url == confFactory.API_webwxsendmsg) data = eval("'" + JSON.stringify(data) + "'");
	
    ...
    ...
    ...
```
此处判断了当前浏览器环境是否为IE，并且是否在IE9下，如果是，则将一些消息经过包装后，进行
`eval("'" + JSON.stringify(data) + "'")`
因为eval中，左右是用单引号进行闭合的，而JSON.stringify方法不会将单引号转义，所以只要输入的内容中含有`'-xss code-'`，即可实现漏洞利用。
### 利用思路
1、可以发送消息给别人，让别人@回复你，发送时触发
2、触发漏洞之后，可以读取联系人，给用户、公众号、文件传输助手发送消息，这些值在当前页面都可以得到。

### 为什么不提交
当我看到这段代码的时候，已经不支持IE8登录了。
