### cdn及浏览器缓存

一、为什么使用CDN或浏览器缓存、解决什么问题

    1 网络延迟
        浏览器获取资源是从网络环境中，所以会带来延迟。为了提高加载速度。


    2 减少数据传输体积
        不使用缓存，每次都会从服务器重新拉取js css img 等资源。造成重复下载大量资源


    3 说明：
        用户第一次请求网站，浏览器会从服务器一次性拉取所有资源。

        浏览器拿到资源后，会根据请求头缓存配置，判断是否将资源缓存到本地

        当第二次请求的时候，浏览器判断是否优先读取缓存


二、CDN或缓存介绍

    1 CDN介绍

        根据不同网络、不同地区用户，创建接近用户网络的边缘服务器，然后将文件缓存在这些边缘服务器（节点）上

        电信cdn 联通cdn 移动cdn 山东cdn 河北cdn


    2 策略
        根据不同cdn厂商配置不同


    3 缺点
        源服务器数据更新，cdn服务器数据未过期
        用户访问到的依旧是过期的缓存资源，这会导致用户最终访问出现偏差

        需要手动刷新缓存

        自动刷新：

            等文件在 CDN 节点的缓存过期之后，
            节点回源拉取源服务器上最新的文件。这个过程由 CDN 自动完成，无需手动操作





三、缓存--浏览器缓存

    策略
    
        1 强缓存

            Expires 

                指定资源到期时间(服务器端的具体时间)， 修改本地时间，造成缓存失效

            Cache-control  

                public：所有内容都将被缓存(客户端、代理服务器-cdn)

                private：内容只缓存到客户端，不缓存到代理服务器

                no-cache：配合Etag Last-modified 使用。(此设置只是cache-control 这一项不起作用，但是协商缓存还是会起作用)

                no-store：所有内容不被缓存

                max-age=xx：缓存内容奖在xxx秒后失效

                例：
                    Cache-Control: max-age=691200，可以在客户端存储 8 天


            以上两个对比

                Expires     http 1.0 产物

                Cache-control   http 1.1 产物  权重高于Expires


        2 协商缓存

            Last-Modified / If-Modified-Since

                秒为单位记时， 如果1秒内修改多次，不会命中缓存

                第一次请求资源，相应头会带有Last-Modified数据，
                然后第二次请求的时候会把 Last-Modified 数据转为 If-Modified-Since 和服务器资源修改时间判断


            ETag / If-None-Match

                原理类似上面的

                第一次请求资源，相应头会带有 ETag 数据，
                然后第二次请求的时候会把 ETag 数据转为 If-None-Match 和服务器资源修改时间判断


            以上两个对比

                ETag 权重高于 Last-Modified

                ETag 精确度高于 Last-Modified

                ETag 性能差于 Last-Modified (ETag 要计算生成)


    缓存机制

        先进行强制缓存，如果生效，直接读取缓存资源，失效进行协商缓存处理。

        协商缓存失效，200ok ，重新返回资源及缓存标识

        协商缓存生效，304，继续使用缓存





    实际应用场景

        频繁改动：cache-control：no-cache。(配合etag + last-modified)

        不常改动：cache-control：max-age=34536000 一年。(类似jq，webpack打包后会重新生成hash，旧的缓存就不用了，而使用新的缓存)


    缓存位置

        service worker

            自由控制缓存哪些文件，如何匹配缓存、如何读取缓存

            并且缓存是持续性的


        memory cache
            
            读取速度快。持续性短

            窗口未关闭时


        disk cache

            读取速度慢

            比memory cache 容量大 存储时间长


        push cache

            只在会话 session中存在

            会话结束后被释放，缓存时间也短


    用户行为的影响

        地址栏输入url： 先disk cache 是否匹配。 没有匹配发起请求

        普通刷新(f5)： 优先使用memory-cache，其次才disk-cache

        强制刷新(ctrl+f5)：浏览器不使用任何缓存






三、知识点

    1、cdn过程

        访问 a.cdn.com
        检查资源缓存配置，200 cache(内存、硬盘)
        a.cdn.com，判断资源是否过期，
        过期：重新拉取源服务器数据，至cdn服务器  200ok
        未过期：返回304 读取本地缓存
        没资源：去源服务器去拉取，这个过程叫 回源


<!-- 
    https://www.jianshu.com/p/baf12d367fe7 
    
    配图

    https://camo.githubusercontent.com/4e1a2fff1565062e3c71363f91dd1fba6a5e0d8b/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031392f312f352f313638316332316530343535326637373f773d3232313326683d39343826663d706e6726733d333131313733
-->












