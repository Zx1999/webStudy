## Ajax 列表渲染案例

> 使用场景：局部更新网页，包括邮箱、地图、上拉刷新、下拉加载、用户名是被注册等等；前后端分离式项目开发
> 知识点：网络通信协议、HTTP协议、跨域操作

### 一 环境搭建

#### 1. HTTP

> HTTP协议可以说规范了浏览器与服务器之前通信的规则，包括怎样请求、怎样相应

**1.1 GET/post请求**

| | GET | POST |
| -- | -- | -- |
| 目的 | GET从指定的资源请求数据 | POST向指定的资源提交要被处理的数据 |
| 请求参数 | GET请求参数是通过URL传递的，多个参数以&连接 | POST请求放在request body中 |

**1.2 响应状态码**  

| 分类 |	分类描述 |
| -- | -- |
| 1xx | 信息，服务器收到请求，需要请求者继续执行操作 |
| 2xx | 成功，操作被成功接收并处理 | 
| 3xx | 重定向，需要进一步的操作以完成请求 |
| 4xx | 客户端错误，请求包含语法错误或无法完成请求 |
| 5xx | 服务器错误，服务器在处理请求的过程中发生了错误 |

#### 2. 同步(Sync) 异步(Async)

##### 2.1 同步/异步

**同步**：所谓同步，就是发出一个功能调用时，在没有得到结果之前，该调用就不返回或继续执行后续操作。
> 例如：B/S模式中的表单提交，具体过程是：客户端提交请求->等待服务器处理->处理完毕返回，在这个过程中客户端（浏览器）不能做其他事。

**异步**：异步与同步相对，当一个异步过程调用发出后，调用者在没有得到结果之前，就可以继续执行后续操作。
> 例如：B/S模式中的ajax请求，具体过程是：客户端发出ajax请求->服务端处理->处理完毕执行客户端回调，在客户端（浏览器）发出请求后，仍然可以做其他的事。

当异步调用完成后，一般通过状态、通知和回调来通知调用者。对于异步调用，调用的返回并不受调用者控制。
对于通知调用者的三种方式，具体如下：

- 状态：即监听被调用者的状态（轮询），调用者需要每隔一定时间检查一次，效率会很低。
- 通知：当被调用者执行完成后，发出通知告知调用者，无需消耗太多性能。
- 回调：与通知类似，当被调用者执行完成后，会调用调用者提供的回调函数。

**由此可见，同步和异步的区别在于：请求发出后，是否需要等待HTTP的响应结果，才能继续执行其他操作。**

##### 2.2 同步/异步与阻塞/非阻塞

**阻塞调用**是指调用结果返回之前，当前线程会被挂起。调用线程只有在得到结果之后才会返回。   

**非阻塞调用**指在不能立刻得到结果之前，该调用不会阻塞当前线程。

> 同步/异步关注的是消息通知的机制，阻塞/非阻塞关注的是程序（线程）等待调用结果（消息通知，返回值）时的状态。

如果“同步”是发起了一个调用后， 没有得到结果之前不进行后续操作， 那它毫无疑问就是被“阻塞”了（即调用进程处于 “waiting” 状态）。
如果“异步”调用发出了以后没有等待后续结果而是继续进行后续操作‘， 毫无疑问， 这个进程没有被“阻塞”。
所以说，在进程通信层面， 阻塞/非阻塞， 同步/异步基本是同义词。

但是在 IO 系统调用层面（ IO system call ）层面， 非阻塞 IO 系统调用 和 异步 IO 系统调用存在着一定的差别，它们都不会阻塞进程，但是返回结果的方式和内容有所差别， 但是都属于非阻塞系统调用（ non-blocing system call ）。想深入了解，参考：https://www.zhihu.com/question/19732473/answer/241673170

##### 2.3 异步与多线程

异步是目的，而多线程是实现这个目的的方法。

#### 3 koa搭建后端环境

快速搭建后端环境来响应前端发起的http请求
执行代码 ```npm run dev```

### 二 初始渲染

#### 1 Ajax

##### 1.1 Ajax基本框架

- 创建xhr对象
- 监听请求是否完成 onload
- 发出HTTP请求 open连接 send发送

##### 1.2 Ajax请求操作

- get请求 将请求体通过**查询字符串参数**的方式放在**url**中： ```xhr.open('GET', 'http://localhost/list?type=phone&count=20', true)```
- post请求 默认以纯文本的方式发送，可以通过setRequestHeader设置键值对类型

##### 1.3 Ajax响应操作

在onload中使用
- responseText 响应字符串
- responseXML 响应XML文档
- responseText 响应字符串类型的json时，使用```JSON.parse(json)```转成对象类型的json 

#### 2 响应JSON到页面

- DOM方式
- 拼接模板字符串方式（逻辑处理和视图层混杂） ES6新特性
- 前端模板引擎（art-templete）
- Vue、React框架

#### 3 发送JSON

post请求 通过setRequestHeader进行JSON数据的发送，并且需要通过```JSON.stringify()```转成字符串类型的JSON

#### 4 API接口文档

接口说明、请求方式、请求URL、请求参数、返回参数等

#### 5 初始化渲染数据

1. 前端：根据接口API向后端传递请求
```javascript
	xhr.open('POST', '/list', true);
	xhr.setRequestHeader('Content-Type', 'application/json');
	xhr.send(JSON.stringify({"page": 0, "count": 10}));
```

2. 后端：根据接口API设置字段及其数据类型，将前端传递过来的参数与之比较
```javascript
  var args = [
    {field: 'page', type: 'number'},
    {field: 'count', type: 'number'},
  ];

  var body = ctx.request.body;

  for(var i = 0; i < args.length; i++) {
    var item = args[i];
    if(!Object.keys(body).includes(item.field)) {
      ctx.body = {
        errcode: -1,
        errmsg: '参数字段错误'
      };
      return;
    }
    else if(typeof body[item.field] != item.type) {
      ctx.body = {
        errcode: -2,
        errmsg: '参数类型错误'
      };
      return;
    } 
  } 
```

3. 后端：对请求进行处理，返回给前端数据
```javascript
  var data = fs.readFileSync('./data/list.json');
  data = JSON.parse(data);
  var list = data.splice(body.page*body.count, body.count);

  ctx.body = {
    errcode: 0,
    errmsg: 'ok',
    list: list      
  };
```
其中使用 `const fs = require('fs');`载入fs模块，用来对文件进行操作

4. 前端：接收到后端返回的数据后对界面进行渲染
```javascript
xhr.onload = function() {
    if(xhr.status == 200) {
        // console.log(xhr.responseText);
        var data = JSON.parse(xhr.responseText);
        if(data.errcode == 0) {
            sportsList.innerHTML = template('tpl-sportsList', data);
        }
    }
}
```
这里使用art-template模板引擎
```javascript
<script id="tpl-sportsList" type="text/html">
    {{each list}}
    <li>
        <div class="sports-list-text">
            <p>
                {{$value.title}}
            </p>
            <p>
                <span>{{$value.comment}}评</span>
            </p>
        </div>
        <div class="sports-list-img">
            <img src="{{$value.img}}" alt="">
        </div>
    </li>
    {{/each}}
</script>
```
art-template模板使用`<script src="javascripts/template-web.js"></script>`进行导入
