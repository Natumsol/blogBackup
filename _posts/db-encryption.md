title: 基于NodeJS和Mysql的数据库加密系统的设计与实现
date: 2015-11-07 15:30:21
categories: 技术
tags:
    - NodeJs
    - Mysql
    - 数据库加密
---
# 起因

这纯粹是一场意外。

研一选课的时候，按照培养计划上的说明，毕业之前，每人必须选修一门跨一级学科（什么鬼！！）的课程，然后我和实验室的其他三人一看就乐呵了，心想可以去文科班蹭课了（此处四人中某人发出了猥琐的笑声），然后就选了一门哲学课——《中国哲学之儒家哲学》。。等到考试成绩粗来的时候一看傻眼了——0分！跑到教务处一翻撕逼，原来我们上错课了（上成了《道家哲学》！！！）。

最后费了好大劲把这件事解决之后，今年学弟们参照我们已选的课程时发现，我们上的哲学课竟然不是跨一级学科的课程！！知道真相的我眼泪掉下来，于是就补选了这门“坑爹”的课程——计算机学院的《数据库安全》。

作为一只平时上课都不认真听的电信狗，被计算机学院学弟们碾压了，好有激情有木有！每次上课我们只有坐最后一排的份，每次老师提问学弟们都抢着答有木有！！因为一些私事，我逃掉了很多课（捂脸，逃，），眼看做Presentation的deadline马上就到了，而我题目还没选，于是硬着头皮选了一个坑爹的题目——数据库加密。

# 设计与实现
## 需求分析
1. 不考虑密钥的管理技术 ， 表内所有数据 ( 每一行的每 个字段 ) 采用相同的密钥加密 ， 充分利用 DBMS 的管理能力；
2. 数据库加解密在客户端进行；
3. 要对字符串查询：LIKE 查询 ( 中英文 )进行测试，不要求做成通用程序，可以专门针对这两类查询设计处理程序； 至少应支持的查询模式: s%， %s， %s% 。
4. 全表查询后解密和直接密文查询后再精确查询进行性能对比。

*注意 : 只处理中文字符的加密，要求用多本金庸小说作为测试数据，以 50 个字为单位截断，加密后保存在数据*
<!-- more -->
## 理论基础
理论基础要详细说的话可以写一篇论文了，这里我粗略的介绍一下。

首先，将待加密的字符串明文用一定的算法映射成一个信息矩阵，这个矩阵包含了这个字符串的所有信息，因为矩阵比较大（256*260），所以需要对其进行压缩与降维处理，转换为一个定长的比特串，这个比特串就作为能代表这个字符串的索引。查询的时候，如果待查询的字符串包含关键字，那么将关键字也转换为定长的比特串后，那么待查字符串索引 &（位与） 关键字比特串 = 关键字比特串。
## 技术调研
后端：作为一个前端狗，后台果断`NodeJS` + `Express`。

数据库：数据库的话MongoDB在这里不适合（`MongoDB`为NoSQL，
不支持传统的SQL语句），所以选用比较常用的`Mysql`。

环境：运行环境选用`Ubuntu 14.04 LTS`。
## 实现
### 数据处理
我选用的小说为《天龙八部》，因为我主要是做中文的查询，为了简单起见，处理掉了小说中所有的非中文字符，包括英文字母，数字，标点符号等等。

因为小说比较大，直接用文本编辑器批量查找替换会，文本编辑器会直接崩掉。这时候`awk`和`sed`等命令行工具就大显身手了，处理兆级的文本文件时间仅为毫秒级。
我处理《天龙八部》的命令为：
```shell
sed 's/[，“”？。：》《……！； ’‘ ·（）()、-*.]//g' data.txt | sed '/^$/d'  | tr -t "\n" " " data_1.txt
```
### 数据入库
我采取的方法比较笨，首先用`awk`生成`sql`语句，然后利用DBMS进行入库。不过还是很快的，入库20000多条数据耗时不到1秒。
### 生成字符串矩阵信息
这个是数据库加密的核心。具体做如下图所示：
![](/images/blog/20151107/matrix.png)
对应的代码为：
```javascript
/**
 * @description 返回字符串的信息矩阵
 * @param  {string} str
 * @return {object}
 */
function generateStrMatrix(str, hasHead, hasTail) {
    var matrix = new Array(260);
    var strUnicodeParis = [];
    for (var i = 0; i < 260; i++) {
        matrix[i] = new Array(256);
        for (var j = 0; j < 256; j++) {
            matrix[i][j] = 0;
        }
    }// 初始化信息矩阵

    for (var i = 0; i < str.length; i++) {
        strUnicodeParis.push(convertChineseUnicode([str[i]]));
    }
    for (var i = 0; i < strUnicodeParis.length - 1; i++) {
        var a1 = strUnicodeParis[i][0];
        var a2 = strUnicodeParis[i][1];
        var b1 = strUnicodeParis[i + 1][0];
        var b2 = strUnicodeParis[i + 1][1];

        matrix[a1][a2] = 1;
        matrix[a1][b1] = 1;
        matrix[a1][b2] = 1;
        try {
            matrix[a2][b1] = 1;
        } catch (e) {
            console.log(str);
            throw e;
        }
        matrix[a2][b2] = 1;
        matrix[b1][b2] = 1;

    } // 填入邻接关系
    if (hasHead) {
        matrix[256][strUnicodeParis[0][0]] = 1; // 头字节信息
        matrix[258][strUnicodeParis[0][1]] = 1; // 第二个字节信息
    }

    if (hasTail) {
        matrix[257][strUnicodeParis[strUnicodeParis.length - 1][1]] = 1; // 尾字节信息
        matrix[259][strUnicodeParis[strUnicodeParis.length - 1][0]] = 1; // 倒数第二个字节信息
    }

    return matrix;

}

```
### 确定索引比特串的长度
经查询相关文献发现，当m,k,n满足`M = k * n / ln2`时，误称率就最低。（k为哈希函数的个数，n为矩阵中1的平均个数，m为压缩比特串的长度）

通过对数据库所有数据进行统计分析得到，n收敛与`228`。根据`m = n * k / ln2`，`k = 10`, `n = 228`,计算得到m大约为`3304`，因为m最好为**素数**，经查询素数表得到m为`3307`。
### 生成比特串
根据比特串生成算法生成比特串，相应的代码如下：
```javascript
/**
 * @description 生成长度为m bit的序列
 * @param  {object} matrix 字符串信息矩阵
 * @param  {number} m      生成比特串的长度
 * @param  {number} k      哈希函数的个数
 * @return {string}        返回生成的M序列
 */
function string_transfer(matrix, m, k) {
    var V = new Array(m);
    var a = matrix.length;
    var b = matrix[0].length;
    for (var i = 0; i < V.length; i++) {
        V[i] = 0;//初始化
    }
    for (var i = 0; i < a; i++) {
        for (var j = 0; j < b; j++) {
            if (matrix[i][j] == 1) {
                var flag = new Array(m);
                for (var p = 0; p < flag.length; p++) {
                    flag[p] = false;//防冲突用的标志位初始化
                }
                for (var d = 0; d < k; d++) {//给比特串的特定位置置 1,共置 k 次
                    var index = hash(((i - 1) * b + j - 1) + "", d) % m + 1;
                    while (flag[index] == true) index++;
                    if (index < m) { //防止越界！！！！fuck it
                        V[index] = 1;
                        flag[index] = true;
                    }
                }
            }
        }
    }
    return V;
}

```
### 存储比特串
因为索引的长度为3307位，为了接下来方便进行位于操作，我们将索引补全为3328位，以64位为一组进行分段，将每段作为一个数据库字段，以bigint（8字节，64位）的形式进行存储。

### 将普通SQL语句转换为加密查询语句
对于%s,去掉信息矩阵的头信息（倒数第一个字符和倒数第二个字符），然后再生成比特串`v`， 相应的`sql`语句`select * from novel where text like "%s"` 转换为`select * from novel where v & index = v`， 其中index为索引所在的字段名，真实情况中可能index被分成了很多份，所以真实的`sql`语句可能为`select * from novel where v1 & index1 = v1 and v2 & index2 = v2 and v3 & index3 = v3 & .. `。

对于%s%，首尾信息均要（默认），`sql`语句可参考%s的情况。

对于s%，去掉尾信息（第一个和第二个字符），`sql`语句可参考%s的情况。

# 测试
项目地址为[数据库加密](https://github.com/Natumsol/db_homework.git)，代码里面有详细的注释。

测试结果如下图：
![DEMO](/images/blog/20151107/db_cryption.png)
可以看出，直接密文查询，性能还是很好的嘛～～哈哈。
# 结语
通过这次实验，让我领悟到`deadline是第一生产力`原来是很有道理的嘛~
