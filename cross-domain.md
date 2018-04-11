### 如何解决跨域问题
---


**JSONP：**

就是动态创建<script>标签，然后利用<script>的src 不受同源策略约束来跨域获取数据。

JSONP 由两部分组成：回调函数和数据。回调函数是当响应到来时应该在页面中调用的函数。回调函数的名字一般是在请求中指定的。而数据就是传入回调函数中的 JSON 数据。


定义回调函数
function handleResponse(response){
    // 对response数据进行操作代码
}

定义api
var script = document.createElement("script");
script.src = "https://api.douban.com/v2/book/search?q=javascript&count=1&callback=handleResponse";
document.body.insertBefore(script, document.body.firstChild);

 JSONP 是从其他域中加载代码执行。如果其他域不安全，很可能会在响应中夹带一些恶意代码，而此时除了完全放弃 JSONP 调用之外，没有办法追究。因此在使用不是你自己运维的 Web 服务时，一定得保证它安全可靠。

其次，要确定 JSONP 请求是否失败并不容易。虽然 HTML5 给<script>元素新增了一个 onerror事件处理程序，但目前还没有得到任何浏览器支持。为此，开发人员不得不使用计时器检测指定时间内是否接收到了响应。


- 原理是：动态插入script标签，通过script标签引入一个js文件，这个js文件载入成功后会执行我们在url参数中指定的函数，并且会把我们需要的json数据作为参数传入
- 由于同源策略的限制，XmlHttpRequest只允许请求当前源（域名、协议、端口）的资源，为了实现跨域请求，可以通过script标签实现跨域请求，然后在服务端输出JSON数据并执行回调函数，从而解决了跨域的数据请求
- 优点是兼容性好，简单易用，支持浏览器与服务器双向通信。缺点是只支持GET请求

- JSONP：json+padding（内填充），顾名思义，就是把JSON填充到一个盒子里

```
<script>
    function createJs(sUrl){

        var oScript = document.createElement('script');
        oScript.type = 'text/javascript';
        oScript.src = sUrl;
        document.getElementsByTagName('head')[0].appendChild(oScript);
    }

    createJs('jsonp.js');

    box({
       'name': 'test'
    });

    function box(json){
        alert(json.name);
    }
</script>
```

**CORS**

- 服务器端对于CORS的支持，主要就是通过设置Access-Control-Allow-Origin来进行的。如果浏览器检测到相应的设置，就可以允许Ajax进行跨域的访问


**通过修改document.domain来跨子域**

- 将子域和主域的document.domain设为同一个主域.前提条件：这两个域名必须属于同一个基础域名!而且所用的协议，端口都要一致，否则无法利用document.domain进行跨域。主域相同的使用document.domain

**使用window.name来进行跨域**

- window对象有个name属性，该属性有个特征：即在一个窗口(window)的生命周期内,窗口载入的所有的页面都是共享一个window.name的，每个页面对window.name都有读写的权限，window.name是持久存在一个窗口载入过的所有页面中的

**使用HTML5中新引进的window.postMessage方法来跨域传送数据**

- 还有flash、在服务器上设置代理页面等跨域方式。个人认为window.name的方法既不复杂，也能兼容到几乎所有浏览器，这真是极好的一种跨域方法


**如何解决跨域问题?**

- jsonp、 iframe、window.name、window.postMessage、服务器上设置代理页面

- 如何解决跨域问题?

  * document.domain + iframe：要求主域名相同 //只能跨子域
  * JSONP(JSON with Padding)：response: callback(data) //只支持 GET 请求
  * 跨域资源共享CORS(XHR2)：Access-Control-Allow //兼容性 IE10+
  * 跨文档消息传输(HTML5)：postMessage + onmessage  //兼容性 IE8+
  * WebSocket(HTML5)：new WebSocket(url) + onmessage //兼容性 IE10+
  * 服务器端设置代理请求：服务器端不受同源策略限制
  
  
  jQuery封装的$.ajax中有一个dataType属性，如果将该属性设置成dataType:"jsonp"，就能实现JSONP跨域了。需要了解的一点是，虽然jQuery将JSONP封装在$.ajax中，但是其本质与$.ajax不一样。
  $.ajax({
                async : true,
                url : "https://api.douban.com/v2/book/search",
                type : "GET",
                dataType : "jsonp", // 返回的数据类型，设置为JSONP方式
                jsonp : 'callback', //指定一个查询参数名称来覆盖默认的 jsonp 回调参数名 callback
                jsonpCallback: 'handleResponse', //设置回调函数名
                data : {
                    q : "javascript", 
                    count : 1
                }, 
                success: function(response, status, xhr){
                    console.log('状态为：' + status + ',状态是：' + xhr.statusText);
                    console.log(response);
                }
            });
        });
  
  
  利用JQuery getJSON来实现，只要在地址中加上callback=?参数
  $.getJSON("https://api.douban.com/v2/book/search?q=javascript&count=1&callback=?", function(data){
                console.log(data);
            });
  