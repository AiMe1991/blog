title: session 学习
date: 2020-01-09
abbrlink: session/intro
categories:
  - 学习
tags:
  - http
  - session

---

session机是一种服务器端的机制，服务器使用一种类似于散列表的结构（也可能就是使用散列表）来保存信息。 

<!-- more -->

### Session 解释

session 中文经常翻译为会话，其本来的含义是指有始有终的一系列动作/消息，比如打电话时从拿起电话拨号到挂断电话这中间的一系列过程可以称之为一个 session 。有时候我们可以看到这样的话“在一个浏览器会话期间，...”，这里的会话一词用的就是其本义，是指从一个浏览器窗口打开到关闭这个期间。最混乱的是“用户（客户端）在一次会话期间”这样一句话，它可能指用户的一系列动作（一般情况下是同某个具体目的相关的一系列动作，比如从登录到选购商品到结账登出这样一个网上购物的过程），然而有时候也可能仅仅是指一次连接，也有可能是指含义，其中的差别只能靠上下文来推断。

然而当session一词与网络协议相关联时，它又往往隐含了`面向连接`和/或`保持状态`这样两个含义，`面向连接`指的是在通信双方在通信之前要先建立一个通信的渠道。比如：打电话，直到对方接了电话通信才能开始，与此相对的是写信，在你把信发出去的时候你并不能确认对方的地址是否正确，通信渠道不一定能建立，但对发信人来说，通信已经开始了。`保持状态`则是指通信的一方能够把一系列的消息关联起来，使得消息之间可以互相依赖，比如一个服务员能够认出再次光临的老顾客并且记得上次这个顾客还欠店里一块钱。

到了 web 服务器蓬勃发展的时代，session 在 web 开发语境下的语义又有了新的扩展，它的含义是指一类用来在客户端与服务器之间保持状态的解决方案。有时候session 也用来指这种解决方案的存储结构，如“把xxx保存在session里”。由于各种用于web开发的语言在一定程度上都提供了对这种解决方案的支持，所以在某种特定语言的语境下，session也被用来指代该语言的解决方案。


### HTTP协议与状态保持

HTTP协议本身是无状态的，这与HTTP协议本来的目的是相符的，客户端只需要简单的向服务器请求下载某些文件，无论是客户端还是服务器都没有必要纪录彼此过去的行为，每一次请求之间都是独立的，好比一个顾客和一个自动售货机或者一个普通的（非会员制）大卖场之间的关系一样。 

然而聪明（或者贪心？）的人们很快发现，如果能够提供一些按需生成的动态信息会使web变得更加有用，就像给有线电视加上点播功能一样。这种需求一方面迫使HTML逐步添加了表单、脚本、DOM等客户端行为，另一方面在服务器端则出现了CGI规范以响应客户端的动态请求，作为传输载体的HTTP协议也添加了文件上载、cookie这些特性。其中cookie的作用就是为了解决HTTP协议无状态的缺陷所作出的努力。至于后来出现的session机制则是又一种在客户端与服务器之间保持状态的解决方案。 

让我们用几个例子来描述一下cookie和session机制之间的区别与联系。`笔者`曾经常去的一家咖啡店有喝5杯咖啡免费赠一杯咖啡的优惠，然而一次性消费5杯咖啡的机会微乎其微，这时就需要某种方式来纪录某位顾客的消费数量。想象一下其实也无外乎下面的几种方案：

> 1. 该店的店员很厉害，能记住每位顾客的消费数量，只要顾客一走进咖啡店，店员就知道该怎么对待了。`这种做法就是协议本身支持状态`。 
> 2. 发给顾客一张卡片，上面记录着消费的数量，一般还有个有效期限。每次消费时，如果顾客出示这张卡片，则此次消费就会与以前或以后的消费相联系起来。`这种做法就是在客户端保持状态`。 
> 3. 发给顾客一张会员卡，除了卡号之外什么信息也不纪录，每次消费时，如果顾客出示该卡片，则店员在店里的纪录本上找到这个卡号对应的纪录添加一些消费信息。`这种做法就是在服务器端保持状态`。 

由于HTTP协议是无状态的，而出于种种考虑也不希望使之成为有状态的，因此，后面两种方案就成为现实的选择。具体来说cookie机制采用的是在客户端保持状态的方案，而session机制采用的是在服务器端保持状态的方案。同时我们也看到，由于采用服务器端保持状态的方案在客户端也需要保存一个标识，所以session机制可能需要借助于cookie机制来达到保存标识的目的，但实际上它还有其他选择。

### 理解cookie机制

cookie机制的基本原理就如上面的例子一样简单，但是还有几个问题需要解决：“会员卡”如何分发；“会员卡”的内容；以及客户如何使用“会员卡”。 

正统的cookie分发是通过扩展HTTP协议来实现的，服务器通过在HTTP的响应头中加上一行特殊的指示以提示浏览器按照指示生成相应的cookie。然而纯粹的客户端脚本如JavaScript等方式也可以生成cookie。

而cookie的使用是由浏览器按照一定的原则在后台自动发送给服务器的。浏览器检查所有存储的cookie，如果某个cookie所声明的作用范围大于等于将要请求的资源所在的位置，则把该cookie附在请求资源的HTTP请求头上发送给服务器。意思是麦当劳的会员卡只能在麦当劳的店里出示，如果某家分店还发行了自己的会员卡，那么进这家店的时候除了要出示麦当劳的会员卡，还要出示这家店的会员卡。

详细信息查看：[cookie 详解](https://zhuanlan.zhihu.com/p/101315335)


### 理解session机制

session机制是一种服务器端的机制，服务器使用一种类似于散列表的结构（也可能就是使用散列表）来保存信息。 

当程序需要为某个客户端的请求创建一个session的时候，服务器首先检查这个客户端的请求里是否已包含了一个session标识，称为  sessionId  。如果已包含一个 sessionId 则说明以前已经为此客户端创建过session，服务器就按照 sessionId 把这个session检索出来使用（如果检索不到，可能会新建一个）；如果客户端请求不包含 sessionId ，则为此客户端创建一个session并且生成一个与此session相关联的 sessionId ， sessionId 的值应该是一个既不会重复，又不容易被找到规律以仿造的字符串，这个 sessionId 将被在本次响应中返回给客户端保存。 

保存这个 sessionId 的方式可以采用cookie，这样在交互过程中浏览器可以自动的按照规则把这个标识发挥给服务器。一般这个cookie的名字都是类似于sessionId。

由于cookie可以被人为的禁止，必须有其他机制以便在cookie被禁止时仍然能够把 sessionId 传递回服务器。经常被使用的一种技术叫做URL重：

> 1. 把 sessionId 直接附加在URL路径的后面，附加方式也有两种，一种是作为URL路径的附加信息，表现形式为：http://...../xxx;sessionId=ByOK3vjFD75aPnrF7C2HmdnV6QZcEbzWoWiBYEnLerjQ99zWpBng!-145788764
> 2. 作为查询字符串附加在URL后面，表现形式为：http://...../xxx?sessionId=ByOK3vjFD75aPnrF7C2HmdnV6QZcEbzWoWiBYEnLerjQ99zWpBng!-145788764

这两种方式对于用户来说是没有区别的，只是服务器在解析的时候处理的方式不同，采用第一种方式也有利于把sessionId的信息和正常程序参数区分开来。 
为了在整个交互过程中始终保持状态，就必须在每个客户端可能请求的路径后面都包含这个sessionId。


在谈论session机制的时候，常常听到这样一种误解`只要关闭浏览器，session就消失了`。其实可以想象一下会员卡的例子，除非顾客主动对店家提出销卡，否则店家绝对不会轻易删除顾客的资料。对session来说也是一样的，除非程序通知服务器删除一个session，否则服务器会一直保留，程序一般都是在用户做退出登录的时候发个指令去删除session。然而浏览器从来不会主动在关闭之前通知服务器它将要关闭，因此服务器根本不会有机会知道浏览器已经关闭，之所以会有这种错觉，是大部分session机制都使用会话cookie来保存sessionId，而关闭浏览器后这个sessionId就消失了，再次连接服务器时也就无法找到原来的session。如果服务器设置的cookie被保存到硬盘上，或者使用某种手段改写浏览器发出的HTTP请求头，把原来的sessionId发送给服务器，则再次打开浏览器仍然能够找到原来的session。 

恰恰是由于关闭浏览器不会导致session被删除，迫使服务器为session设置了一个失效时间，当距离客户端上一次使用session的时间超过这个失效时间时，服务器就可以认为客户端已经停止了活动，才会把session删除以节省存储空间。 


### session 认证流程

1. 用户第一次请求服务器的时候，服务器根据用户提交的相关信息，创建对应的 session
2. 请求返回时将此 session 的唯一标识信息 sessionId 返回给浏览器
3. 浏览器接收到服务器返回的 sessionId 信息后，会将此信息存入到 cookie 中，同时 cookie 记录此 sessionId 属于哪个域名
4. 当用户第二次访问服务器的时候，请求会自动判断此域名下是否存在 cookie 信息，如果存在自动将 cookie 信息也发送给服务端，服务端会从 cookie 中获取 sessionId，再根据 sessionId 查找对应的 session 信息，如果没有找到说明用户没有登录或者登录失效，如果找到 session 证明用户已经登录可执行后面操作。

![session 认证流程图](/assets/image/session.png)

根据以上流程可知，sessionId 是连接 cookie 和 session 的一道桥梁，大部分系统也是根据此原理来验证用户登录状态。


__参考资料：__

[Session机制详解](https://www.cnblogs.com/wangpei/p/4884840.html)


