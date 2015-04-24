---
layout: post
title: "将NodeList转成Array"
tags:
    - NodeList
    - Array
    - slice
description: ""
---

`NodeList`是一个Dom概念，其定义如下：

    interface NodeList {
        Node               item(in unsigned long index);
        readonly attribute unsigned long    length;
    };

它有一个方法和一个属性，可以通过`item`方法来获得指定的元素：

```javascript
nodeList.item(1);
```

也可以像数组一样，用下标来访问：

```javascript
nodeList[1];
```

但它不是数组，其原型链如下：

    myNodeList ——> NodeList.prototype ——> Object.prototype ——> null

而数组的原型链则为：

    myArray ——> Array.prototype ——> Object.prototype ——> null

`NodeList`没有数组的那些方法，对它进行遍历不是很方便，我们得先把它转成数组。

借助`call`以及`Array.prototype.slice`可以做到：

```javascript
var myArray = Array.prototype.slice.call(myNodeList);
```

`call`是把`Array.prototype`中的`this`指向myNodeList，`slice`方法要求this所指的对象有一个length参数，而刚好`NodeList`就有。

`Array.prototype.slice`具体如何工作的，参见[ecma](http://www.ecma-international.org/ecma-262/5.1/#sec-15.4.4.10)。
