## DOM-XSS url params

在所有的DOM-XSS中，因为从URL参数中取值而导致的DOM-XSS数量是最多的，也是最常见的一种

### 常见对象
```
location
location.search
location.href
location.hash
document.URL
```

### 常见取值函数命名
```
getParams
getUrlParam
getQuery
getQueryString
getRequests
```

### 常见取值函数代码
```javascript
function getQueryString(name) 
{
	var reg = new RegExp("(^|&)" + name + "=([^&]*)(&|$)", "i");
    var r = location.search.substr(1).match(reg);
    if (r != null) return unescape(r[2]); return null;
}
```

### 常见形式
1、将URL中的参数，进行 decodeURIComponent 或 unescape 后，写入HTML中
```javascript
var foo = getQueryString("foo");
$("#foo").html(foo);
```

2、将URL中的参数，作为URL地址，进行URL重定向操作
```javascript
var redirect_url = getQueryString("redirect_url");
location.href = redirect_url;
```

3、将URL中的参数，作为JSON解析，但使用了eval函数
```javascript
json_data = getQueryString("json_data");
try{
	var json_data = JSON.parse(json_data);
}
catch(e){
	eval(json_data);	//为了兼容IE7及以下不支持JSON数据的方式
}
```

### 小技巧
当取值的字符串，是以location.search作为对象时，取值范围从?至#前，不包含#和#后的内容
```javascript
function getQueryString(name) 
{
	var reg = new RegExp("(^|&)" + name + "=([^&]*)(&|$)", "i");
    var r = location.search.substr(1).match(reg);
    if (r != null) return unescape(r[2]); return null;
}
```
当取值的字符串，是以location.href作为对象时，取值范围为整个URL，包含#和#后的内容
```javascript
function getQueryString(name) 
{
	var reg = new RegExp("(^|&)" + name + "=([^&]*)(&|$)", "i");
    var r = location.href.substr(1).match(reg);
    if (r != null) return unescape(r[2]); return null;
}
```
当使用第二种取法的时候，可以将参数写在#后，因为#后的内容无法被服务器获取，从而避开了WAF的防护
