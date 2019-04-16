---
title: 支付宝集成-当面付
date: 2019-04-01 15:52:24
tags: Alipay
categories: 第三方接口
---
前段时间做项目，用到AliPay的接口，手册看得很懵，东一个西一个不知道从哪开头。花了些天看了下当面付的Demo文档，总算弄明白了些AliPay接口开发的流程，这里也以当面付Demo为案例，介绍Pay的开发流程。
<!--more-->
事先申明，该文只是对AliP当面付Demo进行解析，非原创。
当面付的[官方文档](https://docs.open.alipay.com/194/105170/)，同时也是Demo下载地址
事实上开发支付，有alipay-sdk-java.jar这个集成SDK就可以了，而Demo中的TradePaySDK是对alipay-sdk-java.jar进一步的封装，但同时也是我们可以参考的工程模板。开始入口看TradePayDemo中的Main.java也可以参考WebRoot下的JSP。
## Demo结构
首先，献上个人理解的工程结构图：
![](https://s2.ax1x.com/2019/04/13/ALjV6f.png)
ps.自己画的思维导图，不合理的地方还请谅解
### 状态类别
hb包中都是用于封装交易进行时的状态常量，通常为枚举，少部分是接受支付宝端返回的参数，需要解析json数据再封装，其中的Adapter类就是为其服务的

### Utils
工具包，这个不用多说吧，有过项目经验的都知道

### request
Builder包的request请求类，负责将请求参数封装到其内部的BizContent中，同时提供输出json字符串的方法，它只是中间类，最终的请求还是要用alipay-sdk-java.jar中封装的request。

### service
service包，其负责服务功能，如支付、查询、退款等。

### result
result包，对支付宝响应结果进一步封装

现在让我们走一遍支付流程
## 支付流程
先看看官方的流程图
![](https://s2.ax1x.com/2019/04/13/ALvRMV.jpg)
讲道理，看能看懂，但一开始却不知道怎么和Demo联系起来，后来自己总结画了一张流程图
![](https://s2.ax1x.com/2019/04/13/ALxpid.png)
ps.艺术水准就这样，见谅啊~
### 参数入场
来到TradePayDemo中的Main
![](https://s2.ax1x.com/2019/04/14/AO5qX9.png)
其static块中，初始化公共参数和实例化service

![](https://s2.ax1x.com/2019/04/14/AOIMcQ.png)
main方法中提供的方法就是给我们测试的功能的，这里以支付功能为例，看看test_trade_pay(tradeService);
![](https://s2.ax1x.com/2019/04/14/AOI4ud.png)
这里需要填写一大堆的参数，即请求参数，完整的参数集[看官方调试接口](https://openhome.alipay.com/platform/demoManage.htm#/alipay.trade.pay)。这里有些是必填，如付款码，有些则可以忽略。随后将参数传入TradePaySDK的request类封装（其实也可以用alipay-sdk-java.jar提供的ModelObject类封装，更简便，有兴趣的可以到最后看我的RESTApi项目），再交由service类处理得到result。到这，只要你zfbinfo.properties中公共参数和请求参数必填项填好，你就可以运行main方法了，不出意外你的沙盒钱包被扣0.01元哦~。
### 业务服务
现在让我们来到TradePaySDK中的service包，接口就不说了，就是定义好提供给main方法，关键是impl中的那些抽象类。

AbsAlipayService定义了支付、查询等都要用的方法，验证bizContent对象不为空，即保证参数json不为空。getResponse()则是整个工程中真正和支付宝平台交互的地方，关键语句：

	AlipayResponse response = client.execute(request);
相信你在查阅支付宝API在线调试时经常看到，事实上如果你只是想简单测试一下API接口，不想写一个这样的工程，一下三句就OK了
```
AlipayClient alipayClient = new DefaultAlipayClient("https://openapi.alipay.com/gateway.do","app_id","your private_key","json","GBK","alipay_public_key","RSA2");

AlipayTradePayRequest request = new AlipayTradePayRequest();
request.setBizContent("{" +
"\"out_trade_no\":\"20150320010101001\"," +
"\"scene\":\"bar_code\"," +
...
"  }");

AlipayTradePayResponse response = alipayClient.execute(request);
```
一个Client和Request发起访问，响应结果。即客户端 --> 支付宝平台。都封装在alipay-sdk-java.jar

AbsAlipayTradeService已经实现了接口和大部分业务流程需要的方法，相比之下感觉AlipayTradeServiceImpl有种多余的赶脚~。之后就如上面的流程图那样，tradePay()中发起支付，同步返回当然最好，但也要做好异常处理。response为null或状态吗异常则进入轮询loopQueryResult()，不断查询订单tradeQuery()，次数在zfbinfo.properties中定义。不管最后有没有查询到结果，都将返回result给main显示结果。

至此完成一次支付操作，至于其他的功能，相信大家也知道怎么开始看了吧
以上纯属个人见解，如有错误欢迎指出


