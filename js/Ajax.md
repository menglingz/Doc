# Ajax

### 1.ajax的使用步骤

- 创建`XMLHttpRequest`对象


- 使用`open`方法设置和服务器交互的信息

- 使用`send`方法发送数据，开始和服务器交互 

- 注册事件

-  更新界面

  ```js
  // 1、创建XMLHttpRequest对象
  var xhr = new XMLHttpRequest()
  // 2、使用open方法设置和服务器交互的信息
  xhr.open('get', './data/data.json', true)
  // 3、设置发送的数据，开始和服务器交互 
  xhr.send()
  // 4、注册事件
  xhr.onreadystatechange = function () {
    // ajax 
    if (xhr.readyState === 4) {
      // 服务器
      if (xhr.status === 200) {
        console.log(xhr.responseText);
      }
    }
  }
  ```

### 2.状态码

##### （1）ajax的状态

- 0：初始化，还没有调用`open()`方法
- 1: 载入 已经调用`send()`方法，正在发送请求 
- 2：载入完成，`send()`方法已经完成，已经收到全部响应内容
- 3: 解析 正在解析响应的内容 
- 4: 完成，响应内容解析完成，可以在客户端调用了

##### （2）服务器的状态

- 1 消息
- 2 成功 -- 200 
- 3 重定向
- 4 请求错误
- 5 服务器错误

### 3.get请求

- `get`方式要给后台传送数据， 把数据放在地址的后面，使用 `？`的方式进行链接，也就是数据要在 `？` 后面数据是以键值对的形式存在 `http://localhost/062ajax/2-get%e6%96%b9%e5%bc%8f.html?name=zhangsan&age=20`

  ```js
  var xhr = new XMLHttpRequest()

  xhr.open('get', './data/data.jsonname=zhangsan&age=20', true)

  xhr.send()

  xhr.onreadystatechange = function () {
    // ajax 
    if (xhr.readyState === 4) {
      // 服务器
      if (xhr.status === 200) {
        console.log(xhr.responseText);
      }
    }
  }
  ```

### 4.post请求

- `pos`t方式要给后台传送数据，需要把数据当做`send()`方法的参数传递数据是以键值对的形式存在  `name=zhangsan&age=20`，注意：如果是`post`需要传递数据，一定在发送之前  去设置请求头`(setRequestHeader)`

  ```js
  var xhr = new XMLHttpRequest()

  xhr.open('post', './data/data.json', true)

  // 是用来声明传递的数据格式的
  xhr.setRequestHeader('Content-Type','application/x-www-form-urlencoded')
  xhr.send('name=zhangsan&age=20')

  xhr.onreadystatechange = function () {
    // ajax 
    if (xhr.readyState === 4) {
      // 服务器
      if (xhr.status === 200) {
        console.log(xhr.responseText);
      }
    }
  }
  ```

### 5.JSON方法

- `JSON.parse()` ：把字符串对象格式的数据转换成json对象格式的数据


- `JSON.stringify()` ：把json对象格式的数据转换成字符串对象格式的数据

### 6.setRequestHeader（）

- setRequestHeader方法：用于设置HTTP请求头信息

  ```js
  xhr.setRequestHeader('Content-Type','application/x-www-form-urlencoded')
  ```


- content-type，传输数据类型，即服务器需要我们传送的数据类型
- 常见的值有以下几种格式：
  - `form`表单类型 ：浏览器的原生`form`表单`application/x-www-form-urlencoded`
  - 使用表单上传文件时：必须让 `form` 的 `enctype` 等于这个值
    `multipart/form-data`
  - 列化后的 `JSON` 字符串：`application/json`
  - XML 作为编码方式的远程调用规范：`text/xml`