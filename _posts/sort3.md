title: 常见排序算法的C语言和JavaSript实现之 归并排序
date: 2016-02-18 21:08:08
categories: 技术
tags:
    - 算法 
    - 排序
---
## 归并排序
### 算法描述
现在我们来介绍一种很巧妙的排序算法——归并排序。为什么说它巧妙呢，归并排序利用递归来实现分治，从而极大的简化了代码量，排序算法清晰明了，而且速度也是相当的快。归并排序的基本操作是合并两个已排序的表，因为这两个表是已排序的，所以若将输出放到第三个表中，该算法可以通过对输出数据一趟排序来完成。

### 算法的时间复杂度
归并的时间复杂度是前三种排序算法里面最好的，为$O(NlogN)$
<!-- more -->
### C语言实现
```c 
void merge_sort(int * data, int num){
    int * dataTemp = (int *) malloc(sizeof(int) * num);
    if(dataTemp != NULL) {
        m_sort(data, dataTemp, 0, num - 1);
    } else {
        printf("No Space For dataTemp!");
    }
}

void m_sort(int * data, int * dataTemp, int left, int right) {
    int center;
    if(left < right) {
        center = (left + right) / 2;
        m_sort(data, dataTemp, left, center);
        m_sort(data, dataTemp, center + 1, right);
        merge(data, dataTemp, left, center + 1, right);
    }
}

void merge(int * data, int *dataTemp, int leftPos, int rightPos, int rightEnd){
    int leftEnd = rightPos - 1,
        tempPos = leftPos, 
        i,
        total = (rightEnd - leftPos + 1);
        
    while(leftPos <= leftEnd && rightPos <= rightEnd) {
        if(data[leftPos] < data[rightPos]) {
            dataTemp[tempPos ++] = data[leftPos ++];
        } else {
            dataTemp[tempPos ++] = data[rightPos ++];
        }
    }
    
    while(leftPos <= leftEnd) {
        dataTemp[tempPos ++] = data[leftPos ++];
    }
    
    while(rightPos <= rightEnd) {
         dataTemp[tempPos ++] = data[rightPos ++];
    }
    
    for(i = 0; i < total; i ++, rightEnd --) {
        data[rightEnd] = dataTemp[rightEnd];
    }
}
```
### JavaScript实现
``` javascript
function merge_sort(data) {
    var dataTemp = new Array(data.length);
    m_sort(data, dataTemp, 0, data.length - 1);
}

function m_sort(data, dataTemp, left, right) {
    if (left < right) {
        var center = Math.floor((left + right) / 2);
        m_sort(data, dataTemp, left, center);
        m_sort(data, dataTemp, center + 1, right);
        merge(data, dataTemp, left, center + 1, right);
    }
}

function merge(data, dataTemp, leftPos, rightPos, rightEnd) {
    var leftEnd = rightPos - 1,
        tempPos = leftPos,
        total = (rightEnd - leftPos + 1);
        
     while(leftPos <= leftEnd && rightPos <= rightEnd) {
         if(data[leftPos] < data[rightPos]) {
             dataTemp[tempPos ++] = data[leftPos ++];
         } else {
              dataTemp[tempPos ++] = data[rightPos ++];
         }
     }
     
     while(leftPos <= leftEnd){
          dataTemp[tempPos ++] = data[leftPos ++];
     }

     while(rightPos <= rightEnd) {
         dataTemp[tempPos ++] = data[rightPos ++];
     }
     
     for(var i = 0; i < total; i ++, rightEnd --) {
         data[rightEnd] = dataTemp[rightEnd];
     }
}
```

### 性能横向测试
* 操作系统：Windows 8 企业版 64位
* 软件平台：Visual Studio 10 和 Nodejs v4.2.1
* 测试数据：相同的`十万条`随机生成的数据

![C语言归并排序](/images/blog/20160214/5.png)
![JavaScript语言归并排序](/images/blog/20160214/6.png)

