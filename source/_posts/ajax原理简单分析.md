---
title: ajax的原理简单分析
date: 2018-04-15 11:43:20
tags: JavaScript
categories: JavaScript
---

#### 关于ajax

​	Ajax的出现是为了解决局部刷新的问题的。在web的远古时代，使用的方式是服务器渲染的技术。也就是说所有内容在服务端进行校验。对当时的web服务来说，这样是完全可行的。因为当时的web主要功能是图文浏览。随着时代的发展，需求的增加。我们需要在网站上完成一些工作。比如说：登录，注册等功能。这个时候，如果使用服务端渲染。用户在注册页面，必须填写完整所有的信息后，将页面发送至服务端进行校验。如果失败的话，用户之前填写的所有内容全部丢失，对用户来说非常不友好。此时，需要一种技术来解决这种困境。Ajax诞生了，可以不用将整个页面发送到服务端进行校验，可以将页面的局部内容发送至服务端进行处理。Ajax的诞生，才有了web的蓬勃发展与各种各样的新奇玩法。用户体验与web的性能有了质的提升。Ajax的主要作用总结如下：

```text
1. 实现了局部刷新，提升了用户体验。
```

##### ajax的实现原理
```javascript
// ajax 实现是依靠XMLHttpRequest对象实现的，IE浏览器下是ActiveXobject对象
let xhr = window.XMLHttpRequest ? new XMLHttpRequest() : new ActiveXObject('Microsoft.XMLHTTP');

// 参数分别是请求的方式，请求地址，同步（默认true）
xhr.open(method, url, true)		

// 等待浏览器响应
xhr.send();						

xhr.onreadystatechange = function () {
    if (this.readyState != 4) return
}
```

###### 实现详解

第一点  `xhr`这个对象的状态`readyState`
0 当xhr对象创建的时候
1 `open()`方法调用的时候
2 已经接受到了响应的响应头
3 正在下载响应体
4 整个响应报文下载完毕

  所以，大家看到定义的`onreadystatechange`方法，用来监听对象的状态变化，当`readyState`值为4，我们可以认为响应已经结束了。我们可以进行下来的处理了。虽然报文下载结束了，但是服务器一般有多种的情况，比如，我们常见的404，这是什么情况呢？其实，`xhr`对象上还有一个属性，`xhr.status`。这个属性值有很多，如下：
```http
200——交易成功

404——没有发现文件、查询或URl
```
还有很多状态码。文章末尾总结一下

  当浏览器返回http状态码的时候，我们可以利用`xhr.status`对响应的状态码进行获取然后处理。比如，ajax的时候，我们一般关心的是浏览器给我返回的是我想要的数据，那么这时候，对应的`xhr.status`是200，所以上述代码的最后一句的判断条件，在只关心服务器返回正常的情况下，是这样
```javascript
if (this.readyState == 4 && this.status == 200) {
    console.log(this.response)			// 打印响应内容
}
```
  第二点要解释的其实是`send()`方法，这个方法可以理解是向服务器请求数据的过程，我们说过了，服务器是一个响应的过程，想到了什么，没错，`send()`是一个异步的方法，加入不是异步的，我们`send()`下面的代码就一点意义也没有。个人认为，`ajax`的异步方式也是体现在这里。我们不用过怎么请求的，我们继续往下进行，到请求结束了，我们获取数据就行了。不会对下面的步骤产生影响。
  基本原理已经清楚了，但是还有小小的不妥。主要是post请求导致的，post，一般我们理解是发送。在网络中，除了发送数据本身（请求体）以外，还需要请求体。所以
send（）方法通过post请求的时候，会带着请求体，
假如：现在要登录，我要发送给服务器的就是我的账户username和密码password，`send()`的格式就是

```javascript
send(`username=${username}&password=${password}`)
// 这种键值对的方式是 urlencoded
```
  在这种情况下，我们必须为请求设置请求头的`Content-Type`，告诉服务器，请求体的格式如下
```javascript
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
```
  到了这里，`ajax`的原理基本了解了，开始使用的时候，觉得有点麻烦，每次ajax那么多代码，当然是封装了。其他的框架或者库都是封装的，如jQuery的ajax，功能强大。

##### 旧版ajax的封装（不推荐）

```javascript
/*
obj       可选参数 
method    请求方式 （支持大小写）
url       请求地址
data      请求体    {}
callback  回调函数
 */

let ajax = (obj) => {
    const method = obj.method.toUpperCase();
    let url = obj.url;
    let data = obj.data;
    const callback = obj.callback;
    // 为了兼容IE浏览器
    const xhr = window.XMLHttpRequest ? new XMLHttpRequest() : new ActiveXObject('Microsoft.XMLHTTP');
    const tempArr = [];
    // 将用户传进来的对象转成urlencoded的格式
    for (key in data) {
        let value = data[key];
        tempArr.push(`${key}=${value}`);
    }
    let params = tempArr.join('&');
   	// get请求需要将数据拼接在URL后
    if (method == 'GET') {
        url = `${url}?${params}`;
    }
    xhr.open(method, url, true);
    data = null;
    // POST 设置requestHeader
    if (method == 'POST') {
        xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
        data = params;
    }
    xhr.send(data);
    xhr.onreadystatechange = function () {
        if (this.readyState == 4 && this.status == 200) {
            callback(this.responseText);
        }
    }
}
```
##### 发送ajax请求

```javascript
btn.onclick = function () {
                ajax({
                    method: 'POST',					//GET
                    url: 'http://localhost:3000/',
                    data: {username: username.value, password: password.value},
                    callback: function (res) {
                        alert(res);
                    }
                })
            }
```

##### 服务端响应ajax

```javascript
const http = require('http');
const url = require('url');
const querystring = require('querystring');

const server = http.createServer((req, res) => {
    res.writeHead(200, {"Content-Type": "text/plain", "Access-Control-Allow-Origin":"*"});
    // 注释的代码是对于get请求的响应
    // let body = url.parse(req.url, true);
    // let parse = querystring.parse(req.url)
    // console.log(body.query);
    // res.end(JSON.stringify(body.query));
    
    // 下面是对于post请求的响应
    let str = '';
    req.on('data', function (data) {
        str += data;
    });
    req.on('end', function () {
        const post_body = querystring.parse(str);
        res.end(JSON.stringify(post_body));
        console.log(post_body);
    });
   
})
server.listen(3000);
```
##### 新版-ajax封装
```javascript
const main = () => {

    const ajax = (options) => {

        // 默认参数对象
        const defaultObj = {
            type: '',
            url: '',
            header: {
                'Content-Type': 'application/application/x-www-form-urlencoded'
            },
            params: {},
            access: (data) => {
                console.log(data);
            },
            error: (err) => {
                console.log(err);
            }
        }

        // 用传递参数覆盖默认参数
        Object.assign(defaultObj, options)

        // 创建实例
        let xhr = new XMLHttpRequest();

        // 拼接参数字符串
        let paramStr = '';
        for (let key in defaultObj.params) {
            paramStr += `${key}=${defaultObj.params[key]}&`
        }

        // 去除最后一个&
        paramStr = paramStr.substring(0, paramStr.length - 1);

        // get请求
        if (defaultObj.type === 'GET') {

            // 设置请求方式及地址
            xhr.open(defaultObj.type, `${defaultObj.url}?${paramStr}`);

            // 发送请求 get请求不传参数 post请求向send()传递参数
            xhr.send();
        } else {

            // 设置请求方式及地址
            xhr.open(defaultObj.type, `${defaultObj.url}`);

            // 请求凡是是post必须设置请求头
            xhr.setRequestHeader('Content-Type', defaultObj.header['Content-Type'])

            // 根据请求头 发送不同格式数据
            if (defaultObj.header['Content-Type'] === 'application/x-www-form-urlencoded') {

                // TODO 请求头设置为application/x-www-form-urlencoded 格式send()中传递必须使用该格式
                // post请求通过send()传递参数
                xhr.send(paramStr);
            } else {
                xhr.send(JSON.stringify(defaultObj.params));
            }

        }
        // 请求成功
        xhr.onload = () => {
            let ContentType = xhr.getResponseHeader('Content-Type');
            let responseText = xhr.responseText;

            // TODO 服务器返回的永远是字符串
            if (ContentType.includes('application/json')) {
                responseText = JSON.parse(responseText);
            }

            // 请求成功
            if (xhr.status == 200) {
                defaultObj.access(responseText, xhr);
            } else {
                defaultObj.error(responseText, xhr);
            }
        }
    }
    ajax({
        type: 'POST',
        url: 'http://localhost:3000/post',
        header: {
            'Content-Type': 'application/json'
        },
        params: {
            name: "wuxue",
            age: 25
        },
    });

}
```

`ajax`的基本原理和基本的封装基本上明白了，我自己也验证了`ajax`请求的可行性，但是大家有没有发现，这里其实是几个不爽的，或者是遗留的问题


1. 假如，我在ajax中得到了服务器响应值之后，进一步的处理是根据响应的数据进行再次的ajax请求呢？多次呢？想一想都觉得可怕！恐怕这就是回调黑洞了。
2. 其实这里还有一个问题，请看服务端的代码。假如去除第五行中`Access-Control-Allow-Origin":"*"`，这一句代码，其实ajax是无法使用的。

**好了，可以开始解决上述的问题了，实践是检验的唯一标准**

