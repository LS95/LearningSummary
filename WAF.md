[toc]

# 概念

Web应用防护系统 Web Application Firewall，简称： WAF
Web应用防火墙是通过执行一系列针对HTTP/HTTPS的安全策略来专门为Web应用提供保护的一款产品。

防御目标:
+ SQL注入  DDOS  XML  XSS
+ 网页篡改
+ 网页挂马等安全事件 

WAF一句话描述，就是解析HTTP请求（协议解析模块），规则检测（规则模块），做不同的防御动作（动作模块），并将防御过程（日志模块）记录下来。



# 原理 

WAF工作在应用层，因此对Web应用防护具有先天的技术优势。基于对Web应用业务和逻辑的深刻理解，WAF对来自Web应用程序客户端的各类请求进行内容检测和验证，确保其安全性与合法性，对非法的请求予以实时阻断，从而对各类网站站点进行有效防护。

WAF利用服务器API去获取应用层数据，然后匹配自己的规则库，利用正则表达式来分析判断数据的合法性，生成白名单，黑名单，最后，进行访问控制。


## 与传统防火墙的区别 

WAF是通过检测应用层数据来进行访问控制或对应用进行控制 

传统防火墙对三四层数据进行过滤 基于IP报文进行检测。只是对端口做限制， 从而进行访问控制,不对应用层数据进行分析。

## 工作过程 


解析HTTP请求==》匹配规则==》防御动作==》记录日志 
具体实现如下：

1.解析http请求：协议解析模块

2.匹配规则：规则检测模块，匹配规则库

3.防御动作：return 403 或者跳转到自定义界面

4.日志记录：记录到elk中

```
ELK 是 elastic 公司旗下三款产品 ElasticSearch 、Logstash 、Kibana 的首字母组合。

ElasticSearch 是一个基于 Lucene 构建的开源，分布式，RESTful 搜索引擎。

Logstash 传输和处理你的日志、事务或其他数据。

Kibana 将 Elasticsearch 的数据分析并渲染为可视化的报表。
```


# 分类 



1. 嵌入式 -- web数据是已经被解析加工好的

+ web容器模块类型WAF
+ 代码层WAF 

2. 非嵌入式 -- web流量的解析完全靠自身

+ 硬件WAF
+ 云WAF
+ 虚拟机WAF

---  

另外的分类方式 

1. 硬件WAF  -- 透明桥接  旁路模式  反向代理  
+  绿盟Web应用防火墙
+  梭子鱼
+  天清等

2. 软件WAF -- 安装在web应用服务器上 
+ 安全狗
+ ModSecurity  

3. 代码级WAF
+  应用程序安全架构的范畴  遵循安全编码最佳实践的产物,php web应用里面 php.ini中修改 

4. 云WAF  

+ aliyun 云盾  访问控制 

#  特性 

1. 异常检测协议，对HTTP的请求进行异常检测 拒绝不符合HTTP标准的请求。
2. 增强的输入验证 ， 应用层输入验证， 客户端浏览器白名单过滤策略，参数化语句保证SQL执行，数据查询转义技术， 向URL发送之前对数据进行编码。

3. 及时补丁 对于具体漏洞场景编写紧急防御规则进行hot patch.
4. 基于规则的保护 和 基于异常的保护 
5. 状态管理 

waf可以判断用户是否是第一次访问并将请求重定向到默认登录页面并记录事件。  

#  攻防

从架构、资源、协议和规则4个层次研究绕过WAF的技术，助于全方位提升WAF防御能力。

+ 从架构层Bypass WAF 。
+ 从资源限制角度bypass WAF。
+ 从协议层面bypass WAF。
+ 从规则缺陷bypass WAF。


##  分块传输绕过WAF 


waf无法识别 但web容器能够正常解析内容

### 思路

在头部加入 Transfer-Encoding: chunked 之后，就代表这个报文采用了分块编码。这时，*post*请求报文中的数据部分需要改为用一系列分块来传输。每个分块包含十六进制的长度值和数据，长度值独占一行，长度不包括它结尾的，也不包括分块数据结尾的，且最后需要用0独占一行表示结束。

分块传输可以在长度标识处加上; 作为注释 

eg:
```
9;kkkkk
1234567=1
4;ooo=222
2345
0
(两个换行)


```

分块编码传输需要将关键字and,or,select ,union等关键字拆开编码，不然仍然会被waf拦截。编码过程中长度需包括空格的长度。最后用0表示编码结束，并在0后空两行表示数据包结束，不然点击提交按钮后会看到一直处于waiting状态。

此方法只适合post数据提交绕过  




# 参考链接

+ [WAF的相关知识记录](https://ixyzero.com/blog/archives/3079.html)
+ [WAF渗透攻防实践](http://www.arkteam.net/?p=3025)
+ [WAF攻防研究之四个层次Bypass WAF](https://xz.aliyun.com/t/15)
+ [利用分块传输吊打所有WAF](https://www.anquanke.com/post/id/169738)
+ [Burp suite 分块传输辅助插件](https://github.com/c0ny1/chunked-coding-converter)
+ [编写Burp分块传输插件绕WAF](https://mp.weixin.qq.com/s?__biz=Mzg3NjA4MTQ1NQ==&mid=2247483787&idx=1&sn=54c33727696f8ee6d67f997acc11ab89&chksm=cf36f9cbf84170dd7da9b48b3365fb05d7ccec6bdeff480d0c38962f712e400a40b2b38dc467&token=360242838&lang=zh_CN#rd)