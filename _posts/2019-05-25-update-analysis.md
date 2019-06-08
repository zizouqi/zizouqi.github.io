---
layout: post
title: 		"19.05.25 自走棋更新详细解析"
subtitle: 	"AutoChess Update 190525 Analysis"
author: 	"Unnamed_User"
header-img: "img/post-bg-autochess-courier.jpg"
header-mask: 0.4
tags:
  - AutoChess
  - 自走棋
---

![Sven](/img/in-post/post-190525/sven.jpg)

>本次的版本更新中，最值得关注的是招募棋子优化、新增棋子流浪剑客、重做巫妖。  
>下文就将对这三点进行解读。

　　

## 流浪剑客“碎神之力”：  

“碎神之力”自带 50% 溅射效果，而且这个溅射范围大于目前狂战斧的范围。  

![Cleave](/img/in-post/post-190525/sven-cleave.jpg)
<center>流浪剑客攻击溅射范围</center>

“碎神之力”主动施放的效果为：流浪剑客这下NB了，他和所有友方恶魔获得60秒的攻击力大幅提升（增加 100%/200%/300% 的攻击力）。

~~另外，释放技能的流浪剑客提升攻击力时不会考虑道具加成，但是其他恶魔（也包括另一个流浪剑客，如果有的话）身上的道具以及灵魂守卫变身后加的攻击力，可以享受“碎神之力”加成。（这个可能是 BUG，因为按下 Alt 看流浪剑客的技能描述，里面有写“基于棋子的基础攻击力”。）~~

**0525.1742更新：**上述 BUG 已被修复，现在攻击力提升仅基于基础攻击力。  

　　

在叠加方面，“碎神之力”比较复杂。从机制上来说，这个技能其实有两个 BUFF，一个作用于施法单位（流浪剑客）自身，另一个作用于其他恶魔：

1. 单个流浪剑客重复释放技能不会叠加（如刷新球，神族减CD后释放）；

2. 如果有两个流浪剑客，对流浪剑客自身来说可以加法叠加（举例：两个一星流浪剑客释放技能后，攻击力为初始的 3 倍）；

3. 对其他恶魔来说，只能享受一个“碎神之力” BUFF。如果有两个不同星级的流浪剑客，攻击力提升数值以较后释放的技能等级为准。（举例：场上有一个影魔，一个一星流浪剑客，一个二星流浪剑客。如果一星的后放技能，则影魔的攻击力增加 100%；如果二星的后放技能，则影魔攻击力增加 200%。）

![Buff](/img/in-post/post-190525/sven-buff.jpg)
<center>如果有两个两星，流浪剑客加恶魔双 BUFF 后一刀就是接近1900 的伤害。</center>

流浪剑客因为自带溅射，所以回蓝效率是远远强于其他棋子的。理论上来说，在不被敌法师克制的情况下，一刀只要溅射到五个敌方棋子就能满蓝。（非法系棋子一次攻击最高 10 点回蓝，恶魔 BUFF 的额外伤害会视为一次额外攻击，两者相加即为对单个棋子造成伤害后共计回蓝 20 点。）

恶魔流是否能崛起，肯定将是一段时间内值得探讨的话题。不过我个人来说，还是不太看好依赖橙卡的阵容。

　　

## 巫妖“冰霜魔盾”：

施加一个冰霜魔盾环绕友方目标棋子，增加它的护甲（10/15/20）。魔盾开启时，每秒都会对附近4格的敌方棋子放出冰霜魔法，造成少量伤害（20/30/40）。攻击魔盾的敌方棋子将受到寒气的影响，攻击速度降低(-30/-45/-60，持续 2 秒)。

![Lich](/img/in-post/post-190525/lich-shield.jpg)
<center>“冰霜魔盾”伤害范围及减速效果测试</center>

单就技能来说，这是一个不错的辅助技能。上图可见月之女祭司攻击 2级冰盾后，攻速从 1.0 降低为了 1.82，另外实测 1.3 攻速的棋子（如船长）攻速会降低为 2.36。

“冰霜魔盾”甚至可以说得上媲美元素羁绊的效果。虽然减速不比晕眩，但该技能对远程也有小。不过因为并非是直接伤害技能，施法后无法瞬间回蓝，所以 CD 虽然短，但还是难以与神族配合。

受益于亡灵户口，相信巫妖在前中期做为一个过度棋子还是会有较高登场率的。

　　

## D+ 机制：

当你招募棋子的时候，将不会招募到你上次招募棋子弃掉未购买的棋子。

**190526.0958更新：**这里补充一下。虽然名为“D+”，但不论是自己手动 D，还是每回合开始时自动刷新棋子，这个机制都是生效的。如果有兴趣的话，可以参见游戏源代码：https://pastebin.com/gsnmy51x

新的机制让玩家在搜卡时更容易找出匹配自身阵容的棋子，同时也降低了棋库扩容带来的影响，让今后添加新棋子有更大空间。整体上来说，地精赌狗流和德鲁伊可能收益更大一点。

这个改动后，有两点值得注意的地方。

1. 在之前的版本中，我们经常会在不影响利息的情况下购买自己用不上的高费棋子，以此提升获取特定目标棋子的概率。而在更新后，这项操作就完全没有必要了，甚至会带来副作用。

2. 在开局时，我们的金钱仅能购买一个棋子，则剩下的棋子在第二轮时都不会出现。如果我们希望购买前期强力棋子的同时，又保留使用其他套路机会的话，那么可以先购买今后想使用的棋子卖掉，再购入现在想使用的棋子。

![Roll](/img/in-post/post-190525/roll.jpg)

举个例子就是先买入小黑又卖掉，再买赏金放上场。这样在第二轮自动抽卡时也有机会刷新出小黑了。另外还可以扩展开来，比如第一轮有两个相同的想使用的棋子，那么就先买入又卖出其中的一个。

总的来说，在钱不够买入棋子又不想锁的时候，都可以用这个技巧。