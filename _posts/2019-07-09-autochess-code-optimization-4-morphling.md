---
layout: post
title: 		"自走棋代码优化：水人施法逻辑"
subtitle: 	"AutoChess Code Optimization 4: Morphling"
author: 	"Unnamed_User"
header-img: "img/post-bg-code.jpg"
header-mask: 0.8
tags:
  - AutoChess
  - 自走棋
  - Code
  - 代码
---

  

> 水人不再放水。

## 现状

目前水人和沙王施放技能时，都是用 `function` 来选择一个最远能攻击到敌人的格子来作为目标位置。  
但在面对缩在角落的敌人时，水人的技能有很大可能无法伤害到敌方棋子，如下图所示：  
![]()  

　　

## 优化·1

`function` 是以棋子攻击范围做为条件来判断目标格是否可选的，那么也许我们可以尝试将棋子攻击范围修改为技能有效范围，来确保棋子施放技能时可以造成伤害。  
查询 Valve 提供的 API，发现使用 `GetAbilityValue` 可以获取技能的 KV 表。而对该表进行遍历，则可获取

#### 匹配
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

#### 避免重复对战
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