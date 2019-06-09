---
layout: post
title: 		"自走棋代码优化：新的对手匹配机制"
subtitle: 	"AutoChess Code Optimization 3: Matching"
author: 	"Unnamed_User"
header-img: "img/post-bg-code.jpg"
header-mask: 0.8
tags:
  - AutoChess
  - 自走棋
  - Code
  - 代码
---

  

> “打得好不如排得好。”——鲁迅

## 现机制

现在自走棋使用的对手匹配机制，可以称为“错位”匹配。  
简单解释的话，就是有一张主场表和一张客场表，两张表里面初始都按玩家 1-8 的顺序排列。  
每轮取一个随机值 N，加到客场表里面成为新的玩家顺序，然后两张表的玩家一一对应匹配对战。  
N 的取值范围为 1 至 “存活玩家数量 -1”。加上 N 后超过 8 的值从 1 开始重新数。  
另外代码逻辑上确保了 N 的值与上一轮不同，即没有玩家出局的情况下，不会连续两轮匹配同一对手。  

下面的表可能更便于理解：  

主场表 | 客场表 | N=2时的客场表 | 对战匹配结果
-|-|-|-
玩家1|玩家（1+N）|玩家3|玩家1 vs 玩家3
玩家2|玩家（2+N）|玩家4|玩家2 vs 玩家4
玩家3|玩家（3+N）|玩家5|玩家3 vs 玩家5
玩家4|玩家（4+N）|玩家6|玩家4 vs 玩家6
……|……|……|……

这种机制下，对单个玩家来说问题不大，确实是按相同概率来分配到对手的。  
但是从整体上来说，缺少了对战多样性。比如场上有 8 名玩家时，一共只有 7 种对战情况。而就随机的角度来说，理论上应该有数千种对战方式。（我概率学得不怎么好，具体数字就不算了。）

　　

## 优化

此次优化的目的，就是丰富对战的多样性。另外，我还增加了基于存活玩家人数来控制连续多少轮不碰上相同对手的功能。  
这里就解释下新的匹配算法，以场上剩余四名玩家举例。  

### 匹配
首先，我们在统计存活玩家的同时，在玩家编号前加上 1-99 的随机值。并将所有结果存放在 `GameRules:GetGameModeEntity().alive_player_table` 内。
```lua
for u,v in pairs(GameRules:GetGameModeEntity().counterpart) do
	if TeamId2Hero(u) ~= nil and TeamId2Hero(u):IsNull() == false and TeamId2Hero(u):IsAlive() == true then
		--活玩家
		alive_player_count = alive_player_count + 1
		table.insert(GameRules:GetGameModeEntity().alive_player_table, RandomInt(1, 99) * 100 + u)
	else
		--死玩家
		GameRules:GetGameModeEntity().counterpart[u] = -1
	end
end
```

可以假设 `alive_player_table` 内的结果为 `4806, 2807, 9308, 1809`。  
此时将 `alive_player_table` 从小到大排序得到结果 `1809, 2807, 4806, 9308`。
去掉前两位随机数，根据表内顺序让前一个玩家编号对战后一个玩家，并将分配结果存放在 `GameRules:GetGameModeEntity().current_round` 内：
```lua
for i, v in pairs(GameRules:GetGameModeEntity().alive_player_table) do
	local home = GameRules:GetGameModeEntity().alive_player_table[i] % 100
	local away = nil
	if GameRules:GetGameModeEntity().alive_player_table[i + 1] ~= nil then
		away = GameRules:GetGameModeEntity().alive_player_table[i + 1] % 100
	else
		away = GameRules:GetGameModeEntity().alive_player_table[1] % 100
	end
	GameRules:GetGameModeEntity().counterpart[home] = away
	table.insert(GameRules:GetGameModeEntity().current_round, home * 100 + away)
end
```
得到当前回合对战结果 `current_round` 为 `0907, 0706, 0608, 0809`。

### 避免重复对战
预期设计：  
当有 7、8 名玩家时，则 3 回合内不匹配相同玩家。  
当有 5、6 名玩家时，则 2 回合内不匹配相同玩家。  
当有 3、4 名玩家时，则 1 回合内不匹配相同玩家。  

储存最近三回合的匹配结果：
```lua
GameRules:GetGameModeEntity().last_3rd_round = GameRules:GetGameModeEntity().last_2nd_round
GameRules:GetGameModeEntity().last_2nd_round = GameRules:GetGameModeEntity().last_round
GameRules:GetGameModeEntity().last_round = GameRules:GetGameModeEntity().current_round
```

检测是否有重复匹配：
```lua
while finished == false and trytime < 10000 do
	trytime = trytime + 1
	GameRules:GetGameModeEntity().current_round = {}

	finished = true
	for i, v in pairs(GameRules:GetGameModeEntity().current_round) do
		if alive_player_count >= 7 then
			finished = not (IsValueInTable(v, GameRules:GetGameModeEntity().last_round) or IsValueInTable(v, GameRules:GetGameModeEntity().last_2nd_round) or IsValueInTable(v, GameRules:GetGameModeEntity().last_3rd_round))
		elseif alive_player_count >= 5 then 
			finished = not (IsValueInTable(v, GameRules:GetGameModeEntity().last_round) or IsValueInTable(v, GameRules:GetGameModeEntity().last_2nd_round))
		elseif alive_player_count >= 3 then 
			finished = not (IsValueInTable(v, GameRules:GetGameModeEntity().last_round))
		elseif alive_player_count >= 1 then 
			finished = true
		else
			return
		end
	end
end

function IsValueInTable(value, table)
	for k,v in pairs(table) do
	  if v == value then
	  	return true
	  end
	end
	return false
end
```

　　

## TLDR
完整的代码实现请见：[DAC Feedback](https://github.com/zizouqi/DAC-Feedback/blob/master/2019-06/%5B0609%5D%E4%BC%98%E5%8C%96%EF%BC%9A%E6%96%B0%E7%9A%84%E5%AF%B9%E6%89%8B%E5%8C%B9%E9%85%8D%E6%9C%BA%E5%88%B6/addon_game_mode.lua)