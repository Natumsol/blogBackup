title: 火星计算器JavaScript实现
date: 2015-09-01 19:43:06
categories: 技术
tags:
	- 算法
	- javascript
---
最近师兄忙着刷算法题，我呢耳濡目染之下也跟着凑热闹，哈哈。虽然牺牲了不少玩的时间，就当为明年找实习做准备咯。

#  题目
自定义运算符 `@,#,$,&,`并且优先级`@ > # > $ > &`,利用下面的计算法则，计算出一个给定的字符串表示的表达式的值，返回整型结果。
x@y = (x-1)\*(y+1);
x#y = (2\*x+5)\*(3\*y+60)
x$y = (x+1)\*(2\*x+3)\*(y-1)\*(2\*y-3)
x&y = ((x+y+1)\*(y-9)+(x+7)/(y-8))
输入：接受多行输入，每行可能有空格。题目保证不会出现除数为0的情况，且所有操作数都是整数。为了简单起见，表达式不含有括号。
输出：对于每一行输入，输出一个运算结果。
```javascript 输入样例：
1@2
1#2
1@1@1
1#1#1
1$1$1
1&1&1
1#1@1
1@1#1
1$1&1
1&1$1
1  #   1  &       1
12 @        12
123    @   123
```

```javascript 输出样例:
0
462
-2
55881
0
186
420
315
-17
-19
-3608
143
15128
```
<!-- more -->
#  思路
拿到这个题目的第一感觉，就是要用逆波兰表达式做。毕竟处理这种四则运算最简单有效的方法就是利用逆波兰表达式进行解析计算，这样就免去了对原表达式进行复杂的语义分析。

题目中定义的四个运算，其实就是新定义了一种运算规则，和我们熟知的四则运算并没有本质上的不同。这样解体思路就很明了了，简述如下：
1. 定义四种运算的优先级，在这里我们利用`javascript`里面的对象来完成：
	`var operator = {"@": 4,"#": 3,"$": 2,"&": 1} //定义运算符优先级`
2. 根据运算符的优先级，对输入进行处理，生成逆波兰表达式。
3. 解析逆波兰表达式，并按照题目中定义的运算法则计算结果。
4. 输出。

# 细节处理
解题思路清楚后，接下来就是代码实现了。这里有个细节需要注意：
怎么正确解析原表达式的数字和运算符？
对于输入“1222 @ 32432 &3”这样的表达式用字符串函数处理的话是很繁琐的，需要考虑大量的情况，而且很可能考虑不全。我的做法是直接利用神器————正则表达式来匹配。这样不仅代码简介，而且可以保证不会出错。
```javascript
//比如输入“1222 @ 32432 &3”， 输出就为["1222", "@", "32432", "&", "3"]
var input = str.replace(/\s/g,"").match(/(\d+)|(@|&|\$|#)/g)
```

# 代码实现


```javascript
/* 生成逆波兰表达式 */
function toReversePolish(str) {
	var input = str.replace(/\s/g,"").match(/(\d+)|(@|&|\$|#)/g)
	var operator = {"@": 4,"#": 3,"$": 2,"&": 1} //定义运算符优先级
	var stackTemp = []; //运算符栈
	var output = []; //输出

	for (var i = 0; i < input.length; i++) {
		if (Number.isInteger(parseInt(input[i]))) { //如果是数字，则直接输出
			output.push(parseInt(input[i]));
		} else {
			if (stackTemp.length != 0) {
				if (operator[input[i]] > operator[stackTemp[stackTemp.length - 1]]) {//当前运算符的优先级比栈定元素高，则当前运算符进栈
					stackTemp.push(input[i]);
				} else {//当前运算符的优先级不比栈定元素高，则栈定元素出栈，当前运算符进栈
					output.push(stackTemp.pop());
					stackTemp.push(input[i]);
				}
			} else {//如果运算符栈为空，则直接进栈
				stackTemp.push(input[i]);
			}
		}
	}
	if (stackTemp.length) { //处理栈中剩下的运算符
		while (stackTemp.length)
			output.push(stackTemp.pop());
	}
	return output;
}

/* 火星计算器 */
function marsCalculator(str) {
	var str = toReversePolish(str);//根据输入得到逆波兰表达式
	var result = [];//结果栈
	for (var i = 0; i < str.length; i++) {
		if (Number.isInteger(parseInt(str[i]))) {
			result.push(str[i]);//如果为数字，则直接进栈
		} else {
			switch (str[i]) {//如果是运算符，则按照运算规则计算
				case "@":
					var y = result.pop();
					var x = result.pop();
					var resultTemp = (x - 1) * (y + 1);
					result.push(resultTemp);
					break;
				case "#":
					var y = result.pop();
					var x = result.pop();
					var resultTemp = (2 * x + 5) * (3 * y + 60);
					result.push(resultTemp);
					break;
				case "$":
					var y = result.pop();
					var x = result.pop();
					var resultTemp = (x + 1) * (2 * x + 3) * (y - 1) * (2 * y - 3);
					result.push(resultTemp);
					break;
				case "&":
					var y = result.pop();
					var x = result.pop();
					var resultTemp = (x + y + 1) * (y - 9)  +  (x + 7) / (y - 8);
					result.push(resultTemp);
					break;
			}
		}
	}
	return result[0]; // 返回结果
}

```
#  测试
![测试结果](/images/blog/20150901/testResult.png)
经测试，程序满足题目要求。
