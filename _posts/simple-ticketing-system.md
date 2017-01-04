title: 简单的火车票售票系统
date: 2015-09-06 09:39:38
categories: 技术
tags:
	- 算法
	- JavaScript
---
# 题目

实现一个简单的火车票售票系统，支持如下功能：
1. 设置火车路线及总的待售火车票（乘车区间为从路线始发站至终点站）。
2. 销售火车票。售票时可以拆分待售火车票，详细规则见下面的售票规则.
3. 查询包含指定乘车区间的代售火车票。

## 名词解释:
火车线路：一条火车线路为单向，由一个始发站、N个中间站（N>0）和一个终点站组成，线路自身不存在交叉和环。为了简化，只要求系统支持一条火车线路。
乘车区间：指在火车线路方向上，乘车从发站至到站组成的区间。
火车票：火车线路上的乘车凭证。火车票具有乘车区间的属性。
待售火车票：指系统中未售出的火车票，包括车票拆分时剩下区间形成的火车票。
## 售票规则
购买一张指定乘车区间(发展A，到站B)的火车票：
### 规则1
检查是否包含指定乘车区间的火车票。如果不存在，售票失败。如果存在，从中选取满足如下条件的一张火车票：
* 先选择发站与指定乘车区间发站A距离最近的。
* 如果满足条件1的有多张，再选择到指定乘车区间到站B最近的。
* 如果满足条件2的有多张，任选一张。

### 规则2
对规则1中选取的这张火车票（发站S，到站D）进行拆分，拆出最多三张乘车区间票:A->B（指定乘车区间比）；S->A（如果S与A相同，则不存在）；B->D（如果B与D相同，则不存在）。A->B的火车票售出，其他两个剩余区间票如果存在，成为新的待售火车票。
*说明：待售火车票不能合并。*

## 输入
``` javascript
整数stationNumber， 火车站点数目，2 <= stationNum <= 30
一行整数，共stationNumber个。站点标示ID，每一个站点用一个唯一的ID标识。第一个站点标识始发站，最后一个站点表示终点站。
整数ticketCount，从线路始发站至终点站的代售火车票，1 <= ticketCount <= 1000。
后续不定行，每一条命令格式为：
`command [para]`
比如出售一张指定乘车区间的火车票
`sellTicket srcStationId destStationId`
参数:
srcStationId:乘车区间的发站
destStationId:乘车区间的到站
输出：若成功，则不输出；若失败，则输出-1。
查询系统中包含指定乘车区间的待售火车票
`findLeftTickets srcStationId destStationId`
参数：
srcStationId:乘车区间的发站
destStationId:乘车区间的到站
```
## 输出

```
包含指定乘车区间的待售火车票，包含与指定区间完全相同的火车票。如果没有则返回0,入参非法也返回0。
```

# 输入输出模拟

```javascript 输入示例：
6
1 2 3 4 5 6
10
findleftTickets 1 6
sellTicket 1 6
findleftTickets 1 6
```


```javascript 输出示例：
	10
	9
```

<!--more -->
# 解题思路

从火车票的乘车区间是连续的大致可以得出需要用线性表来解这道题。具体的数据结构可以选择链表或者顺序表（数组实现）。为了方便，这里采取顺序表。

要解决这个问题，需要编写两个函数，一个是用来寻在满足要求的火车票`findLeftTickets`；一个是用来卖票（找到最适合的火车票，进行拆分，并移除已卖出的火车票）`sellTicket`。

`findLeftTickets`实现起来很容易，只要遍历所有的火车票，比较所有火车票的起始站、终点站与指定乘车区间的起始站、终点站的大小关系，如果`ticket[0] <= srcStationId && ticket[ticket.length -1] >= destStationId`。那么这张票就是符合要求的；否则，不符合要求。

`sellTicket`实现起来稍微复杂一点，因为要根据选票规则从众多候选票里选出一张最合适的。这就要用代码实现题目中的选票规则，选票规则用伪代码可以描述如下：
``` javascript sellTicket 伪代码实现
var tickets = findLeftTickets();
var smallset //定义比较的标准
var filtertickets = [];
var selected; //最终被选中的火车票
//规则1
for (var i = 0; i < tickets.length; i++) {
	if (smallest == srcStationId - tickets[i][0]) {
		filtertickets.push(tickets[i]) //满足距离srcStationId最近的火车票有多张
	} else if (smallest < srcStationId - tickets[i][0]) {
		while (filtertickets.length) {
			filtertickets.pop; //找到距离srcStationId更近的火车票，则移除非距离srcStationId非最近的。
		}
		filtertickets.push(tickets[i]); //将距离srcStationId最近的保存下来
	}
}

if (filtertickets.length == 1) {
	selected = filtertickets[0];
	拆分火车票();
	return;
}

//规则2
tickets = filtertickets;
filtertickets = []; //为第二次过滤初始化
for (var i = 0; i < tickets.length; i++) {
	if (smallest == tickets[i][tickets[i].lenght - 1] - destStationId) {
		filtertickets.push(tickets[i]) //满足距离destStationId最近的火车票有多张
	} else if (smallest < tickets[i][tickets[i].lenght - 1] - destStationId) {
		while (filtertickets.length) {
			filtertickets.pop; //找到距离destStationId更近的火车票，则移除非距离destStationId非最近的。
		}
		filtertickets.push(tickets[i]); //将距离destStationId最近的保存下来
	}
}
selected = filtertickets[0]; //无论是否有多张，选第一张总没错
拆分火车票();
return;
```

#  细节处理

因为这题要用到手动输入输出，考虑到在浏览器端输入输出的诸多不便，故转移到NodeJS平台。在`NodeJS`平台处理控制台输入输出和传统的`C/C++/Java`有很大的不同，究其原因，是`NodeJs`的异步特性造成的。不过要在`NodeJs`里处理输入输出也是很容易的，利用内置的`process.stdin`就可以方便的处理，为了清晰起见，举个简单的栗子:
比如，我想通过用户的输入给`str`这个变量赋值，可以这样处理：
```javascript
process.stdin.setEncoding('utf8');
var str;
process.stdin.on('readable', function () {
	var chunk = process.stdin.read();
	if (chunk !== null) {
		str = chunk;
		console.log(str);
	}
});

```
但是这样有个问题：每次输入都会为`str`重复赋值。这点就是和`C/C++/Java`有很大差异的地方。在`C/C++/Java`中我们会为某个变量显示的调用输入输出函数一次，不存在这样重复赋值的问题。这主要因为`NodeJS`为所有输入事件绑定了一个函数，只要有输入，就会调用输入处理函数，如果我们不对`str`的赋值语句加以限制，每次有输入来的时候，`str`都会被重新赋值。为了繁殖这种事发生，我们可以加一点限制条件：
```javascript
process.stdin.setEncoding('utf8');
var str;
process.stdin.on('readable', function () {
	var chunk = process.stdin.read();
	if (chunk !== null) {
		if(!str) {
			str = chunk;
			console.log(str);
		}
	}
});

```
这样只有当`str`未被初始化时，才能为其赋值。

如果我们要给多个变量赋值，当所有的赋值操作完成时要进行程序初始化操作，该怎么办呢？这里我们可以利用`NodeJS`的事件机制，很容易的完成，举例说明如下：
```javascript
process.stdin.setEncoding('utf8');
var events = require("events");
var emitter = new events.EventEmitter(); //事件发射器
var str1, str2, str3;
process.stdin.on('readable', function() {
	var chunk = process.stdin.read();
	if (chunk !== null) {
		if (!str1) str1 = chunk;
		else if (!str2) str2 = chunk;
		else if (!str3) {
			str3 = chunk;
			emitter.emit("init");
		}
	}
});
emitter.addListener("init", function() {
	console.log("str1:" + str1);
	console.log("str2:" + str2);
	console.log("str3:" + str3);
})
```
这样，当`str1,str2,str3`赋值完成后，`emitter`就会发射一个`init`事件，事件监听程序收到`init`事件后，就会调用的`init`事件注册的回调函数。

#  代码实现

```javascript
process.stdin.setEncoding('utf8');
var events = require("events");
var emitter = new events.EventEmitter();
var stationNumber, stationId, ticketCount, command, tickets = [];
Array.prototype.clone = function () {
	return this.slice(0);
}
//处理输入输出
process.stdin.on('readable', function () {
	var chunk = process.stdin.read();
	if (chunk !== null) {

		if (!stationNumber) {
			stationNumber = parseInt(chunk);
		} else if (!stationId) {
			stationId = chunk.split(" ");
			for (var i = 0; i < stationId.length; i++) {
				stationId[i] = parseInt(stationId[i]);
			}
		} else if (!ticketCount) {
			ticketCount = parseInt(chunk);
			emitter.emit("init");
		} else {
			command = chunk.trim();
			emitter.emit("command", command);
		}
	}
});
// 处理命令
emitter.addListener("command", function (command) {
	if (!command) return;
	if (command == "q") process.exit(1);
	else {
		var command = command.match(/\w+/g);
		if (command[0] == "findLeftTickets") {
			console.log(findLeftTickets(command[1], command[2]).length);
		} else if (command[0] == "sellTicket") {
			sellTicket(command[1], command[2], tickets);
		}
	}
});

// 初始化
emitter.addListener("init", function () {
	for (var i = 0; i < ticketCount; i++) {
		tickets.push({
			station: stationId.clone(),
			id: uuid()
		});
	}
})
process.stdin.on('end', function () {
	process.stdout.write('end');

});
// uuid生成器
function uuid() {
	var s = [];
	var hexDigits = "0123456789abcdef";
	for (var i = 0; i < 36; i++) {
		s[i] = hexDigits.substr(Math.floor(Math.random() * 0x10), 1);
	}
	s[14] = "4"; // bits 12-15 of the time_hi_and_version field to 0010
	s[19] = hexDigits.substr((s[19] & 0x3) | 0x8, 1); // bits 6-7 of the clock_seq_hi_and_reserved to 01
	s[8] = s[13] = s[18] = s[23] = "-";

	var uuid = s.join("");
	return uuid;
}

function sellTicket(srcStationId, destStationId, tickets) {
	srcStationId = parseInt(srcStationId);
	destStationId = parseInt(destStationId);
	if (srcStationId >= destStationId) return console.log(-1);
	var leftTickets = findLeftTickets(srcStationId, destStationId);
	var filteredTickets = [];
	var selected;
	if (!leftTickets.length) return console.log("-1");
	else {
		var smallest = srcStationId - leftTickets[0].station[0]; // smallest >= 0;
		for (var i = 0; i < leftTickets.length; i++) {
			if (srcStationId - leftTickets[i].station[0] == smallest) {
				filteredTickets.push(leftTickets[i])
			} else if (srcStationId - leftTickets[i].station[0] < smallest) {
				smallest = srcStationId - leftTickets[i].station[0];
				while (filteredTickets.length) {
					filteredTickets.pop();
				}
				filteredTickets.push(leftTickets[i]);
			}
		}
		if (filteredTickets.length == 1) {
			selected = filteredTickets[0];
			splitTicket(selected, tickets, srcStationId, destStationId)
			return;
		}

		leftTickets = filteredTickets;
		filteredTickets = [];
		smallest = leftTickets[0].station[leftTickets[0].station.length - 1] - destStationId;
		for (var i = 0; i < leftTickets.length; i++) {
			if (leftTickets[i].station[leftTickets[0].station.length - 1] - destStationId == smallest) {
				filteredTickets.push(leftTickets[i])
			} else if (leftTickets[i].station[leftTickets[0].station.length - 1] - destStationId < smallest) {
				smallest = leftTickets[i].station[leftTickets[0].station.length - 1] - destStationId;
				while (filteredTickets.length) {
					filteredTickets.pop();
				}
				filteredTickets.push(leftTickets[i]);
			}
		}
		selected = filteredTickets[0];
		splitTicket(selected, tickets, srcStationId, destStationId);
	}
}

function findLeftTickets(srcStationId, destStationId) {
	srcStationId = parseInt(srcStationId);
	destStationId = parseInt(destStationId);
	var result = [];
	for (var i = 0; i < tickets.length; i++) {
		if (tickets[i].station[0] <= srcStationId && tickets[i].station[tickets[i].station.length - 1] >= destStationId) {
			result.push(tickets[i]);
		}
	}
	return result;
}

function splitTicket(selected, tickets, srcStationId, destStationId) {
	var selectedId = selected.id;
	selected = selected.station;
	var left = srcStationId - selected[0];
	var right = selected[selected.length - 1] - destStationId;
	var ticket;
	if (left > 0) {
		ticket = {
			station: selected.splice(0, left),
			id: uuid()
		}
		tickets.push(ticket);
	}
	ticket = {
		station: selected.splice(0, destStationId - srcStationId + 1),
		id: uuid()
	}
	// console.log("ticket[" + srcStationId + ", " + destStationId + "] " + "was sold!!");
	if (right > 0) {
		ticket = {
			station: selected,
			id: uuid()
		}
		tickets.push(ticket);
	}
	var index;
	for (var i = 0; i < tickets.length; i++) {
		if (tickets[i].id === selectedId) {
			index = i;
			break;
		}
	}
	tickets.splice(index, 1); // 删除原票
}
```

# 测试
![测试结果](/images/blog/20150906/testResult.png)
经测试，程序满足题目要求。
