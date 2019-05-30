---
layout: post
title: 		"自走棋代码优化：提高棋子移动效率"
subtitle: 	"AutoChess Code Optimization 2: Move"
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

  

> 攻击逻辑之后，再来看看移动行为的优化。

## 基本行为

在自走棋中，棋子的单次移动能力和国际象棋中的皇后类似，可以在横、直、斜方向上做不限格数的移动：  
![Basic](/img/in-post/post-autochess-code-optimization-2-move/move-basic.jpg)

棋子的索敌逻辑，则是优先寻找离自己最近且可以攻击到地方的格子进行移动。在一定条件下，这种方式并非为最优解，如图所示：
![Before](/img/in-post/post-autochess-code-optimization-2-move/before.gif)
![Issue](/img/in-post/post-autochess-code-optimization-2-move/issue.jpg)

很显然，在上面的例子中，最优的移动方式是直接做一次斜向跳动，而非跳动两次到最近的可攻击格上。

　　

## 优化

在代码里，非刺客单位的寻路是由 `FindNextSkipPosition` 来控制，这个 function 的逻辑是遍历整个棋盘来找出离自己最近的可攻击格。  
```lua
function FindNextSkipPosition(u)
	local team_id = u.at_team_id or u.team_id
	local skip_postion = nil
	local length2d = 99999
	local pos1 = XY2Vector(u.x,u.y,team_id)
	for x=1,8 do
		for y=1,8 do
			local pos2 = XY2Vector(x,y,team_id)
			if IsGridCanAttackEnemy(x,y,u) == true then
				local next_skip = IsGridCanReach(x,y,u)
				if next_skip ~= nil and (pos2-pos1):Length2D() < length2d then
					skip_postion = next_skip
					length2d = (pos2-pos1):Length2D()
				end
			end
		end
	end
	return skip_postion
end
```

我们可以在依靠 `math.abs(u.x - x) == math.abs(u.y - y)` （横竖位移相等）这个条件来找出用斜向移动能直接到达的格子，之后优先以这个格子为目标进行移动。

上面一步比较简单，实测中达到了优化的初衷。但是也带来了几个问题：
1. 当跳往斜向优先格的路径中有其他棋子做障碍时，他仍然会优先尝试绕路到达。由于每次移动是单独计算，所以这种情况甚至可能造成反复绕路。
![Worse](/img/in-post/post-autochess-code-optimization-2-move/worse.jpg)
2. 斜向移动的优先级更高，所以当直线面对敌方棋子间距一格时，会优先斜向移动而不是直线移动；
![Near](/img/in-post/post-autochess-code-optimization-2-move/near.jpg)
3. 斜向移动的优先级更高，所以棋子有可能攻击离自己更远的目标；
![Far](/img/in-post/post-autochess-code-optimization-2-move/far.jpg)

为了避免第一个问题，我们就需要限制棋子和“优先移动目标”中间没有障碍物，一次跳动可达。  
自走棋中的寻路算法是 `function FindPath`，在这里面有对障碍物进行判断。但是这个算法牵扯的东西太多，进行修改的话非常复杂，所以我决定暂时不碰这一块，先看看有没别的解决方式。  
在源代码中，我发现 `function IsGridCanReach` 就是输入棋子原位置和最终目标位置，调用 FindPath 后输出棋子应该移动的位置。如果目标无法直达，则 FindPath 输出的下一跳位置则和最终目标位置不同。  
那么我们只要加上 `pos2 == IsGridCanReach(x,y,u)` （目标位置和下一跳位置相同）这个条件，就可以解决优先斜向移动带来的绕路问题。  

针对上面的第二个问题，可以用 `(skip_postion_alt-pos1):Length2D() > 200` （优先移动位置和原位置必须超过两格）来避免。  

第三个问题其实不修改的话对近战阵容来说其实是一个提升，因为可以绕过前排攻击对方的脆皮输出。  
不过为了更贴近原来的移动逻辑，我们可以加上 `(skip_postion_alt-skip_postion):Length2D() < 200` 来限制优先移动位置与最近可攻击敌人的位置不超过一格。

　　

## TLDR　
![对比](/img/in-post/post-autochess-code-optimization-2-move/move.gif)

```lua
function FindNextSkipPosition(u)
	local team_id = u.at_team_id or u.team_id
	local skip_postion = nil
	local skip_postion_alt = nil
	local skip_postion_new = nil
	local length2d = 99999
	local length2d_alt = 99999
	local pos1 = XY2Vector(u.x,u.y,team_id)
	for x=1,8 do
		for y=1,8 do
			local pos2 = XY2Vector(x,y,team_id)
			if IsGridCanAttackEnemy(x,y,u) == true then
				local next_skip = IsGridCanReach(x,y,u)
				if next_skip ~= nil and (pos2-pos1):Length2D() < length2d then
					skip_postion = next_skip
					length2d = (pos2-pos1):Length2D()
				end
				if next_skip ~= nil and (pos2-pos1):Length2D() < length2d_alt and math.abs(u.x-x) == math.abs(u.y-y) and pos2 == IsGridCanReach(x,y,u) then
					skip_postion_alt = next_skip
					length2d_alt = (pos2-pos1):Length2D()
				end
			end
		end
	end
	
	if skip_postion_alt ~= nil and (skip_postion_alt-skip_postion):Length2D() < 200 and (skip_postion_alt-pos1):Length2D() > 200 then
		skip_postion_new = skip_postion_alt
	else
		skip_postion_new = skip_postion
	end
	return skip_postion_new
end
```
