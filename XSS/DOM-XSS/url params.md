## DOM-XSS url params

在所有的DOM-XSS中，因为从URL参数中取值而导致的DOM-XSS数量是最多的，也是最常见的一种。

#### 常见对象
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

### 常见的DOM-XSS形式
1、将URL中的参数，进行 decodeURIComponent 或 unescape 后，写入HTML中。


#### 小技巧
