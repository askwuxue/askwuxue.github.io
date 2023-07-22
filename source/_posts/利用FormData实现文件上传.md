---
title: 使用Ajax结合FormData对象实现文件上传
date: 2021-03-16 23:07:14
tags: Node
categories: Node
---

### 使用`Ajax`结合`FormData`实现文件上传
_使用`FormData`之前，我们需要知道它的作用，或者说是本质吧。`FormData`本质上是`HTML`提供的一个对象，可以模拟HTML表单，相当于将HTML表单映射成表单对象，自动将表单对象中的数据拼接成请求参数的格式。用一个对象来代替正常的`FORM`表单提交，但是又比表单更强大。_

#### 1 `FormData`使用方法

##### 1.1 创建一个表单对象

因为`FormData`对象不能脱离`form`元素使用，所以我们必须先有一个表单。

**注意：创建表单，表单中的元素，如input标签等，必须有name属性，因为`FormData`对象要根据`name`属性来获得对应的值，或者是操作对应的值。下文会有详细介绍**

```html
    <form id="form" name="form">
        <input type="text" name="username">
        <input type="password" name="password">
    </form>
```

##### 1.2 创建一个`FormData`实例，将表单转化为`FormData`对象

```ts
// 使用FormData构造函数创建一个实例，注意该构造函数接受一个参数，参数必须是一个表单对象。如下：
// 获得表单
1. let form = document.getElementById('form');
// 将表单转化为Form对象，此时，FormData其实就是包含了表单form内容的一个对象。具体的形式为{key, value}的形式，key就是form表单的name属性值，因此，form中的控件必须有name属性。
2. let formData = new FormData(form);
```
**注意: 完成了上面步骤后，如果此时你在开发者工具尝试输出`FormData`对象，输出会是一个空对象，因为`FormData`是一种特殊格式，无法输出。**


##### 1.3 使用`Ajax`发送`FormData`
**创建的`FormData`对象，使用`Ajax`中的`send()`进行发送，`FormData`对象只能使用`send()`进行发送，不能使用`get`请求的`url`拼接参数的方式。**
如下，客户端使用Ajax进行了一次post请求。

```ts
let xhr = new XMLHttpRequest();

xhr.open('POST', 'http://localhost:3000/FormData');

// TODO 对于FormData对象，不能设置常规的Content-Type
// xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');

xhr.send(formData);

xhr.onload = () => {
	console.log(xhr.responseText);
}
```
Node服务端进行响应（使用express框架，formidable第三方包）

```ts
// 路由
app.post('/FormData', (req, res, next) => {
	// form 使用formidable的一个实例，用来接受处理FormData的请求，fields参数是FormData的内容。
    form.parse(req, (err, fields, files) => {
    	console.log(fields)    //{"username":"123","password":"123"}
        // 返回结果给客户端
        res.send(fields);
        next();
    })
})
```

_如上，`FormData`对象的基础对象就结束了，`FormData`对象上有很多方法，请自行查阅文档学习，我们本次文件上传只用到了append()方法，语法为append(name, value)。另一些说明在代码中进行了详细注释_

**完整的实现代码**

**html**

```html
<!-- 容器 -->
    <div class="container" style="overflow: hidden;">
    <!-- 进度条 -->
        <div class="progress" style="margin: 20px auto;">
            <div id="progress" class="progress-bar progress-bar-striped progress-bar-animated" role="progressbar" aria-valuenow="75" aria-valuemin="0" aria-valuemax="100" style="width: 0"></div>
        </div>
         <!-- 空的form表单 -->
        <form id="form" name="form">
        </form>
         <!-- 文件上传控件 -->
        <input type="file" id="upload-file" name="file">
         <!-- 进行文件上传成功后的图片动态显示显示 -->
        <div class="avatar">
        </div>
    </div>
```
**JavaScript**
```javascript
<script>
        // 获得form表单
        let form = document.getElementById('form');
        let file = document.getElementById('upload-file');
        // 存放动态渲染图片的区域
        let avatar = document.querySelector('.avatar');
        // 控制进度显示
        let progress = document.getElementById('progress');
        // 创建FormData实例
        let formData = new FormData();
        // TODO 事件处理函数内部需要使用this时，不要使用箭头函数 导致this丢失
        function handleUploadFile() {
            let xhr = new XMLHttpRequest();
            // TODO 文件上传this.files是一个数组 存储了选择的文件
            // 传递的应该是对应的文件如第一个文件就应该是此处的this.files[0]
            // 将选择的文件添加到FormData对象中
            formData.append('file', this.files[0]);
			// 发送Ajax请求
            xhr.open('POST', 'http://localhost:3000/upload');
            // 将XMLHttpRequest.upload.onprogress 放到send()方法后是不生效的
            xhr.upload.onprogress = function(ev) {
            	// loaded 已加载内容  total 总共的内容
                let per = (ev.loaded / ev.total) * 100 + '%';
                progress.style.width = per;
            }
            // 发送FormData表单 
            xhr.send(formData);
            // 提前知道服务端返回的是图片的地址，所以此处声明一个变量，接受返回地址
            let imgPath = '';
			// 服务端返回
            xhr.onload = () => {
                // 服务端返回的数据格式为json字符串 需要进行反序列化
                let responseData = JSON.parse(xhr.responseText);
                // 利用服务端返回的信息 利用\upload\进行字符串切割 取得文件路径
                imgPath = responseData.path.split('\\public\\')[1];
				// 请求返回成功
                if (xhr.status === 200) {
                	// 动态创建一个img标签
                    let img = document.createElement('img');
                    // 图片的地址给img添加
                    img.src = imgPath;
                    // 图片加载完成
                    img.onload = function() {
                        // 将img标签添加到DOM中
                        avatar.appendChild(img);
                    }
                }
            }
        }
        file.addEventListener('change', handleUploadFile);
    </script>
```
**服务端处理**
```javascript
const express = require('express');
const port = 3000;
const public = express.static('public');
const formidable = require('formidable');
const form = new formidable.IncomingForm();
const path = require('path');
// 设置文件上传的路径
form.uploadDir = path.join(__dirname + '/public' + '/upload');
// 默认文件上传后会去除后缀名。开启保留文件后缀名
form.keepExtensions = true;
const app = express();
app.use('/', public);

// 文件上传   主要看这里就可以了
app.post('/upload', (req, res) => {
    form.parse(req, (err, fields, files) => {
        //files.file是一个对象存放了接收到的文件的相关信息
        // 文件在服务器端的地址 
        let path = files.file.path;
        // 默认对象会以json的格式返回 
        res.send({ path: path });
    })
})

app.listen(port, () => {
    console.log('express start access port is 3000')
})
```
**以上就是利用`Ajax`结合`FormData`对象实现的简易版的文件上传功能。实践出真知。虽然看着很简单，但是在完成的过程中，遇到了各种各样的小问题。慢慢的Google，慢慢的看文档，思考。收获的还是蛮多的。一起动手试试吧**