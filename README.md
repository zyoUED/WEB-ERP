# WEB-ERP

### 一、开发工具(任选一个)
* Webstorm
* Sublime Text
* Atom

### 二、开发前的准备
1. css+html
2. javascript
3. [Express](http://www.expressjs.com.cn/)
4. [Vue.js (1.x)](http://vuejs.org.cn/guide/installation.html)

### 三、ERP文件说明和结构
##### 说明：
此项目结构采用java+node.js+html/html5

原本java那边只提供rest接口，其他逻辑由node层来处理，目的是后期其他端的接口可以由node端来处理，java只出一套就好了，但是，由于某种神奇的原因，变成了java的开发人员把所有的逻辑都处理了并且出了接口（在此非常感谢java的开发人员）。

node端在crud.js中用request模块处理了java端的接口。

>var request = require('request')


```
get:function( receiveurl,callback){
        request({ url: receiveurl, method: 'get'}, function(error,response,body){
            callback(error,response,body);
        })
}
```

***
##### 结构：
 ![erp1.png](https://rainagain.github.io/images/erpimages/erp1.png)

* ERP文件夹中是项目的代码文件


1. .gitignore是git上传忽悠文件配置
2. config.json是docker在服务器上的配置文件
3. Dockerfile同是docker在服务器上的配置文件


![erp2.png](https://rainagain.github.io/images/erpimages/erp2.png)

* 基本结构是用express生成器生成的（添加了controller和models文件夹）

![erp3.png](https://rainagain.github.io/images/erpimages/erp3.png)

* bin是启动目录，在此不做说明了


1. <font color=#00ff00>controller </font>中<font color=#ff0000>common.js</font>和<font color=#ff0000>crud.js</font>文件用来处理java后台写的接口，并返回前端要用的接口
2. <font color=#00ff00>models </font>中<font color=#ff0000>interface.js</font>和<font color=#ff0000>rest.js</font>文件用来存储java端的接口（其他文件暂且不管，是提供给移动端和boss后台用的接口）
3. <font color=#00ff00>public </font>是Express生成器生成的用来存储js和css可供前端页面使用的文件夹
4. <font color=#00ff00>routes </font>是Express生成器生成的用来定义前端页面路由的文件夹
5. <font color=#00ff00>views </font>是Express生成器生成的用来放前端html页面的文件夹
6. <font color=#ff0000>app.js </font>是用来启动项目的文件 在项目对应的cmd中`node app.js`
7. <font color=#ff0000>config.js </font>设置系统配置参数
8. <font color=#0000ff>package.json </font> 本来是项目中所依赖用到的模块信息，但此项目由于开始的时候，技术不成熟故没有考虑到，所以，在这个项目中，目前是作废了，只能从公司的gitlab全都拉下来



### 四、ERP开发及使用

整个项目请求的接口分三种

* rest接口，即java后端用restful生成的接口（处理一张表的）

* erp接口，即java后端自己写的接口（带有逻辑处理，且大多数关联两张表及以上的）

* 代理的接口
---
####  项目98%的数据是通过jQuery的ajax方式来请求的（处理rest接口和erp接口）



比如启动后访问 http://localhost:5500/system/operator/maintenance

这里有个路由匹配的知识点，不在此多做说明，看官网文档比我说的清楚（[Express+Router](http://www.expressjs.com.cn/4x/api.html#router)、[相关router知识点这里](http://www.expressjs.com.cn/guide/routing.html)）


这里的数据是由ajax请求来的

```
showOperatorInfo: function (flag) {
    var self = this;
    if(flag){
        self.page = 0;
    }
    var url = "/erp/showOperatorInfo?storeId=" + self.storeSelect + "&inputCode=" + self.queryInputCode + "&name=" + self.queryName + "&page="+self.page+"&size="+self.size;
    $.ajax({
        type: 'get',
        url: url,
        dataType: 'json',
        success: function (msg) {
            if (msg.code == 200) {
                self.contents = msg.content.data.content;
                var page = {
                    size: msg.content.data.size,
                    totalElements: msg.content.data.totalElements,
                    totalPages: msg.content.data.totalPages,
                    number: msg.content.data.number
                };
                self.reqpage = page;
            }
            self.queryCheck=[];
        }
    })
}
```

代码中的url为`"/erp/showOperatorInfo"`

![crud.png](https://rainagain.github.io/images/erpimages/crud.png)

如上图，ajax在进行get请求的时候express.Router的router模块会响应客户端的请求。这里的路由路径用了正则，所以可以匹配所有`/erp/xxxx`开头的请求。（[相关router知识点这里](http://www.expressjs.com.cn/guide/routing.html)）

##### 说明:


>由于开始的时候沟通不全及需求不明确
>
1.项目中有大量无用代码，如上图中的接收put请求和delete请求，在此处是无用的。
>
```
CommonFun.Interfaceful.get(url,function(error,response,body){
    try{
        if(!error && response.statusCode == 200){
            res.send({
                code:response.statusCode,
                content:JSON.parse(body)
            });
        }
    }catch(e){
        res.send({
            code: response.statusCode,
            content:'网络出错'
        });
    }
})
```
2.如上代码所示，此处在处理erp接口的时候，封装了一层返回的code值和content，是因为，一开始只出了rest接口，而rest接口是没有返回code：200这种状态码的，故node这层来进行了封装，由于沟通不畅，且前期开发过程中有发现部分接口也没有返回状态码，故在处理erp接口的时候，也进行了同样的封装，然，后期各方面说要去掉这层包装，但是前端已经有大量接口调用了，所以后期也没有改

##### 注意：
1. 在处理ajax返回的数据的时候大多数要如下代码一样的处理
```
if (msg.code == 200) {
   if (msg.content.code != 200) {
        $.dialog_alert(msg.content.description);
    } else {
        //处理
    }
} else {
    $.dialog_alert('错误码：' + msg.code);
}
```

2. 在ajax进行post请求的时候需要把数据进行JSON.stringify(data),如下代码所示，原因是后台处理不了json对象的数据（深表无奈0.0）
```
$.ajax({
    url: url,
    type: 'POST',
    data: JSON.stringify(data),
    contentType: 'application/json',
    dataType: 'json',
    success: function (resp) {
        console.log(resp);
        if (resp.content.code == 200) {
            $.dialog_alert("保存成功！");
        } else {
            $.dialog_alert("错误码：" + resp.content.code);
        }
    }
});
```

#### 代理的接口

可以打开库存查询页面http://localhost:5500/stock/stockmanage/stock-query

里面有个导出功能的方法，方法中有如下代码，就是用来到处excel文件的，excel文件是后台生成的，不需要前端进行任何处理。
```
var url = "/erp/1/stock/stockManage/stockExcel?storeId="+self.selectData.storeId+"&depotId="+self.selectData.depotId+"&inputCode="+self.selectData.inputCode
                +"&beginTime="+beginTime+"&endTime="+endTime+"&licenseNumber="+self.selectData.licenseNumber
                +"&isPrescription="+self.selectData.isPrescription+"&page="+self.page+"&size="+self.size + "&beginPrice=" + self.selectData.beginPrice
                +"&endPrice=" + self.selectData.endPrice + "&beginQuantity=" + self.selectData.beginQuantity + "&endQuantity=" + self.selectData.endQuantity
                +"&beginValidity="+self.selectData.beginValidity+"&endValidity="+self.selectData.endValidity;
window.location = url;
```
##### 注意

这边的地址，也会去请求erp接口，然，我们上面所说的接收erp请求的处理方式并不能处理故，我们在处理这种接口的时
