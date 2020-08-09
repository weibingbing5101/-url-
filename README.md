### 图解输入url到浏览器渲染完成发生了什么

一、导航流程

    1 浏览器进程接收到用户输入
        搜索内容：旧页面替换前 beforeunload
        url：自动加协议 https ，  通过IPC 把url请求发送到网络进程


    2 网络进程中发起真正的URL请求
        检查本地是否有缓存
        DNS解析，获取IP地址 (https 建议TLS连接)
        利用IP建立TCP连接(三次握手，三包确认，四次挥手)，http请求：构建请求行、头(cookie content-type等)
        服务器响应 生成响应头、行、体，发送给网络进程，
            

    3 网络进程接收到响应头数据，便解析响应头数据，并将数据转发给浏览器进程
        接收后解析响应头内容：(重定向 301 302 再发起http请求)  200继续处理请求
        响应数据处理：根据 Content-Type 做出不同的处理并转给浏览器进程


    4 浏览器进程接收到网络进程的响应头数据之后，发送“提交导航” 消息到渲染进程


    5 渲染进程接收到“提交导航”的消息之后，便开始准备接收HTML数据，接收数据的方式是直接和网络进程建立数据管道
        

    6 最后渲染进程会向浏览器进程“确认提交”，这是告诉浏览器进程：“已经准备好接受和解析页面数据了”


    7 浏览器进程接收到渲染进程“提交文档”的消息之后，便开始移除之前旧的文档，然后更新浏览器进程中的页面状态



二、渲染流程上

    1、渲染进程 将 HTML 生成 DOM
        html标签转换为DOM树结构放在内存中，供js操作使用  (因为浏览器不能直接理解使用html)
    
    2、样式计算(Style)
        1 渲染引擎接收到css文本时，将其转换为浏览器可以理解的 styleSheets结构
            控制台输入 document.styleSheets
        2. 转换样式表中的属性值，使其标准化
            em => px     blue => rgba      bold => 700
        3. 计算出 DOM 树中每个节点的具体样式
            继承 层叠    默认：UserAgent
    
    3、创建布局树(Layout)  计算元素的布局信息
        计算出 DOM 树中可见元素的几何位置，这个计算过程叫做布局
        1 创建布局树  浏览器展示用
            遍历内存中的DOM树，把可见节点添加到布局树中

        2 布局计算
            有了完整的布局树，计算其树节点的坐标位置，重写至布局树  (新一代布局系统layoutNG 将输入和输出分开)


三、 渲染流程下

    1、 对布局树进行分层(生成图层树 Layer)
        特定节点生成图层(z-index 3D变换 页面滚动)，无对应图层，属于父级图层
        
        如下两点会生成图层
            1 拥有层叠上下文属性的元素提升为单独一层(定位，滤镜、透明)
            2 需要裁剪的被单独创建图层(文字超出，显示滚动条)


    2、 为每个图层生成绘制列表(paint)，并将其提交到合成线程
        每个图层循依次，按各个绘制指令绘制(背景、前景、边框有不同指令)
        生成 绘制顺序和绘制指令列表 提交给  渲染引擎的合成线程

    
    3、 合成线程将图层分成图块(tiles)，并在光栅化线程池中将图块转换成位图
        合成线程会按照视口附近的图块来优先生成位图，
        实际生成位图的操作是由栅格化来执行的。
        
        所谓栅格化(raster)，是指将图块转换为位图

        栅格化 --- 一般都会使用GUP加速生成位图，存在GPU内存中，在GUP中完成。



    4、 合成线程发送绘制图块命令 DrawQuad 给浏览器进程
        所有图块都被光栅化后，合成线程生成DrawQuad提交给浏览器进程的viz组件


    5、 浏览器进程根据 DrawQuad 消息生成页面，并显示到显示器上
        根据DrawQuad命令，将页面绘制到内存中，现实屏幕上


四、以上设计知识点

    1、 重排
        JavaScript 或者 CSS 修改元素的几何位置属性，如 宽 高 
        重排需要更新完整的渲染流水线，所以开销也是最大的

    2、 重绘
        JavaScript 更改某些元素的背景颜色
        重绘省去了布局和分层阶段，所以执行效率会比重排操作要高一些

    3、 直接合成阶段
        transform 来实现动画效果，这可以避开重排和重绘阶段，直接在非主线程上执行合成动画操作

    4、打开一个页面为什么会有4个进程
        1 浏览器主进程：用户交互，子进程管理，文件存储
        2 网络进程：面向渲染进程、浏览器进程等提供网络下载
        3 GPU进程：栅格化(图块转换为位图)
        4 渲染进程：把下载的 html css js 图片 等资源解析为可以显示和交互的页面。(渲染进程的代码从网上获取，所以不被信任，放到沙箱中执行)

        5 插件进程(看页面是否有)
        
        拓展问题：
        不同标签页共用一个渲染进程    
            多标签页面共用一个渲染进程： 同一站点     https协议相同  根域名相同   端口相同    

            多标签页面生产多个渲染进程： 极客帮 打开 infoq   会有不同渲染进程

    
    5、进程和线程
    
        两者区别
            进程：
                占用资源大
                一个进程就是一个程序的运行实例
            线程：
                占用资源更小
                不能单独存在的，由进程来启动和管理
                概念-----启动一个程序的时候，操作系统会为该程序创建一块内存，用来存放代码、运行时数据和一个执行任务的主线程，我们把这样的一个运行环境叫进程

        特点：
            1、进程中的任意一线程执行出错，都会导致整个进程的崩溃

            2、线程之间共享进程中的数据

            3、当一个进程关闭之后，操作系统会回收进程所占用的内存

                （ 
                    当一个进程退出时，操作系统会回收该进程所申请的所有资源；
                    即使其中任意线程因为操作不当导致内存泄漏，当进程退出时，这些内存也会被正确回收。
                    比如之前的 IE 浏览器，支持很多插件，而这些插件很容易导致内存泄漏，这意味着只要浏览器开着，内存占用就有可能会越来越多，但是当关闭浏览器进程时，这些内存就都会被系统回收掉
                ）

            4、进程之间的内容相互隔离。互不干扰。进程通信IPC机制



    6、http请求过程
        1 构建请求
        2 查找缓存(cache-control等)
        3 DNS(缓存)，返回IP
        4 准备IP地址和端口(默认80)
        5 等待TCP队列(chrome最大6个)
        6 建立TCP连接(三次握手--发送3个包--4次挥手)
        7 发起http请求
            发送 请求行 
            发送 请求头
        8 服务器处理请求
        9 服务器响应请求
            服务器回复 响应行 301状态码 (输入baidu.com)      或   methods: GET POST方法
            服务器回复 响应头 location 重定向www.baidu.com  或  (set-cookie 浏览器设置cookie 下次请求携带)   系统信息
            服务器回复 正文
        10 断开tcp 
            判断 请求头 Connection:Keep-Alive  是否重用tcp连接




五、http发展过程

    http1.1
    
        缺陷：
            1 高延迟--页面加载速度降低

                队头阻塞，带宽无法充分被利用：一个请求被卡住，后面的也会卡住

                解决办法：
                    将同一页面的资源分散到不同域名下，提升连接上限
                    Chrome有个机制，对于同一个域名，默认允许同时建立 6 个 TCP持久连接，使用持久连接时，虽然能公用一个TCP管道，但是在一个管道中同一时刻只能处理一个请求，在当前的请求没有结束之前，其他的请求只能处于阻塞状态。
                    另外如果在同一个域名下同时有10个请求发生，那么其中4个请求会进入排队等待状态，直至进行中的请求完成。


            2 无状态特性--巨大http头部

                报文Header一般会携带"User Agent""Cookie""Accept""Server"等许多固定的头字段
                但Body却经常只有几十字节


            3 明文传输--带来的不安全性
                免费WiFi陷阱


            4 不支持服务器推送消息




    SPDY协议与http2
        简介 
            HTTP/2基于SPDY，专注于性能，最大的一个目标是在用户和网站间只用一个连接（connection）


        1 二进制传输
            HTTP/2传输数据量的大幅减少,主要有两个原因:以二进制方式传输和Header 压缩。
            HTTP/2 将请求和响应数据分割为更小的帧，并且它们采用二进制编码。
            多个帧之间可以乱序发送，根据帧首部的流标识可以重新组装。

        2 Header 压缩
            "HPACK”算法
            在客户端和服务器端使用“首部表”来跟踪和存储之前发送的键-值对，对于相同的数据，不再通过每次请求和响应发送

        3 多路复用
            
            多路复用的技术可以只通过一个 TCP 连接就可以传输所有的请求数据

            利用二进制分帧
            数据以消息形式发送，消息又由一个或多个帧组成，多帧之间可以乱序发送，利用帧首的流标识重新组装
            并行交错发送多个请求/响应，且之间不影响

            每个请求都可以带一个31bit的优先值，0表示最高优先级， 数值越大优先级越低。

        4 Server Push 服务推送
            比如，在浏览器刚请求HTML的时候就提前把可能会用到的JS、CSS文件发给客户端，减少等待的延迟，这被称为"服务器推送"

            服务端可以主动推送，客户端也有权利选择是否接收

            如果服务端推送的资源已经被浏览器缓存过，浏览器可以通过发送RST_STREAM帧来拒收
            主动推送也遵守同源策略

        5 提高安全性

            “事实上”的HTTP/2是加密的

            互联网上通常所能见到的HTTP/2都是使用"https”协议名，跑在TLS上面。HTTP/2协议定义了两个字符串标识符：“h2"表示加密的HTTP/2，“h2c”表示明文的HTTP/2。



    http3

        HTTP/2 的缺点            
            主要是底层支撑的 TCP 协议造成的


            1 TCP 以及 TCP+TLS建立连接的延时

                HTTP/2都是使用TCP协议来传输

                https的话，还需要TLS协议，也需要一个握手过程。这样就需要有两个握手延迟过程

                多个请求是跑在一个TCP管道中的。但当出现了丢包时，整个 TCP 都要开始等待重传，那么就会阻塞该TCP连接中的所有请求


            2 TCP的队头阻塞并没有彻底解决


        http3 简介
            基于 UDP 协议的“QUIC”协议(基于UDP)，让HTTP跑在QUIC上而不是TCP上。 真正“完美”地解决了“队头阻塞”问题。




        QUIC新功能
            QUIC基于UDP，不需要“握手”和“挥手”，比TCP来得快，实现了可靠传输，保证数据一定能够抵达目的地，引入http2的“流”和“多路复用”

            特点
                实现了类似TCP的流量控制、传输可靠性的功能(QUIC在UDP基础之上)

                实现了快速握手功能
                    QUIC是基于UDP的，所以QUIC可以实现使用0-RTT或者1-RTT来建立连接

                集成了TLS加密功能
                    因为集成了TLS，减少了握手所花费的RTT个数


                多路复用，彻底解决TCP中队头阻塞的问题
                    和TCP不同，QUIC实现了在同一物理连接上可以有多个独立的逻辑数据流（如下图）。实现了数据流的单独传输，就解决了TCP中队头阻塞的问题。

    总结

        HTTP/1.1有两个主要的缺点：安全不足 和 性能不高。
            明文传输
            大头儿子
            tcp rtt数量 


        HTTP/2完全兼容HTTP/1，是“更安全的HTTP、更快的HTTPS"，头部压缩、多路复用等技术可以充分利用带宽，降低延迟，从而大幅度提高上网体验；
            默认加密
            共用一个tcp，数据流以消息形式发送，消息分为多个帧，帧以二进制传输，根据帧首流标识重组
            字典去重请求头，"HPACK”算法


        QUIC 基于 UDP 实现，是 HTTP/3 中的底层支撑协议，该协议基于 UDP，又取了 TCP 中的精华，实现了即快又可靠的协议








