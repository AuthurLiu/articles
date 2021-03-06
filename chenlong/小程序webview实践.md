# 小程序webview实践 -- 张所勇

![image](https://img1.zhuanstatic.com/common/img/01.png)

大家好，我是转转开放业务部前端负责人张所勇，今天主要来跟大家分享小程序webview方面的问题，但我并不会讲小程序的webview原理，而我主要想讲的是小程序内如何嵌入H5。

那么好多同学会想了，不就是用web-view组件就可以嵌入了吗，是的，如果咱们的小程序和H5的业务比较简单，那直接用webview接入就好了，但我们公司的h5除小程序之外，还运行在转转app、58app、赶集app等多个端，如何能实现一套代码在多端运行，这是我今天主要想分享的，因此今天分享更适合h5页面比较复杂，存在多端运行情况的开发者，期待能给大家提供一些多端兼容的思路。

![image](https://img1.zhuanstatic.com/common/img/02.png)

下面我先跟大家介绍下今天演讲主要的提纲。
1.  小程序技术演进
1.  webview VS 小程序
1.  h5多端兼容方案
1.  小程序sdk设计
1.  webview常见问题

## 1 转转小程序演进过程

![image](https://img1.zhuanstatic.com/common/img/03.png)

相信在座的很多同学的产品跟转转小程序经历了类似的发展过程，我们转转小程序是从去年五月份开始开发的，那时候也是小程序刚出来不久，我们就快速用原生语法搭建了个demo，功能很简单，就是首页列表页详情页。

然后我们从7月份开始进入了第二个阶段，这时候各种中大型公司已经意识到了，借助微信的庞大用户群，小程序是一个很好的获客渠道，因此我们也从demo阶段正式的开始了小程序开发。

![image](https://img1.zhuanstatic.com/common/img/9c59a59c-bf61-40c8-ad88-654f82123564.jpg)

那时候我们整个团队从北京跑到广州的微信园区里面去封闭开发，我们一方面在做小程序版本的转转，实现了交易核心流程，苦苦的做了两三个月，DAU始终也涨不上去，另一方面我们也在做很多营销活动的尝试，我们做了一款简单的测试类的小游戏，居然几天就刷屏了，上百万的pv，一方面我们很欣喜，另一方面也很尴尬，因为大家都是来玩的，不是来交易的，所以我们就开始了第三阶段。

这个阶段我们进行了大量的开发工作，让我们的小程序功能和体验接近了转转APP，那到了今年6月份，我们的小程序进入了微信钱包里面，我们的DAU峰值也达到了千万级别，这时候可以说已经成为了一个风口上的新平台，这个时候问题来了，公司的各条线业务都开始想接入到小程序里面。

![image](https://img1.zhuanstatic.com/common/img/04.png)

于是乎就有了上面这段对话。
所以，为了能够更好接入各业务线存量h5页面和新的活动页，我们开始着手进行多端适配的工作。

那我们会考虑三种开发方案（我这里只说缺点）

![image](https://img1.zhuanstatic.com/common/img/05.png)

在webview这个组件出来以前，我们是采用第一种方案，用纯小程序开发所有业务页面，那么适合的框架有现在主流的三种，wepy，mpvue、taro，缺点是不够灵活，开发成本巨大，真的很累，尤其是业务方来找我们想介入小程序，但他们的开发者又不会小程序，当时又不能嵌入H5，所以业务方都希望我们来开发，所有业务都来找，你们可以想想成本又多高，这个方案肯定是不可行，第二种方案，就是一套代码编译出多套页面，在不同端运行，mpvue和taro都可以，我们公司有业务团队在使用mpvue，编译出来的结果不是特别理性，一是性能上面没有达到理想的状态，二是api在多端兼容上面二次改造的成本很高，各个端api差异很大，如果是一个简单的活动页还好，但如果是一个跟端有很大功能交互的页面，那这种方式其实很难实现。

那我们采用的是第三种方案，目前我们的小程序是作为一个端存在，像app一样，我们只做首页、列表、详情、购买等等核心页面都是用小程序开发，每个业务的页面、活动运营页面都是H5，并且用webview嵌入，这样各个业务接入的成本非常低，但这也有缺点，1是小程序与h5交互和通信比较麻烦，二是我们的app提供了很大功能支持，这些功能在小程序里面都需要对应的实现

## 2 webview VS 小程序

![image](https://img1.zhuanstatic.com/common/img/07.png)

这张图是我个人的理解。（并不代表微信官方的看法）
把webview和小程序在多个方面做了比对。

## 3 h5多端兼容方案

![image](https://img1.zhuanstatic.com/common/img/09.png)

未来除了小程序之外，可能会多的端存在，比如智能小程序等等，那我们期望的结果是什么呢，就是一套H5能运行于各个环境内。

![image](https://img1.zhuanstatic.com/common/img/10.png)

这可能是最差的一个case，h5判断所在的环境，去调用不同api方法，这个case的问题是，业务逻辑特别复杂，功能耦合非常严重，也基本上没有什么复用性。

![image](https://img1.zhuanstatic.com/common/img/11.png)

那我们转转采取的是什么方案呢？

分三块，小程序端，用WePY框架，H5这块主要是vue和react，中间通过一个adapter来连接。我们转转的端特别多，除了小程序还包括纯转转app端，58端，赶集端，纯微信端，qq端，浏览器端，所以H5页面需要的各种功能，在每个端都需要对应的功能实现，比如登录、发布、支付、个人中心等等很多功能，这些功能都需要通过adapter这个中间件进行调用，接下来详细来讲。

![image](https://img1.zhuanstatic.com/common/img/12.png)

我这里就不贴代码了，我只讲下adapter的原理，首先adapter需要初始化，做两件事情，一是产出一个供h5调用的native对象，二是需要检测当前所处的环境，然后根据环境去异步加载sdk文件，这里的关键点是我们要做个api调用的队列，因为sdk加载时异步的过程，如果期间页面内发生了api调用，那肯定得不到正确的响应，因此你要做个调用队列，当sdk初始化完毕之后再处理这些调用，其实adapter原理很简单，如果你想实现多端适配，那么只需要根据所在的环境去加载不同的sdk就可以了。

![image](https://img1.zhuanstatic.com/common/img/13.png)

做好adapter之后，你需要让每个h5的项目都引入adapter文件，并且在调用api的时候，都统一在native对象下面调用。


## 4 小程序sdk设计

![image](https://img1.zhuanstatic.com/common/img/15.png)

我们总结小程序提供给H5的功能大体分为这四种，第一是基于小程序的五种跳转能力，比如关闭当前页面。

![image](https://img1.zhuanstatic.com/common/img/16.png)

那我们看到小程序提供了这五种跳转方式。

![image](https://img1.zhuanstatic.com/common/img/17.png)

第二是直接使用微信sdk提供的能力，比如扫码，这个比较简单。第三是h5打开小程序的某些页面，这个是最常用的，比如进入下单页等。

![image](https://img1.zhuanstatic.com/common/img/18.png)

对应每个api，小程序这边都需要实现对应的页面功能，大家看这几个图，skipToChat就是进到我们的IM页面，下面依次是进入个人主页，订单详情页，下单页面，其实我们的小程序开发的主要工作也是去做这些基础功能页面，然后提供给H5，各个业务基本都是H5实现，接入到小程序里面来，我们只做一个平台。

![image](https://img1.zhuanstatic.com/common/img/19.png)

这是进入个人主页方法的实现，其实就是进入了小程序profile这个页面。

![image](https://img1.zhuanstatic.com/common/img/20.png)

第四就是h5与小程序直接的通信能力，这个比较集中体现在设置分享信息和登录这块。

### 4.1 设置分享

![image](https://img1.zhuanstatic.com/common/img/22.png)

上面（adapter）做好了以后，在h5这块调用就一句话就可以了。

![image](https://img1.zhuanstatic.com/common/img/23.png)

小程序和h5 之间的通信基本上常用两种方式，一个是postMessage，这个方法大家都知道，只有在三种情况才可以触发，后退、销毁和分享，但也有个问题，这个方法是基础库1.7.1才开始支持的，1.7.1以下就只能通过第二种方法来传递数据，也就是设置和检测webview组件的url变化，类似pc时代的iframe的通信方式。

![image](https://img1.zhuanstatic.com/common/img/24.png)

sdk这块怎么做呢，定义一个share方法，首先需要检测下基础库版本，看是否支持postMessage，如果支持直接调用，如果不支持，把分享参数拼接到url当中，然后进行一次重载，所以说通过url传递数据有个缺点，就是页面可能需要刷新一次才能设置成功。

![image](https://img1.zhuanstatic.com/common/img/25.png)

我们看下分享信息设置这块，小程序这端，首先通过bindmessage事件接收h5传回来的数据，然后在用户分享的时候onShareAppMessage判断有没有回传的数据，如果没有就到webviewurl当中取，否则就是用默认分享数据。

![image](https://img1.zhuanstatic.com/common/img/26.png)

h5调分享这块，我们也做了一些优化，传统方式是要四步才能掉起分享面板，点页面里的某个按钮，然后给用户个提示层，用户再去点右上角的点点点，再点转发，才能拉起分享面板。

![image](https://img1.zhuanstatic.com/common/img/27.png)

我们后来改成了这样，点分享按钮，把分享信息带到一个专门的小程序页面，这个页面里面放一个button，type=share，点一下就拉起来面板了，虽然是一个小小的改动，但能大幅提高分享成功率的，因为很多用户对右上角的点点点不太敏感。

### 4.2 登录

接下来我们看看登录功能

![image](https://img1.zhuanstatic.com/common/img/29.png)

分两种情况，接入的H5可能一开始就需要登录态，也可能开始不需要登录态中途需要登录，这两种情况我们约定了h5通过自己的url上一个参数进行控制。

1. 一开始就需要登录态的情况，那么在加载webview之前，首先进行授权登录，然后把登录信息拼接到url里面，再去来加载webview，在h5里面通过adapter来把登录信息提取出来并且存到cookie里，这样h5一进来就是有登陆态的。

2. 一开始不需要登录态的情况，一进入小程序就直接通过webview加载h5，h5调用login方法的时候，把needLogin这个参数拼接到url中，然后利用api进行重载，就走了第一种情况进行授权登录了。

## 5 webview常见问题

### 5.1 区分环境

第一是你如何区分h5所在的环境是小程序里面的webview还是纯微信浏览器，为什么要区分呢，因为你的H5在不同环境需要不同的api，比如我们的业务，下单的时候，如果是小程序，那么我们需要进入小程序的下单页，如果是纯微信，我们之间进纯微信的下单页，这两种情况的api实现是不一样的，那么为什么难区分，大家可能知道，小程序的组件分为内置组件和原生组件，内置组件就是小程序定义的view、scroll-view这些基本的标签，原生组件就是像map啊这种，这其实是调用了微信的原生能力，webview也是一种类似原生的组件，为什么说是类似原生的组件，微信并没有为小程序专门做一套webview机制，而是直接用微信本身的浏览器，所以小程序webview和微信浏览器的内核都是一样的，包括UA头都是一模一样，cookie、storage本地存储数据都是互通的，都是一套，因此我们很难区分具体是在哪个环境。

![image](https://img1.zhuanstatic.com/common/img/32.png)

还好微信提供了一个环境变量，但这个变量不是很准确，加载h5以后第一个页面可以及时拿到，但后续的页面都需要在微信的sdk加载完成以后才能拿到，因此建议大家在wx.ready或者是weixinjsbridgeready事件里面去判断，区别就在于前者需要加载jweixin.js才有，但这里有坑，坑在于h5的开发者可能并不知道你这个检测过程需要时间，是一个异步的过程，他们可能页面一加载就需要调用一些api，这时候就可能会出错，因此你一定要提供一个api调用的队列和等待机制。

### 5.2 支付

第二个常见问题是支付，因为小程序webview里面不支持直接掉起微信支付，所以基本上需要支付的时候，都需要来到小程序里面，支付完再回去。

![image](https://img1.zhuanstatic.com/common/img/34.png)

上面做好了以后，在h5这块调用就一句话就可以了。

![image](https://img1.zhuanstatic.com/common/img/35.png)

我们转转的业务分两种支付情况，一是有的业务h5有自己完善的交易体系，下单动作在h5里面就可以完成，他们只需要小程序付款，因此我们有一个精简的支付页，进来直接就拉起微信支付，还有一种情况是业务需要小程序提供完整的下单支付流程，那么久可以之间进入我们小程序的收银台来，图上就是sdk里面的基本逻辑，我们通过payOnly这个参数来决定进到哪个页面。

![image](https://img1.zhuanstatic.com/common/img/36.png)

我们再看下小程序里面精简支付怎么实现的，就是onload之后之间调用api拉起微信支付，支付成功以后根据h5传回来的参数，如果是个小程序页面，那之间跳转过去，否则就刷新上一个webview页面，然后返回回去。

### 5.3 formId收集

第三个问题是formId，webview里面没有办法收集formId，这有什么影响呢，没有formId就没法发服务通知，没有服务通知，业务就没办法对新用户进行召回，这对业务来讲是一个很大的损失，目前其实我们也没有很好的方案收集。

![image](https://img1.zhuanstatic.com/common/img/38.png)

我们目前主要通过两种方式收集，访问量比较大的这种webview落地页，我们会做一版小程序的页面或者做一个小程序的中转页，只要用户有任何触摸页面的操作，都可以收集到formid，另外一种就是h5进入小程序页面时候收集，比如支付，IM这些页面，但并不是每个用户都会进到这些页面的，用户可能一进来看不感兴趣，就直接退出了，因此这种方式存在很大的流失。

### 5.4 左上角返回

那怎么解决这种流失呢，我们加了一个左上角返回的功能，。

![image](https://img1.zhuanstatic.com/common/img/40.png)

首先进入的是一个空白的中转页，然后进入h5页面，这样左上角就会出现返回按钮了，当用户按左上角的返回按钮时候，页面会被重载到小程序首页去，这个看似简单又微小的动作，对业务其实有很大的影响，我们看两个数字，经过我们的数据统计发现，左上角返回按钮点击率高达70%以上，因为这种落地页一般是被用户分享出来的，以前纯h5的时候只能通过左上角返回，所以在小程序里用户也习惯如此，第二个数字，重载到首页以后，后续页面访问率有10%以上，这两个数字对业务提升其实蛮大的。

![image](https://img1.zhuanstatic.com/common/img/41.png)

其实现原理很简单，都是通过第二次触发onShow时进行处理。

以上就是我今天全部演讲的内容，谢谢大家！

![image](https://img1.zhuanstatic.com/common/img/43.png)

这是我们“大转转FE”的公众号。里面发表了很多FE和小程序方向的原创文章。感兴趣的同学可以关注一下，如果有问题可以在文章底部留言，我们共同探讨。

同时也感谢掘金举办了这次大会，让我有机会同各位同仁进行交流。在未来的前端道路上，与大家共勉！
