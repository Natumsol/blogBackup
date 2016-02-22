title: 常见排序算法的C语言和JavaSript实现之 希尔排序
date: 2016-02-14 21:53:49
categories: 技术
tags:
	- 排序
    - 算法
---
上面我们介绍了在速度上比较慢的`插入排序`，现在我们来介绍一种在速度上秒杀`插入排序`的排序算法——`希尔排序`。
## 希尔排序
### 算法描述
希尔排序也成为增量排序。希尔排序使用一个序列$\\{h_1, h_2, h_3, … h_t\\}$，叫增量序列。只要$h_1 = 1$，任何增量序列都是可以的。在使用增量$h_k$的一趟排序后，对于每一个$i$我们有$A[i] \leq A[i + h_k]$（A为待排序的数组），所有相隔$h_k$的元素都将被排序，此时称其为`hk-排序`的。希尔排序的一个重要性质是：一个$h_k$排序的文件将保持它的`h_k排序性`，也就是说经$h_{k-1}$排序后，任然满足$A[i] \leq A[i + h_k]$。
### 算法时间复杂度
依据所选择的增量序列的不同，算法的时间复杂度也有所不同。对于shell本人推荐的$ht = N / 2， hk = h_{k+1} / 2$增量序列来说，时间复杂度为：$O(N ^ {\frac{3}{2}})$。
<!-- more -->
### C语言实现
<!--more-->
```c 
int * shell_sort(int * data, int num){
	int i, j, increment, temp;
	for(increment = num / 2; increment > 0 ; increment = increment / 2){
		for(i = increment; i < num; i ++){
			temp = data[i];
			for(j = i; j >= increment; j -= increment){
				if(temp < data[j - increment])
					data[j] = data[j - increment];
				else
					break;
			}

			data[j] = temp;
		}  
	}
	return data;
}
```
### JavaScript实现
```javascript
function shell_sort(data) {
    var increment = Math.floor((data.length / 2));
    var temp;
    for (; increment > 0; increment =  Math.floor((increment / 2))) {
        for (var i = increment; i < data.length; i ++) {
            temp = data[i];
            for(var j = i; j >= increment; j -= increment) {
                if(temp < data[j - increment]) {
                    data[j] = data[j - increment];
                } else {
                    break;
                }
            }
            data[j] = temp;
        }
    }
}
```
### 性能横向测试
测试平台和代码上篇已经介绍过了，这里就不赘述了，直接给出测试结果。

![C语言希尔排序](/images/blog/20160214/3.png)
![JavaScript语言希尔排序](/images/blog/20160214/4.png)