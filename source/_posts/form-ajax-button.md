---
title: Ajax提交表单的一些bug
tags:
  - Ajax
  - form
  - bug
categories:
  - 前端
abbrlink: 18896
date: 2022-03-10 14:19:19
---



# 关于使用Ajax提交表单后页面异常加载和报错问题



## 1、bug出现场景

本想要实现表单的异步多文件上传功能，通过点击按钮调用ajax方法，然后与后端进行通信。

html代码：

```html
<form id="test_form">
    <input type="text" id="txt" value="111">
    <button onclick=test_form()>点击测试</button>
</form>
```

js代码：

```javascript
<script>
        function test_form() {
            $.ajax({
                type: 'post',
                url: '/test',
                data: 'aaa',
                dataType: "json",
                success: function (res) {
                    alert('success')
                },
                error: function (err) {
                    alert('error')
                }
            })
        }
</srcipt>
```

后端Java代码：

```java
@PostMapping("test")
@ResponseBody
public String Test() {
    System.out.println("测试表单提交...");
    return "sss";
}
```

预计结果应该是弹出“success”，可结果却是”error“，并且浏览器地址也会发生改变，可实际上我并没有设置form的action属性。而且控制台也会输出`“测试表单提交...”`字样，说明后台方法是已经执行了的。

![image-20220311150130064](https://gitee.com/qingy735/blogimg/raw/master/img/202203111501339.png)



## 2、分析原因

既然后台也执行了，那么为什么会出现这种情况？后来我又测试了直接通过按钮调用ajax请求而不通过表单，奇怪的事情发生了，可以完美的运行程序，并且地址栏也不会发生改变。于是猜测会不会是表单会在提交ajax请求后会自己再提交一次请求，于是我就在表单`action`属性添加了地址，并且将后台方法设置为`get`。

```html
<form id="test_form" action="/test2">
    <input type="text" name="txt" value="111">
    <button onclick=test_form()>点击测试</button>
</form>
```

```java
@GetMapping("test2")
@ResponseBody
public String Test2() {
    System.out.println("测试表单提交2...");
    return "sss2";
}
```

然后奇怪的事情又发生了，运行程序后地址栏会发生这样变化：

![image-20220311151009967](https://gitee.com/qingy735/blogimg/raw/master/img/202203111510068.png)

看到这样的变化似乎找到了原因。我们都知道`?`代表的是`get`方法传递参数，同时我ajax方法传递的就是表单中的`txt`数据，而`/test2`则是表单`action`数据。那么就代表表单进行了两次提交，一次是`/test`，另一次是`/test2`。这就代表`button`在表单中可能默认代表`submit`。于是百度查了一下，果然`button`在表单中默认为`type=submit`，所以会提交两次，如果想要只提交一次的话就设置`type=button`，确实有点我非我我感觉。



## 3、总结

尽量使用`input`并设置`type=button`进行表单提交，这样可以避免不必要的麻烦和bug，同时记住`button`也是有`type`属性的，并且在表单中默认为`submit`。（纯粹有种脱裤子放屁的感觉，button不再是button还得设置`type=button`，我非我了属于是
