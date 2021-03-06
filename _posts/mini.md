---
title: 腾讯SNG雏鹰计划Mini项目杂谈
date: 2016-08-02 16:42:00
tags: 
	- 实习
---
经过小组成员九天九夜的奋战，我们的Mini项目——闲鹅，在上周五获得了评委们的肯定，赢得了金奖。这让我们无比的自豪和激动，想起那些熬夜加班的日子，什么都值了！
## 分组
在分组前，我就向做过Mini项目的室友了解了一些情况，他给我的建议是：作为一名技术人员，要准备好推销自己。本来以为自己精心准备的“广告语”能在自我介绍时大放光彩，没想到因为前台和后台的人数众多，自我介绍环节就取消了...取消了...然后PM就开始了紧张的抢人工作，我也开始向PM毛遂自荐。作为一名前后台通吃的万金油前端，应该很抢手才是，但令人意外的是处处碰壁，每当我说明自己是一个前端后，产品们都表达出自己需要的是做APP的，而不是写页面的...写页面的...最后的最后，我和同桌的Stark来到了“黄金海盗船”，由此，故事拉开了序幕...

##  选题
小组人员确定后，因为时间很紧，只有短短的9天，所以必须尽快确定下来做什么。经过头脑风暴后，大家一致认为做一个面向鹅场的二手交易平台是很有市场和现实意义的。选题确定后，可能大部分同学之前没有参与过web系统的设计与实现，大家七嘴八舌，莫衷一是，这样下去不仅浪费了时间，还会造成团队的涣散。因为自己之前在实验室做过两年的Web系统的开发，参与了两个项目从设计到实现的全过程，所以这方便比较有经验。于是自告奋勇的当起了“项目负责人”， 组织后台人员和前台开发人员梳理需求、设计表结构和API。因为大家都是新来的实习生，为了方便大家更好的协同工作，依据之前在创业团队待过的经验，利用`worktile`这个平台搭建起了一个协同工作环境，这样大家的文档（包括需求文档和API文档等等）都可得到很好的迭代，项目进度也可以得到很好的把控。
<!-- more -->

##  技术选型
等到API确定后，大家就面临一个比较大的问题了——技术选型。为什么说它"大"呢？这是因为技术选型的成功与否直接关系到接下来开发工作的顺利与否。对于前端来说，如果是写传统的页面，那工作量是可想而知的大，为了加快开发效率，我们选用了目前非常火爆的`React`框架，配合`Webpack`进行打包，`Babel`进行编译，`Gulp`进行自动化管理。因为小组没有美工，我们选用了`Material-UI`作为我们的UI套件，它不仅用React完美的实现了谷歌`Material Design`的设计理念，而且充分的组件化，各组件之间耦合度低，写View的时候，就像搭积木一样，非常快速和方便。

## 开发
开发是所有环节中最时间最紧张的。为了兼顾开发效率和技术含金量，我们最后采用了利用`React`来开发一个单页面应用`SPA`（Singal Page Application）。因为因为是`SPA`，所以基本上前后端完全分离了，所以为了加快开发效率，我们在确定好前后台交互API后，就开始“闭门造车”了，利用`Fiddler`和`MockServer`伪造后端数据，完全不用等后台的接口是否完成，等后台开发完成后，联调对接就好了。事实证明，这种开发模式确实让前后端节省了许多沟通成本，大家都各司其职，专心Coding，避免了经常被打断的尴尬。当然，开发的过程中也遇到了不少问题，自己之前虽然接触过`React`，但是也只停留在写写小Demo的程度，而且是跟着书本上走的，比较初级，而现在的任务是利用React来写一个用于生产环境的系统，所以这是一个不小挑战。所幸的是，同组的Stark同学`React`经验比较丰富，带领我一步步走，在开发过程中给予了我很大的帮助，在这里感谢Stark同学！

## 总结
本次Mini项目的成功是小组成员共同努力的结果。产品的PPT超赞，以一则微博引出话题，然后市场调研，用户分析和SWOT分析等等娓娓道来，相比于其他组纯粹的技术分享和功能介绍，我们
成功的征服了评委们的心，看来鹅唱“人人都是产品经理”所言不虚。后台的开发效率也超赞，技术选型时确定的是一个大家都不熟悉的框架，但给力的是两天就把API全部写完，而且架构上用到了很多大型系统才会用到的技术：负载均衡，主从备份和数据库读写分离，超屌的有木有。安卓组也超棒，说的是“安卓组”，其实只有一个人，刷夜两天，成功的完成了安卓客户端，单之无愧的“单挑大神”。测试组也相当棒，准备工作做得到很充分，在最后答辩的时候，成功地给评委们“致命一击”，吹响了胜利的号角。当然，我们前端组也是很厉害的啦，哈哈。

在庆祝的时候也不该忘了反思，本次Mini项目也暴露了一些问题：
1. 团队协作没有充分的展开。虽然大家利用了团队协作平台`worktile`，但是在实际开发过程中基本上“单干”。
2. 前期准备工作不足。API没有经过详细的设计，造成开发过程中频繁的改动，一度拖慢开发进度。

当然这些问题对于我们新组建的团队来说是很正常的，希望经过本次Mini项目，大家能吸取教训，更好的投入到本职工作中去。

附：
![](/images/blog/20160803/Mini.jpg)


