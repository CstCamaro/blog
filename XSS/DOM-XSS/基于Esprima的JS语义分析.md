# 基于Esprima的JS语义分析

### Esprima是什么
Esprima是一个javascript解释器，遵循ECMAScript，目前支持到ES6的语法解析。
通过Esprima提供的API可以将一段javascript代码解析成节点语法树。

#### Source code
```javascript=
var name = 'xray'
```
#### Syntax tree
```JSON
[
  {
    "type": "VariableDeclaration",
    "declarations": [
      {
        "type": "VariableDeclarator",
        "id": {
          "type": "Identifier",
          "name": "name"
        },
        "init": {
          "type": "Literal",
          "value": "xray",
          "raw": "'xray'"
        }
      }
    ],
    "kind": "var"
  }
]
```

### 为什么要对JS进行语义分析
之前挖过一些DOM-XSS，觉得通过人工审计js的方式来挖掘漏洞，准确性较高，但效率较低。
也尝试过使用chrome extension做过一些挂钩分析的事情，但是效果都不太理想。
总结了一些之前挖过的XSS漏洞，大多数漏洞中，漏洞点都出现在`赋值表达式`和`危险函数调用`内。

赋值表达式造成的DOM-XSS
```javascript=
//    标签
document.getElementById('id').innerHTML = code
//    属性
document.getElementById('id').src = code
//    伪协议
window.location.href = code
```

危险函数调用造成的XSS
```javascript=
//    eval
eval(code)
//    document.write
document.write(code)
```
以上两类，都是一眼就能看出这是存在问题的代码。
但是随着前端技术不断进步，自从引入了webpack技术之后，审计JS变得没有那么容易了，代码压缩、变量名混淆等，让代码审计变得十分头疼，例如以下这段代码：

#### Source code
```javascript=
function getQueryString(name) {
    var reg = new RegExp("(^|&)" + name + "=([^&]*)(&|$)", "i");
    var r = window.location.search.substr(1).match(reg);
    if (r != null) return unescape(r[2]);
    return null;
}

var url = getQueryString("url")
window.location.href = url
```
#### webpack
```javascript=
!function (e) {
    var n = {};

    function t(r) {
        if (n[r]) return n[r].exports;
        var o = n[r] = {i: r, l: !1, exports: {}};
        return e[r].call(o.exports, o, o.exports, t), o.l = !0, o.exports
    }

    t.m = e, t.c = n, t.d = function (e, n, r) {
        t.o(e, n) || Object.defineProperty(e, n, {enumerable: !0, get: r})
    }, t.r = function (e) {
        "undefined" != typeof Symbol && Symbol.toStringTag && Object.defineProperty(e, Symbol.toStringTag, {value: "Module"}), Object.defineProperty(e, "__esModule", {value: !0})
    }, t.t = function (e, n) {
        if (1 & n && (e = t(e)), 8 & n) return e;
        if (4 & n && "object" == typeof e && e && e.__esModule) return e;
        var r = Object.create(null);
        if (t.r(r), Object.defineProperty(r, "default", {
            enumerable: !0,
            value: e
        }), 2 & n && "string" != typeof e) for (var o in e) t.d(r, o, function (n) {
            return e[n]
        }.bind(null, o));
        return r
    }, t.n = function (e) {
        var n = e && e.__esModule ? function () {
            return e.default
        } : function () {
            return e
        };
        return t.d(n, "a", n), n
    }, t.o = function (e, n) {
        return Object.prototype.hasOwnProperty.call(e, n)
    }, t.p = "", t(t.s = 0)
}([function (e, n) {
    var t, r,
        o = (t = new RegExp("(^|&)" + "url" + "=([^&]*)(&|$)", "i"), null != (r = window.location.search.substr(1).match(t)) ? unescape(r[2]) : null);
    window.location.href = o
}]);
```
一个DOM-XSS的demo被打包后，不光是对初学者，即使是对常年阅读JS的人来说，也是一件非常头疼的事情。如果再加上一些逻辑判断、加密算法之后，让代码审计变得难上加难。

### 这个方案可行吗
但是冷静下来仔细阅读代码，我们就会发现，即使代码再怎么混淆，它都是有源可溯的。
最终触发XSS的核心代码就是这一段：
```
var t, r,
    o = (t = new RegExp("(^|&)" + "url" + "=([^&]*)(&|$)", "i"), null != (r = window.location.search.substr(1).match(t)) ? unescape(r[2]) : null);
window.location.href = o
```
最核心的就是变量o这个值，往上可以追溯到location.search.substr(1).match(t)
t可以拆分成：
![](https://s3.in.chaitin.net/hackmd/uploads/upload_59f44a78ee0bc67b46ad2ed42167a6af.png)

最终的触发点为：
```
window.location.href = o
```
不难看出，最终的可控输入点为`window.location.searh`，最终的触发点为`window.location.href`
![](https://s3.in.chaitin.net/hackmd/uploads/upload_e759031b346586215dd45ff5f2e38e05.png)


我们只需要将语法树进行解析，最终判断出o是从哪里来的即可。由于condition中的条件判断起来难度较大（我菜），所以暂时不做考虑，只需要知道它这个参数是从URL中过来的即可。
于是借助MDN Web 文档，查询了一些关于字符串处理的方法
```
'split', 'replace', 'concat', 'match', 'matchAll', 'sub', 'substr', 'substring', 'toLocaleLowerCase', 'toLocaleUpperCase', 'tirm', 'toLowerCase', 'toUpperCase', 'toString', 'trimEnd', 'trimStart', 'valueOf', 'raw'
```
如果一个字符串的property是这些方法的话，那么说明该字符串大部分情况下，还是在可控范围内的。
如果一个字符串的property是length的话，那么紧接着后面的一段表达式也没有必要继续跟踪下去了。

所以通过以上这个简单的小例子，我们可以确定，通过语法树来解析DOM-XSS是可行的。
### 方案的难点和坑在哪里
方法固然可行，那么在尝试解析的过程中，我遇到了哪些坑呢？

- 脚本一时爽，重构火葬场

最初使用Esprima时，直接从拉了一段代码进去，尝试进行解析，解析了一段时间后发现，js表达式远比我想象中复杂的多。
一个赋值操作，就可能需要考虑许多因素。
`
var a = b;
`

![](https://s3.in.chaitin.net/hackmd/uploads/upload_c85a0e026299b7c409c5e97d0f0355c2.png)


- 茴香豆的“茴”字有几种写法？

js的语法十分灵活，这也导致了解析难度很大，我们常见的赋值表达式，大概有以下这些：
```
//    AssignmentExpression expression
a = parent.top.window
b = parent.top.window.location
c = parent.top.window.location.href
d = parent.top.window.document
e = parent.top.window.document.cookie
f = parent.top.window.document.URL
g = window.name
h = parent.top.window.name
i = location
j = window
k = parent
l = document
m = document['cookie']
n = top.parent.window.parent['name']
o = top.parent.window['parent']['name']
p = 'location'
q = window[p]
```
其中有一些对象是可控的，比如window.location.href、window.name这种，还有一些对象是部分可控，比如document.URL、document.cookie可控，而document.domain不可控，我们要做的就是从这些对象中区分出哪些可控，哪些不可控。

```
b -> location
c -> location.href
e -> document.cookie
f -> document.cookie
g -> window.name
h -> window.name
i -> location
m -> document.cookie
n -> window.name
o -> window.name
q -> location 
```
经过解析，我们得到了可控的值。当表达式左边为单个变量名时，较为简单，如果表达式为以下形式
```
a.b.c = d.e.f
```
就需要拆分出每个变量的原始值、作用域，并且分析出该对象是否可控。

这里抛砖引玉，希望有兴趣的同学一起来研究JS语法树，让简单的、表层的DOM-XSS变得更加易于挖掘，这样就能有更多的精力和时间去学习更深层的语法特性。
