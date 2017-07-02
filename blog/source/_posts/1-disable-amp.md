---
title: 我决定在我的网站上禁用AMP[译文]
---

原文：[https://www.alexkras.com/i-decided-to-disable-amp-on-my-site](https://www.alexkras.com/i-decided-to-disable-amp-on-my-site)
（有删减）

![图片](https://www.alexkras.com/wp-content/uploads/amp-link.png)

我和Google的Accelerated Mobile Pages (AMP) 项目有一段很长的渊源，但昨天我忍无可忍了。

我在Twitter（iPhone 6的Safari）上注意到有人用一个AMP链接到我的网站。我回复了一个真实链接，但当我点击这个链接的时候，又被重定向到了AMP的版本。

<!--more-->

我复制了Twitter生成的这个链接，它看起来是这样子的：`https://t.co/6drRK5Cugz?amp=1`

注意链接中的`amp=1`。当我点击链接，它返回了下面的HTML页面：

```html
<head>
    <noscript>
        <meta http-equiv="refresh" 
            content="0;URL=https://www.alexkras.com/simple-guide-to-finding-a-javascript-memory-leak-in-node-js/amp/" 
        />
    </noscript>
    <title>https://www.alexkras.com/simple-guide-to-finding-a-javascript-memory-leak-in-node-js/amp/</title>
</head>
<script>
window.opener = null;
location
    .replace(`https:\/\/www.alexkras.com\/
        simple-guide-to-finding-a-javascript-memory-leak-in-node-js\
        /amp\/`)
</script>
```

这个页面的唯一目的就是重定向到链接的AMP版本。

## 作为一个发布者AMP对我造成的问题

当AMP刚问世的时候，我是很看好的。AMP的目的是为了让访问网络更快，这个我很赞赏。

作为一个发布者，我很不喜欢的是事实上Google在缓存AMP内容，并且从他们自己的缓存和自己的域名下面提供这些内容。最终的链接会变成这样：

`https://www.google.com/amp/www.bbc.co.uk/news/amp/39130072`

![](https://www.alexkras.com/wp-content/uploads/France-Election.png)

换句话说，内容不是从BBC.co.uk上获取的，而是从google.com上获取的。

这个结果造成了几个问题：

1. 它“欺骗”了Google的用户。如果用户点击了上面截图的“x”他们会被带回Google搜索结果。而一个普通的跳转会让用户更可能的留在BBC的网站。相反，AMP的返回按钮提供的这个功能让用户更可能回到Google。这对用户是无关紧要的，但对于作者却是个糟糕的结果。
2. 这个系统会被滥用。例如从AMP上获取的假新闻可能会被一个比较单纯的用户信任，因为新闻是从google.com上获取的——一个很权威的域名。

#### 讽刺的是，这并不是Twitter在做的。

我想Twitter的做法基于了一个前提：通过AMP格式提供的内容对于用户来说更好。因此，他们仅仅是为了取悦用户并且提供最可能好的内容格式。

我不能责怪Twitter把我的推送内容变成AMP版本，要怪就怪我自己，因为是否使用AMP是完全可选的，但是我选择了再我的网站上激活。

## 作为一个用户AMP对我造成的问题

AMP由三个组件组成：HTML，Javascript库和缓存。我想说一下上一节提到的缓存问题。缓存的好处，google的AMP工程师跟我们解释了，就是它允许Google（或者其他平台）为用户预加载内容。当用户点击一个AMP链接，Google几乎可以立刻渲染结果，因为它已经在后台获取了。

但又一次讽刺的是，Twitter并没有用缓存这一层。Twitter在打赌，移动用户在AMP可用的情况下宁愿阅读AMP渲染的内容，即使它没有予加载到后台。

而我的问题是我根本就不喜欢阅读AMP内容。AMP对我来说有很多小问题真的恶心到我了。

例如：

#### 坑爹的滚动

在iPhone上，AMP似乎覆盖了浏览器的默认滚动，结果是AMP页面的滚动体验不好。

#### 很难分享链接

用AMP很难分享链接到原内容。
从浏览器顶部的地址栏上不能拷贝到链接，用户必须点击[一个特殊的按钮](https://www.alexkras.com/amp-toolbar-now-has-a-button-to-view-and-copy-original-url/)才能够看到源地址，然后用户点击源链接，等待调转，才可以从地址栏上复制到链接。

PS：那个源链接按钮其实跟原先的AMP版本比已经是一个很大的进步了，原先用户必须手动删除`https://www.google.com/amp/`的一部分才能够获取到真实链接。

我不喜欢用AMP内容因为我想确保链接访问的是真正的来源内容。我相信大部分用户不会去复制源链接，他们只会复AMP链接，像`https://www.google.com/amp/www.bbc.co.uk/news/amp/39130072`，然后分享出去。例如我的老婆，她总是发给我AMP链接。我很惊讶很多发布者惊人没有跟我一样有这些困扰。

#### AMP内容一般都会被阉割

回想一下我们以前用的WAP页面——只在移动设备上使用的特定web页面。而发布者选择AMP，某种程度上就像是回到了用WAP页面的时代。发布者不用响应式设计（确保一个网站的版本可以在全部设备上正常展现），而被迫每个页面要维护两个版本，一个常规的版本供大的设备还有不用Google的手机，还有一个AMP版本。

AMP的好处是通过严格显示内容格式让页面加载变快。这个方法的问题是AMP就变成了源内容的子集。例如，用户评论常常就被移除了。

我也发现了[图片在AMP的加载很怪异](https://github.com/ampproject/amphtml/issues/9397)。AMP试图让图片对用户可见的时候才加载，而没有加载前就是一块方形的空白。但我根据我的经验我经常发现文章中的图片会只是一个方形的空白。

一些不同的网站还有很多其他小问题。例如，Reddit的评论（网站的很重要一部分）被AMP缓存了。结果，新的评论会在Reddit的桌面版本显示，而不会在Reddit的AMP上显示，直到页面的AMP缓存更新了。

#### AMP对于用户来说不是可选的

对于发布者来说是否添加AMP支持是取决于他们自己的。但是对于用户，他们没有办法取消AMP。

如果Google可以提供一个用户层面上的设置来取消AMP那就好了。不幸的是，即使他们要加上这个设置，那也不会有太大效果，因为Twitter和Facebook可能只是从AMP上提供内容。

## 为什么刚开始我在我的网站设置了AMP

我一开始在我的网站上设置AMP只有一个原因——提高google搜索排名。

AMP成型前不久，google宣布他们将开始处理在移动设备渲染比较慢的网站。我的网站有一个响应式的主题，但我不是很确定这对移动设备是不是够友好。因此，当我知道WordPress有一个AMP插件后，我做了设置。虽然Google官方说明了额AMP并不会影响网站的搜索排名，但我觉得这也不会有什么问题。

另一个AMP的搜索排名优点，就是直邮AMP网站会被显示在Google的carousel中。我倒是对这个无所谓，但是这对于那些大的内容网站来说是很重要的。

[](https://www.alexkras.com/wp-content/uploads/carousel.png)

## 它正在扩散

当我开始发现Google缓存我的网站，我开始考虑禁用AMP，但我后面决定还是留着，这是我的两个理由：

1. 我想保持我的搜索排名
2. 我想把AMP当作一个选择，那些喜欢这个格式的读者可以选择它。

知道这次的Twitter体验，我才意识到设置了AMP，允许了其他网站可以选择如何链接到我的内容。

两周前我写了这些：

> 我对AMP本身没有问题。我不在意Facebook，Instant，Articles或者Pinterest使用AMP。

但兄弟我错了。直到我看到我在twitter上的链接被强行用AMP渲染，我想我还是在意的。

## 用户不需要AMP

几周前有人在Twitter上说我不喜欢AMP因为我有一个“特权”——我的互联网环境比较快。虽然这是真的，但我不认为在一个网速慢的环境我会用AMP。我会让浏览器金庸javascript（或者图片也仅用掉，如果网速真的很慢）。这或许不能对所有网站起作用，但我的网站是在服务器端渲染和缓存，用户只需要下载一点html就能看到页面。为什么要强制他们去下载AMP这个javascript库呢？

当然，Google推出AMP的原因是他们想让用户看到广告，这些如果禁用掉javascript就看不到了。我也想让用户看到广告，我服务器的花的钱就是通过这个赚到的。话虽如此，如果用户访问我的网站速度很慢然后选择退出让我赚不到这个钱的话我也无所谓。

不管怎样，为了测试我的理论我打开chrome开发者工具然后限制我的网速到最差的有效值并且禁用了javascript。我打开我网站上的一篇文章用了三秒加载完毕。我尝试用Google搜索也很快，但是我没有看到AMP链接。这当然没有，因为AMP链接只有当javascript可用的时候才出现。

我又启用了javascript（网速还是设置很慢）然后搜索一些AMP内容，结果只为了加载新闻carousel，就花了10秒钟。

正如我想的，静态内容还是更胜一筹。

## 请畅所欲言

九个月前我第一次写了关于AMP的担心。这引起了Hacker News的一些注意，并且Google的AMP人员积极参与了讨论：[Hacker News discussion](https://news.ycombinator.com/item?id=12722590)，[comments of my post](https://news.ycombinator.com/item?id=12731311)。他们甚至邀请我一起吃午餐以便讨论更多我的担心。

两周前我写了一篇相似的文章：[Please Make Google AMP Optional](https://www.alexkras.com/please-make-google-amp-optional/)。它受到了更多关注，但是没有Google的AMP团队参与。

貌似Google的AMP团队意识到和我们这一部分不喜欢AMP的开发者“交战”不会有什么收益。他们知道我们是少数的一部分不是他们的主要受众。我的妈妈和老婆不会上Hacker News。他们不知AMP是什么并且也不关心web问题。

但另一方面Google有很多工程师关心这个web问题。我很惊讶我没有听到一些反对AMP的声音。我猜这是一个高层支持的项目，并且制造一些异议会有一些风险。


但是，我们这些不喜欢AMP的人必须反抗。

你是不是有一个WordPress网站？关掉AMP或者不要激活它。

你是不是为一个发版商工作然后他用着AMP。那跟你的老板说明AMP的缺点，说明它们有可能造成的风险。

幸亏在WordPress上取消AMP很方便。并且不到24小时google搜索结果就没有显示我网站的AMP版本了。

