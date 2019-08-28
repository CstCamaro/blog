## DOM-XSS document.cookie

document.cookie通常是用来保存用户的登录状态，但是在某些情况下，也可以作为DOM-XSS的输入点，并且这种类型的DOM-XSS在利用时，更为隐蔽和持久。

### 常见对象
```
document.cookie
```

### 常见取值函数命名
```
getCookie
getCookieValue
gCookie
cookie.get
$.cookie
```

### 常见取值函数代码
```javascript
function getCookie(name) 
{
    var arr,reg=new RegExp("(^| )"+name+"=([^;]*)(;|$)"); 
    return (arr=document.cookie.match(reg))?unescape(arr[2]):null;
}
```

### 常见形式
1、将cookie的值，进行 decodeURIComponent 或 unescape 后，写入HTML中
```javascript
var foo = getCookie("foo");
$("#foo").html(foo);
```

2、将URL中的参数，作为URL地址，进行URL重定向操作
```javascript
var redirect_url = getCookie("redirect_url");
location.href = redirect_url;
```

3、将URL中的参数，作为JSON解析，但使用了eval函数
```javascript
json_data = getCookie("json_data");
try{
	var json_data = JSON.parse(json_data);
}
catch(e){
	eval(json_data);	//为了兼容IE7及以下不支持JSON数据的方式
}
```

### 小技巧
当cookie的值具有HttpOnly属性时，只能被浏览器所使用发向服务端，而不能被js读取到。
在http response header中，设置HttpOnly如下
```
Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT; Secure; HttpOnly
```
在js中，没有任何的API可以设置cookie的HttpOnly属性。
