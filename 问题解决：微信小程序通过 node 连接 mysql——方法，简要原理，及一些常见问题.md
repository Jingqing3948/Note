---
title: 问题解决：微信小程序通过 node 连接 mysql——方法，简要原理，及一些常见问题
date: 2023-03-02
tags:
- WXMiniProgram
categories:
- 破浪（遇到的bug及解决方案）
description: 微信小程序通过 node 连接 mysql 问题解决
---

# 前言

博主自己在22年夏天根据课程要求做了一个小程序连接阿里云服务器的案例，在最近又碰到了相应的需求。

原参考文章：[微信小程序 Node连接本地MYSQL_微信小程序nodejs连接数据库_JJJenny0607的博客-CSDN博客](https://blog.csdn.net/m0_56598099/article/details/119105587?app_version=5.14.2&code=app_1562916241&csdn_share_tail={"type"%3A"blog"%2C"rType"%3A"article"%2C"rId"%3A"119105587"%2C"source"%3A"jtwqwq"}&uLinkId=usr1mkqgl919blen&utm_source=app) ,还请多多支持原作者！

第二次尝试的时候已经熟练许多了，但是还是遇到了很多挺烦人的问题，也浪费了很多时间排查 bug。因此博主打算趁着刚做过记忆还深刻的时候，把全过程、博主自己的理解和遇到的一些问题的解决方法整理出来。当然我也不可能能解决所有问题，遇到的部分问题也只是针对我自己的情况可以解决。因此如果 solutions 不起效或遇到其他问题，可以在评论区留言，我会与大家沟通（不保证能否解决~）。

# 原理

![55ff06e4153087c9216fd7b56763ea7](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202505212343956.png)

nodejs 作为中介服务器，可以让小程序连接 mysql 数据库。

# 步骤

如果不出错误的话， 基本流程就是跟着上面的文章来的。

1. 下载 node。可以在 cmd 中输入 `node -v` 查看有无成功下载。

   ![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202505212344379.png)

2. 新建一个数据库，这一部分很简单，有 mysql 相关基础的读者应该没问题，也可以看博主的 mysql 专栏学习。

3. 新建一个小程序，在结构中新建 server 文件夹用于存储 node 本地服务器相关代码。

   ![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202505212344368.png)

4. 右键 server 文件夹，点击“在内建终端中打开”，安装一些所需的包。

```
npm init -y
npm i mysql --save-dev
npm install body-parser --save-dev
npm install express --save-dev
```

5. 在文件夹下新建 server.js 文件，编写代码。

```js
const express=require('express')
const bodyParser =require('body-parser')
const app=express()
const mysql = require('mysql')
const IPAddress='[你的数据库地址]'//因为这里是要链接远程数据库，ip 地址是 mysql 的地址！！本地就是 127.0.0.1，服务器上就自己找找看
const UserName='[mysql 用户名]'
const PWD='[mysql 密码]'
const DBName='[要操作的数据库名]'
app.use(express.json())
app.use(bodyParser.json())
app.use(bodyParser.urlencoded({ extended: false }))
//这上面一段固定的，[] 的部分需要根据自己数据库的信息修改



//处理get请求。这里是一个 get 请求的方法演示，作用是查询 table1 表中的所有数据并返回。
app.get('/getUser',(req,res)=>{ //这里的是 get 方法 getUser，对应了刚才的页面发来的请求。就会执行这个方法。
  //参数传入是在 req.body 对象里面。比如上面的语句是获取传入的 openid 变量，并且我们新定义一个叫 openid 的变量存储传入的 openid 变量
  var connection=mysql.createConnection({
    host:IPAddress,
    port: 3306,		//端口号，mysql 固定3306
    user:UserName,
    password:PWD,
    database:DBName
  })//配置连接的属性
  connection.connect();//尝试连接
  connection.query("select * from table1",function(error,results,fields){//执行查找语句
    if(error) console.log(error);//执行失败的话
    res.json(results)
    console.log(results)
    
  })
  connection.end();//断开连接
  
})

app.listen(3000,()=>{//这是一个监听端口，会输出监听到的信息。上面的 console.log 就会在这里输出
  console.log('server running at http://'+IPAddress+':3000')
})

```

我们的 nodejs 是部署在本地的，就在 server 文件夹里。node_modules 是下载的 nodejs 包，server.js 是我们的操作 nodejs 的代码。

我们在需要调用数据库的页面里先调用本地服务器，比如在 mysqlTest.js 中，onLoad 里：

```js
onLoad() {
    //我们写一个调用数据库函数，使得本页面加载时调用此函数，查找 youqi.data 里的所有条目。
        wx.request({
            method: 'GET',//这里要和 server.js 定义的 post or get 一致！！！
            url: 'http://[本地 IPV4 地址]:3000/getUser',//这里的 ip 地址不是数据库的地址，而是你的电脑本地的地址，因为这一步的操作是要找到本地 nodejs 服务器。getUser 要和 server.js 中定义的方法名一致。
            data: {
              //这里面是传入参数。比如我们要 select * from data where openid= 给定的 openid，就可以从这里传入
            },
            success: function (res) {//成功获取到值，返回一个 res 对象。如果不知道 res 对象里面包含什么，可以先输出 res 对象看一下其中都包含什么
              console.log(res);
            },
            fail: function () {//没有获取到值，说明这中间出问题了。
              console.log("获取失败");
            }
          })
    },
```

本地 IPV4 地址是怎么查出来的？打开 cmd，输入 ipconfig。无线局域网适配器 WLAN 里的 IPV4 即是。

nodejs 接收到了请求，就会执行 server.js 里对应的方法。

# 如何运行

首先 **nodejs 服务器需要手动开启**，我还不会自动开启。

右键 server 文件夹，在内建终端中打开，然后在终端中输入：

![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202505212344060.png)

下面那句话说明正常启动服务器并且开始监听。

我们输出一下 res，看看成功会返回什么信息。调试器里的 console 输出：

![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202505212343123.png)

data 里是我们数据库的查询结果。

![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202505212343704.png)

res.data 获取这个15长度的 Array。

![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202505212343282.png)

然后比如我们要获取其中的 time 值，就是 res.data[0].time, res.data[1].time... res.dat[14].time。

{} 表示其中是一个对象变量，通过句号+索引名称获取特定的值。

而刚才的一直在监听的内建终端里输出：

![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202505212344070.png)

这个的输出对应 function(error,results,fields) 里的 console.log(results)。如果查询失败，error 也是在这个内建终端里输出。

# 常见问题

## 数据库无法连接？

建好数据库后，最好就先用 cmd 或 workbench 连接一下试试。如果能成功连接并查询，之后出问题也可以排除 mysql 的一部分问题。

如果采用云服务器（如阿里云）连接不上，可能的问题有：

- 服务器端口 80 和 3306 没开。（不建议一键全部放行，有被hacker get 的风险）

- mysql 需要有一个用户能在所有 ip 地址里对该数据库进行读写操作，这里可以参考 DCL 的内容来新建符合要求的 user。[Mysql_9 SQL 语句——DCL_sql语言dcl_灰海宽松的博客-CSDN博客](https://jingqing3948.blog.csdn.net/article/details/124430926)

- 服务器防火墙的问题，要开启3306端口。

  ![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202505212343761.png)

## 小程序报错 ERR TIMEOUT

打开网站可以看到 404 显示。

- 首先先确保前面那个问题确实解决了，该数据库确实可以从本地连接，，不是数据库的问题。

- 再确认 get post 方法是否对应，方法名是否写错。很多时候输入了错误的链接也会显示 404.

- 小程序不校验合法域名是否打开。

- 启用或关闭 windows 功能——Internet Information Services，及其可承载 web 核心，**子项**全部打开。点击确定。如果成功启动，在浏览器中输入 127.0.0.1，应该可以看到：

  ![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202505212343506.png)

- 检查方法名，get post 类别是否写错。