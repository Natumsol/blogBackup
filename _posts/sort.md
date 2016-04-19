title: 常见排序算法的C语言和JavaSript实现之 插入排序
date: 2016-2-14 17:08:00
categories: 技术
tags:
    - 算法
    - 排序
---

# 前言
排序，一个看似简单的问题其实存在着很深的学问。都说天下武功，无快不破，其实排序也是。排序的速度基本上决定了一个排序算法的好坏（暂不考虑空间复杂度）。常见的排序算法有插入排序，希尔排序，归并排序和快速排序。现在我们分别用`C语言`和`JavaScript`分别实现，并比较这四种排序算法的优劣。此外，我们还比较了不同语言版本（`C语言`和`JavaScript`）的同一种排序算法排序快慢。

# 插入排序
## 算法描述
插入排序这4中排序里面最慢的，但同时也是最简单的排序算法。它理解起来其实很简单，对于N个元素的排序，插入排序由`N - 1`趟组成。对于这P = 1到 P= N - 1趟中的每一趟P而言，插入排序保证从位置0到位置P上的元素均为已排序状态。在第P趟的时候，算法将位置P上的元素向左移动到它在前P+1个元素中的正确位置。
## 算法时间复杂度
因为算法需要进行的循环次数为$1 + 2 + 3 + ... + N = (N + 1) * N / 2$，所以算法的时间复杂度为$O(N^2)$。
<!-- more -->
# C语言实现
``` c
int * insert_sort(int* data, int num){
	int i,j, temp;
	for(i = 1; i < num; i ++){
		temp = data[i];
		for(j = i; j > 0 && data[j - 1] > temp; j --){
			data[j] = data[j - 1];
		}
		data[j] = temp; //将第i趟的第i个元素放到前i+1个元素的合适位置
	}
	return data;
}
```
<!-- more -->
# JavaScript实现
``` javascript
function insert_sort(data){
    var temp;
    for(var i = 1 ; i < data.length; i ++) {
        temp = data[i];
        for(var j = i; j > 0; j --) {
            if(temp < data[j]) {
                data[j - 1] = data[j]
            }
        }
        data[j] = temp; //将第i趟的第i个元素放到前i+1个元素的合适位置
    }
}
```

# 性能横向比较

* 操作系统：Windows 8 企业版 64位
* 软件平台：Visual Studio 10 和 Nodejs v4.2.1
* 测试数据：100000条随机生成的数据

## C语言版本的测试代码

``` c C语言版本测试代码
/**
*C语言版本的插入排序测试
*/
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
int * insert_sort(int* data, int num);
int main(){
    FILE * file;
    int data_size = 100000;// 测试数据的大小
    int *data = (int*) malloc(sizeof(int) * data_size),temp, i;
    time_t start, end;
    file = fopen("src.txt", "r");
    if(file) {
        for(i = 0; i < data_size && !feof(file); i ++) {
            fscanf(file, "%d", &temp);
            data[i] = temp;
        }// 读取文件中的数据到内存
        start = clock();
        insert_sort(data, data_size);
        end = clock();
        printf("C语言版insert_sort: %dms", end - start);
    } else {
        printf("open file src.txt error!");
    }
    getchar();
    return 0;
}

int * insert_sort(int* data, int num){
	int i,j, temp;
	for(i = 0; i < num; i ++){
		temp = data[i];
		for(j = i; j > 0 && data[j - 1] > temp; j --){
			data[j] = data[j - 1];
		}
		data[j] = temp;
	}
	return data;
}

```

## JavaSript版本的测试代码
``` javascript JavaSript版本的测试代码
var fs = require("fs");
var result;
function insert_sort(data){
    var temp;
    for(var i = 0 ; i < data.length; i ++) {
        temp = data[i];
        for(var j = i; j > 0; j --) {
            if(temp < data[j]) {
                data[j - 1] = data[j]
            }
        }
        data[j] = temp; //将第i趟的第i个元素放到前i+1个元素的合适位置
    }
}

fs.readFile("src.txt", function(err, result){
   data =  result.toString("ascii").split("\n").map(function(value){
        return parseInt(value)
    });
    console.time("javascript版本insert_sort");
    insert_sort(data);
    console.timeEnd("javascript版本insert_sort")
})

```
## 测试结果

![C语言插入排序](/images/blog/20160214/1.png)
![JavaScript语言插入排序](/images/blog/20160214/2.png)
从图中我们可以看到，`C语言`版本的插入排序和`NodeJS`版本的插入排序在性能上基本相当，看到`Nodejs`平台借用V8引擎，叼叼的啊!
