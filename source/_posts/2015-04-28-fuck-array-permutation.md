title: "数组全排列"
date: 2015-04-28 18:04:15
tags:
s: fuck array permutation
---

如何输出一个数组全部可能的排列组合？

比如`[1, 2]`，应该输出`[1, 2]`、`[2, 1]`。

再比如`[1, 2, 3]`，应该输出`[1, 2, 3]`、`[1, 3, 2]`、`[2, 1, 3]`、`[2, 3, 1]`、`[3, 1, 2]`、`[3, 2, 1]`。

观察以上数据，其实可以推断出，假设一组数`p = {r1, r2, r3, ..., rn}`，全排列表示为perm(p)，数组中除rn之外的数据集合表示为pn = p - {rn}。

那么perm(p) = r1perm(p1), r2perm(p2), r3perm(p3), ..., rnperm(pn)。

数据全部可能的排列组合就是r1perm(p1)，r2perm(p2)，....，rnperm(pn)的集合。

<!-- more -->

算法：

```text
// 对从k到m的子数组进行排列
perm(arr, k, m):
    if k > m
        // 递归结束，输出一个排列
        console.log(arr);
    else
        for i from k to m
            // 以arr[k]为开头的排列
            // 先将i处的数交换到k处
            // 那么剩下的数就为arr - {arr[k]}
            swap(arr, k, i)
            // 排列的问题就转换成arr[k]perm(arr, k + 1, m)
            perm(arr, k + 1, m)
            // 将原来的开头的数交换回去
            // 继续以数组下一项为开头的排列
            swap(arr, k, i)
```

js实现如下：

```javascript
define(
    function (require) {
        function swap(arr, i, j) {
            var temp = arr[i];
            arr[i] = arr[j];
            arr[j] = temp;
            return arr;
        }

        function perm(arr, start, end, result) {
            if (start > end) {
                result.push(arr.slice(0));
            }
            else {
                for (var i = start; i <= end; i++) {
                    swap(arr, start, i);
                    perm(arr, start + 1, end, result);
                    swap(arr, start, i);
                }
            }
            return result;
        }

        return function (arr) {
            var result = [];
            perm(arr, 0, arr.length - 1, result);
            return result;
        };
    }
);

```

使用方式：

```javascript
require(['module-id'], function (perm) {
    var result = perm([1, 2, 3]);
    console.log(JSON.stringify(result));
});
```

输出`[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,2,1],[3,1,2]`。


