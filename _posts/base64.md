title: Base64的原理、实现及应用
date: 2015-11-17 11:00:22
categories: 技术
tags:
	- Base64
---
## 简介
`Base64`是一种基于64个可打印字符来表示二进制数据的表示方法。由于2的6次方等于64，所以每6个比特为一个单元，对应某个可打印字符。三个字节有24个比特，对应于4个`Base64`单元，即3个字节需要用4个可打印字符来表示。它可用来作为电子邮件的传输编码。在`Base64`中的可打印字符包括字母`A-Z`、`a-z`、数字`0-9`，这样共有62个字符，此外的两个可打印符号在不同的系统中而不同，一般为`+`和`/`。

## 转换原理
Base64的直接数据源是二进制序列(Binary Sequence)。当然，你也可以将图片、文本和音视频转换成二进制序列，再然后转换为Base64编码。我们这里讨论的是如何将二进制转换为Base64编码，对于如何将图片，文本和音视频转换为二进制序列敬请期待。

在转换前，先定义一张索引表，这张表规定了如何转换。
![索引](/images/blog/20151118/index.png)
转换的时候我们先将二进制序列分组，每6个比特为一组。但是如果编码的字节数不能被3整除，那么最后就会多出1个或两个字节，可以使用下面的方法进行处理：先使用0字节值在末尾补足，使其能够被3整除，然后再进行base64的编码。在编码后的base64文本后加上一个或两个'='号，代表补足的字节数。也就是说，当最后剩余一个八位字节（一个byte）时，最后一个6位的base64字节块有四位是0值，最后附加上两个等号；如果最后剩余两个八位字节（2个byte）时，最后一个6位的base字节块有两位是0值，最后附加一个等号。 参考下表：
![索引](/images/blog/20151118/2.png)
<!-- more -->
## 用JavaScript实现Base64

原理明白了以后，实现起来就很容易了。
```javascript

define(function(require, exports, module) {

    var code = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/".split(""); //索引表

    /**
     * @author natumsol@gmail.com
     * @description 将二进制序列转换为Base64编码
     * @param  {String}
     * @return {String}
     */
    function binToBase64(bitString) {
        var result = "";
        var tail = bitString.length % 6;
        var bitStringTemp1 = bitString.substr(0, bitString.length - tail);
        var bitStringTemp2 = bitString.substr(bitString.length - tail, tail);
        for (var i = 0; i < bitStringTemp1.length; i += 6) {
            var index = parseInt(bitStringTemp1.substr(i, 6), 2);
            result += code[index];
        }
        bitStringTemp2 += new Array(7 - tail).join("0");
        if (tail) {
            result += code[parseInt(bitStringTemp2, 2)];
            result += new Array((6 - tail) / 2 + 1).join("=");
        }
        return result;
    }

    /**
     * @author natumsol@gmail.com
     * @description 将base64编码转换为二进制序列
     * @param  {String}
     * @return {String}
     */
    function base64ToBin(str) {
        var bitString = "";
        var tail = 0;
        for (var i = 0; i < str.length; i++) {
            if (str[i] != "=") {
                var decode = code.indexOf(str[i]).toString(2);
                bitString += (new Array(7 - decode.length)).join("0") + decode;
            } else {
                tail++;
            }
        }
        return bitString.substr(0, bitString.length - tail * 2);
    }

    /**
     * @description 将字符转换为二进制序列
     * @param  {String} str 
     * @return {String}    
     */
    function stringToBin(str) {
        var result = "";
        for (var i = 0; i < str.length; i++) {
            var charCode = str.charCodeAt(i).toString(2);
            result += (new Array(9 - charCode.length).join("0") + charCode);
        }
        return result;
    }
    /**
     * @description 将二进制序列转换为字符串
     * @param {String} Bin 
     */
    function BinToStr(Bin) {
        var result = "";
        for (var i = 0; i < Bin.length; i += 8) {
            result += String.fromCharCode(parseInt(Bin.substr(i, 8), 2));
        }
        return result;
    }
    exports.base64 = function(str) {
        return binToBase64(stringToBin(str));
    }
    exports.decodeBase64 = function(str) {
        return BinToStr(base64ToBin(str));
    }
})
```
## 将图片数据进行Base64编码
将图片数据转换为Base64，首先要获取到图片的二进制数据。图片的二进制数据可以通过`canvas`接口得到。具体实现为：
```javascript
function getCanvas(w, h) {
		var c = document.createElement('canvas');
		c.width = w;
		c.height = h;
		return c;
	}

function getPixels(img) {
	var c = getCanvas(img.width, img.height);
	var ctx = c.getContext('2d');
	ctx.drawImage(img, 0, 0);
	return ctx.getImageData(0, 0, c.width, c.height);
}
```
取到图片的二进制数据后，接下来就要进行编码了。因为图片不仅包含像素信息，还包含长度，宽度信息。所以在编码像素信息的同时也应将宽度和高度信息按某一约定进行编码，我是这样处理的：

1. 将图片的像素数值数据转换为二进制序列；
2. 将宽度和高度信息组合成字符串`$$width,height$$`，转换为二进制序列；
3. 将图片像素信息的二进制序列和图片宽高度的二进制序列组合起来，然后再进行Base64的编码

具体实现为：
```javascript
function img2Base64(img) {
	var imgData = getPixels(img).data;
	var imgWidth = getPixels(img).width;
	var imgHeight = getPixels(img).height;
	var bin = "";
	for (var i = 0; i < imgData.length; i++) {
		bin += base.numToString(imgData[i]);
	}
	bin = bin + base.stringToBin("$$" + imgWidth + "," + imgHeight + "$$");
	return base.binToBase64(bin);
}
```



## 将图片Base64数据进行解码
解码是编码的逆过程。过程大致为：

1. 将图片的Base64信息进行解码，得到包含图片像素信息和宽高度信息的二进制序列；
2. 然后将这个二进制序列解码成字符串，获取高度和宽度信息；
3. 去除二进制序列中的高度和宽度信息，得到像素信息；
4. 根据像素信息生成像素矩阵；
5. 根据像素矩阵、宽度和高度创建图片对象`ImageData`；
6. 利用`putImageData`将图像绘制出来。

具体的代码实现为：
```javascript
function paint(imgData) {
		var canvas = document.getElementById("myCanvas");
		var ctx = canvas.getContext("2d");
		ctx.fillRect(0, 0, imgData.width, imgData.height);
		ctx.putImageData(imgData, 0, 0);
	}

function base642img(data) {
	var str = base.BinToStr(base.base64ToBin(data));
	var imgWidth = str.match(/\$\$(\d+),(\d+)\$\$$/, "")[1];
	var imgHeight = str.match(/\$\$(\d+),(\d+)\$\$$/, "")[2]
	var imgData = base.base64ToBin(data).replace(base.stringToBin("$$" + imgWidth + "," + imgHeight + "$$"), "");

	var ImageDataArray = new Uint8ClampedArray(imgWidth * imgHeight * 4);
	for (var i = 0; i < ImageDataArray.length; i++) {
		ImageDataArray[i] = parseInt(imgData.substr(i * 8, 8), 2);
	}
	return new ImageData(ImageDataArray, imgWidth, imgHeight);

}

```

## DEMO演示
在线演示：[ DEMO ](/project/base64)
源码请移步[ Github ](https://github.com/Natumsol/base64)
![截图](/images/blog/20151118/1.png)