title: 常见排序算法的C语言和JavaSript实现之 快速排序
date: 2016-02-20 12:34:41
categories: 技术
tags:
	- 排序
    - 算法
---
## 快速排序
### 算法描述
快速排序是在事件中已知的最快排序算法，该算法之所以这么快是因为其内部高度优化的内部循环。它的最坏情形耗时$O(N^2)$，但是稍加修正就可以避免这种情形。和归并排序类似，快速排序也是一种分治的递归排序。对于待排数组`S`，快速排序可以简单的秒速为：

1. 如果`S`中的元素个数问哦0或1，则返回。
2. 取`S`中任意一元素`v`，称之为枢纽元（pivot）。
3. 将$S-\\{v\\}$(S中的其余元素)分成两个不相交的集合：$S_1= \\{x \in S - \\{v\\} | x \leq v\\}$ 和 $S_2= \\{x \in S - \\{v\\} | x \geq v\\}$。
4. 返回`quick_sort(S1)`后，继随`v`，继而`quick_sort(S2)`。

由于对枢纽元的处理的不同，第三步分割的描述不是唯一的，因而这就成了设计上的一个决策。
<!-- more -->
### 算法的时间复杂度
算法平均时间复杂度为：$O(NlogN)$
## C语言实现
```c C语言版的快速排序算法
void swap(int* p, int* q) {
    int temp;
    temp = *q;
    *q = *p;
    *p = temp;
}

int media3(int *data, int left, int right){
    int center = (left + right) / 2;
    
    if(data[left] > data[center]) {
        swap(&data[left], &data[center]);
    } 
    if(data[left] > data[right]) {
        swap(&data[left], &data[right]);
    }
    if(data[center] > data[right]) {
        swap(&data[center], &data[right]);
    } // 将data[left], data[center], data[right]从小到大排好序
    
    swap(&data[center], &data[right - 1]); // 将pivot放到倒数第二个位置
    
    return data[right - 1];
    
}

void quick_sort(int* data, int left, int right){
    int pivot, i = left, j = right - 1;
    if(left + CUTOFF <= right) { // 如果所剩元素小于三个， 那么这不能利用快速排序了，因为Pivot的选取至少需要三个元素
        pivot = media3(data, left, right);
        while(1){
            while(data[++ i] < pivot) {}
            while(data[-- j] > pivot) {}
            if(i < j) {
                swap(&data[i], &data[j]);
            } else {
                break;
            }
        }
        swap(&data[i], &data[right - 1]);
        quick_sort(data, left, i - 1);
        quick_sort(data, i + 1, right);
    } else {
        insert_sort(data + left, right - left + 1 );// 对于小雨3个的元素，直接利用插入排序来进行排序
    }
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

## JavaScript实现
```javascript JavaScript版快速排序实现
function swap(data, pos1, pos2){
    var temp = data[pos1];
    data[pos1] = data[pos2];
    data[pos2] = temp;
}

function media3(data, left, right) {
    var center = Math.floor((left + right) / 2);
    if(data[left] > data[center]) {
        swap(data, left, center);
    }
    if(data[left] > data[right]) {
        swap(data, left, right);
    }
    if(data[center] > data[right]) {
       swap(data, center, right);
    }   
    swap(data, center, right - 1);
    return data[right - 1];
}

function quick_sort(data, left, right){
    if(left + 3 <= right) {
        var center = Math.floor((left + right) / 2);
        var pivot = media3(data, left, right), i = left, j = right - 1;
        while(1){
            while(data[++ i] < pivot) {}
            while(data[-- j] > pivot) {}
            if(i < j) {
                swap(data, i , j);
            } else {
                break;
            }
        }
        swap(data, center, right - 1);
        quick_sort(data, left, i - 1);
        quick_sort(data, i + 1, right);
    } else {
        insert_sort(data, left, right);
    }
}

function insert_sort(data, left, right){
    if(left < right){
        for(var i = 1; i < right - left + 1; i ++) {
            var temp = data[i];
            for(var j = i; j > 0 && data[j - 1] > temp; j --) {
                    data[j] = data[j - 1];
            }
            data[j] = temp;
        }
    }
    return data;
} 

```

## 横向性能测试
* 操作系统：Windows 8 企业版 64位
* 软件平台：Visual Studio 10 和 Nodejs v4.2.1
* 测试数据：相同的`一千万条`随机生成的数据
测试结果：

![C语言归并排序](/images/blog/20160214/7.png)
![JavaScript语言归并排序](/images/blog/20160214/8.png)
