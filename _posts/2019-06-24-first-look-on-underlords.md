---
layout: post
title: 		"从后台文件聊聊《刀塔霸业》"
subtitle: 	"First Look On Underlords"
author: 	"Unnamed_User"
header-img: "img/post-bg-code.jpg"
header-mask: 0.8
tags:
  - AutoChess
  - 自走棋
  - Code
  - 代码
  - 刀塔霸业
  - Underlords
---

  

> V 社历居然准了？

如 Valve 所承诺的一样，经过内测一周之后，刀塔霸业在电脑端和移动端同步上线开始公测了。
在大家可以上手游玩之后，整个游戏的基本框架和玩法已经不再是秘密。而这篇文章则是根据程序后台文件，来聊聊目前尚不为人知的花絮以及未来的各种可能性。


## 服务器
做为一款多人游戏，连接服务器的稳定性是决定玩家游戏体验很重要的一环。从后台来看，刀塔霸业的服务器在亚洲分布于香港、台湾、新加坡、日本四个地方。
![](/img/in-post/post-first-look-on-underlords/server.jpg)
至于连接哪个服务器，现在是由游戏自动决定的，玩家无法自己选择。不过据我的测试来看，可能和游戏语言有一定关系。
我的客户端默认是英语，第一次进游戏时场上没有一个中文 ID 的玩家，抓包看了下 IP 地址是连到了 Valve 在斯德哥尔摩的服务器。
之后将游戏设置为中文，再抓包查看则是连接到了香港服务器。
(不过有在欧洲的朋友反应他们用中文语言时连的也仍然是欧洲服务器。)


## 棋盘
可能有的玩家之前已经看到过视频了，目前游戏中是存在其他棋盘的。
具体来说，除了默认棋盘之外，后台还有名为“森林(Forest)”、“丛林(Jungle)”、“测试(Test)”的三个棋盘，他们的样子可见下图。
![](/img/in-post/post-first-look-on-underlords/test.jpg)
![](/img/in-post/post-first-look-on-underlords/forest.jpg)
![](/img/in-post/post-first-look-on-underlords/jungle.jpg)
“测试”很明显就是低精度建模纯粹用来测试的，而另外两个亮色的棋盘可能会更讨国内玩家喜欢吧。 
其实国外玩家也说过，由于不像游廊版自走棋那样是大地图并且有交互，刀塔霸业玩起来感觉很孤独甚至有压抑感，感觉像是被困在地牢中一样。今后可以换成亮色棋盘的话也许会有所缓解。


## 饰品
刀塔 2 本身的饰品设定非常有特色，甚至被一些玩家玩成了“遗迹暖暖”。所以在刀塔霸业公布的时候，很多刀塔 2 老玩家都在期待两个游戏间饰品可以互通。
那么有这个可能性吗？我们来看看两个游戏的皮肤机制。在刀塔 2 里的英雄模型代码中，首先是有人物的基本模型，然后可以设置在不同部位装配的饰品。
下图是一个剑圣的例子，可以看到有“武器”、“头部”、“手臂”这些部件位置。(部件位置太多后面的我就没截图了)：
![](/img/in-post/post-first-look-on-underlords/jugg-code.jpg)
而在刀塔霸业中，对一个三星末日守卫的设定是这样的：
![](/img/in-post/post-first-look-on-underlords/doom.jpg)
我们可以看到，这里的三星末日守卫只用了一个整体模型，并没有各个部件的选项，也就是说至少目前 Valve 还没有让饰品互通的打算。
这也许会让一些刀塔 2 玩家失望了。不过对新玩家来说，也不会有刚进游戏就看见对方棋子全身粒子特效，自己却只有一身破烂的失落感了。
但毕竟游戏目前还在 beta 阶段，再联系到之前 Valve 以 2000 美元价格和创意工坊作者签约一套英雄饰品的新闻来看，也许今后还是有一定程度上在刀塔霸业中使用 Dota 2 饰品的可能性吧。


## 通行证
设置通行证，几乎已经是现在游戏的惯例了。其实之前在程序后台就可以看到一些相关图片文件，今天公测时同步上线的官网内容中，Valve 也不出所料正式宣布了刀塔霸业会有 Battle Pass 通行证，其中内容就包括了上面提到的棋盘和饰品。
![](/img/in-post/post-first-look-on-underlords/battlepass-pic.jpg)
![](/img/in-post/post-first-look-on-underlords/battlepass-text.jpg)
同样的，我们在后台代码中也找到了一些关于通行证的信息。
![](/img/in-post/post-first-look-on-underlords/battlepass-code.jpg)
这个文件描述中的 bp1 应该是指 BattlePass 第 1 级，里面的一些奖励描述如边框、宝箱、头衔这些都并不出人意料。
但是其中“攻击伤害加成”、"生命值加成"、"技能伤害加成"这三项内容就比较耐人寻味了，难道 Valve 也会搞“充值才能变强”那一套？
我个人还是不怎么相信 V 社会简单粗暴在 PvP 游戏中弄出这种属性加成的，这也许是纯测试占位之用，也可能是未来加入 PvE 模式时的一些效果。Valve 应该还是有基本操守的，我们就等着再看吧。


## Valve 996
Valve 这次对刀塔霸业的投入确实让人吃惊，甚至在周末时都有数个更新放出，让人看不到半点“度假社”的影子，反而似乎变成了国内 IT 企业常见的 996 工作制。国外玩家在 Twitter 上甚至都开玩笑说“这不是真的 V 社，你们把真正的 V 社怎么了！”
这次 Valve 的工作效率有多高，我们从后台文件中也可以窥见一斑。
![](/img/in-post/post-first-look-on-underlords/model-date.jpg)
从上面的代码中我们可以看到，V 社最先做出来有时间标记的棋子是先知，创建时间是 5 月 15 日。刀塔霸业的内测开始于 6 月 14 日，那我们几乎可以猜测，Valve 在全力投入一两个月后，就把这个游戏做了出来。
刀塔霸业目前应该还算是在 beta 阶段，相信“更新霸主”这个戏称还会持续一段时间了。
![](/img/in-post/post-first-look-on-underlords/update.jpg)


## 花絮
刀塔霸业源自游廊版自走棋，这一点是无需多言了。Valve 在自己的公告中也感谢了巨鸟多多并大方为他们做了宣传。不过在看后台文件之前，我都没想到 Valve 能“大方”到这种地步。
![](/img/in-post/post-first-look-on-underlords/valve.jpg)
上面是 V 社其他游戏 Dota 2、Artifact、CS:GO 的程序目录结构，都能直接看到项目名(dota 和 csgo 一目了然，dcg 则是 Dota/Digital Card Game 的缩写)。
而刀塔霸业，是这样的：
![](/img/in-post/post-first-look-on-underlords/underlords.jpg)
DAC？是不是很熟悉了？再看看巨鸟多多游廊自走棋的项目名呢……
![](/img/in-post/post-first-look-on-underlords/dac.jpg)
没错，DAC 正是 Dota Auto Chess。

**Valve，你可也太耿直了。**