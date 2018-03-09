---
title: solidity基因解析算法
date: 2018-03-09 18:29:58
tags: [blockchain,dev]
---
#solidity基因解析算法

solidity储存是比较昂贵的，所以需要对数据做压缩，因为最初的设计问题，单位的基因被设定为了uint256，也就是一个256位的int，相当于32个byte。

而项目的需求是，需要把这32个byte解析为超过50个属性，经过和策划的讨论，最终定了具体方案。无非就是用某个byte表示某个属性，或者某些bit表示某些属性，大部分都很容易通过bitwise位操作来实现。但是有一个部分逻辑实现起来比较复杂，整理成类似ACM的需求：
```
输入 Mbyte integer, Number N, Length L
M*8<L<M*16
N<50
L<N
输出 array[L] 内容为0~N的随机数，并且不重复
```

春节前写了一个算法大概实现了，但是随机数不均匀，而且感觉实现很烂。昨天灵机一动换了一个思路，其实完全可以把这个问题转换为一个洗牌问题。
相当于N张牌，用一个随机数种子seed洗牌，取出前L张。
这样问题就简化了，只需要实现核心的根据根据seed洗牌函数即可，可以使用 [Fisher–Yates shuffle](https://gaohaoyang.github.io/2016/10/16/shuffle-algorithm)算法。用MD5可以在于如何保证M个byte生成N个数字。

Javascript代码如下
```Javascript
function seedArray(seedHex,length){
  let hexLen = seedHex.length / 2;
  let array = new Array();
  for(let i=0;i<length && i <hexLen;i++){
    array.push(getByteFromHexWithOut0x(seedHex,i));
  }
  let left = length - array.length;
  if(left == 0) return array;
  let md5Count = (left + 1)/16;//要生成left个MD5
  let str = "";
  let last = "" + seedHex;
  for(let i=0;i<md5Count;i++){
    let t = md5(last);
    str += t;
    last = t;
  }
  for(let i=0;i<left;i++){
    array.push(getByteFromHexWithOut0x(str,i));
  }
  return array;
}

function shuffle(array,seed){
  let _seedArray = seedArray(seed,array.length);
  for(let i=0;i<array.length;i++){
     let randomIndex = _seedArray[i] % array.length;
     let itemAtIndex = array[randomIndex];
     array[randomIndex] = array[i];
     array[i] = itemAtIndex;
  }
}
```
