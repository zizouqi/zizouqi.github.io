---
layout: post
title: 		"自走棋代码优化：避免棋子无意义移动、重复抬手"
subtitle: 	"AutoChess Code Optimization"
author: 	"Unnamed_User"
header-img: "img/post-bg-code.jpg"
header-mask: 0.8
tags:
  - AutoChess
  - 自走棋
  - Code
  - 代码
  - Bug
---

  

> 最近在测试 DPS 数据时发现远程单位经常在射程足够的情况下仍然做出移动行为。  
> 所以我看了下代码尝试找出原因。

![Before](/img/in-post/post-autochess-code-optimization/before.gif)  

## 棋子移动

自走棋代码中，是通过 `local attack_result = FindAClosestEnemyAndAttack(u)` 的结果来判断棋子是进行攻击还是移动： 

```lua
--决定是否要攻击 
local attack_result = FindAClosestEnemyAndAttack(u) 
if attack_result ~= nil and attack_result > 0 then 
	return attack_result + ai_delay 
end 
--不攻击就走动 
if attack_result == nil then 
	--寻路 
	--(略) 
	return RandomFloat(0.5,1) + ai_delay
else
	return 1 + ai_delay
end
```

显然是因为 `FindAClosestEnemyAndAttack(u)` 为空，所以导致了棋子进行移动。  
接下来看看这个函数，函数中判断射程范围内是否有敌人的逻辑条件是“**单位间距离**小于**射程减目标单位半径**”：  
`(u.attack_target:GetAbsOrigin() - u:GetAbsOrigin()):Length2D() < u:Script_GetAttackRange() - u.attack_target:GetHullRadius()`   
以及：  
`d < attack_range - v:GetHullRadius()`

分析以上条件，可能出错的原因有两个：
1. 条件中仅考虑了被攻击目标的单位半径，是否也应该算上攻击者的单位半径？
2. 单位的射程属性是计算模型边缘之间的距离，还是计算模型中心点之间的距离？

我没有 Dota 2 自定义地图的制作经验，也没有相关文档。觉得查起来有点麻烦，于是就直接改代码测试了。  
摸索后，感觉游戏中的距离关系应该如下图：  
![Distance](/img/in-post/post-autochess-code-optimization/range.jpg)

明确距离关系后，将距离判断条件修改为了：  
`(u.attack_target:GetAbsOrigin() - u:GetAbsOrigin()):Length2D() < u:Script_GetAttackRange() + u.attack_target:GetHullRadius() + u:GetHullRadius()`  
以及  
`d < attack_range + v:GetHullRadius() + u:GetHullRadius()`  

经测试，棋子不会再进行多余的移动。

　

## 重复抬手

尽管棋子不再移动，但还是会观察到有攻击前摇被打断，重新抬手的问题（最容易观察到的情况是刺客跳跃后攻击要重复抬手）。  
思考了下，可能的原因是游戏中的单位有默认 AI 模板，当范围内有敌方时便会自动攻击。而自走棋代码中进行了一次额外的索敌操作，会打断之前的动作。  
在这个思路下，我删除了 `FindAClosestEnemyAndAttack` 函数中强行更改攻击目标的两段的相关代码。经过测试，棋子就不再有重复抬手的动作。  
![After](/img/in-post/post-autochess-code-optimization/after.gif)
![Comparison](/img/in-post/post-autochess-code-optimization/comparison.gif)

　

## 近战棋子优化

在正确计算了攻击范围后，近战棋子也可以攻击斜方向的敌人了，而不用像以前一样必须移动到 3、6、9、12 点钟位置。  
![Before](/img/in-post/post-autochess-code-optimization/melee.gif)
以上仅修改了攻击的判断。接下来进一步优化近战单位的 AI，和上面类似需要修改 `IsGridCanAttackEnemy` 中的距离判断代码。
![Path](/img/in-post/post-autochess-code-optimization/path.gif)

　

## TLDR　
![对比](/img/in-post/post-autochess-code-optimization/combine.gif)
[![TLDR](/img/in-post/post-autochess-code-optimization/tldr.png)](/img/in-post/post-autochess-code-optimization/tldr.png)
注释掉第一段以避免攻击时重复抬手，第二段修改判断条件以避免无意义移动。 
