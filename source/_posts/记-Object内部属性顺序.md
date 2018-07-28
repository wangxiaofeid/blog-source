---
title: '记:Object内部属性顺序'
date: 2018-07-28 14:55:56
tags:
---

## 结论
1. 声明变量keys值为一个空列表（List类型）
2. 把每个Number类型的属性，按数值大小升序排序，并依次添加到keys中
3. 把每个String类型的属性，按创建时间升序排序，并依次添加到keys中
4. 把每个Symbol类型的属性，按创建时间升序排序，并依次添加到keys中

通过Object.keys()可验证；

### eg
```
Object.keys({
  5: '5',
  a: 'a',
  1: '1',
  c: 'c',
  3: '3',
  b: 'b'
})
// ["1", "3", "5", "a", "c", "b"]
```
### 上面介绍的排序规则同样适用于下列API：
* Object.entries
* Object.values
* for...in循环
* Object.getOwnPropertyNames
* Reflect.ownKeys
注意：以上API除了Reflect.ownKeys之外，其他API均会将Symbol类型的属性过滤掉。

## Array.sort方法不稳定原因
可参考文档[Array.protopyte.sort](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)

### 原因
* 默认排序顺序是根据字符串Unicode码点
* 也可指定比较函数compareFunction
  1. 如果 compareFunction(a, b) 小于 0 ，那么 a 会被排列到 b 之前
  2. 如果 compareFunction(a, b) 等于 0 ， a 和 b 的相对位置不变
  3. 如果 compareFunction(a, b) 大于 0 ， b 会被排列到 a 之前
  4. compareFunction(a, b) 必须总是对相同的输入返回相同的比较结果，否则排序的结果将是不确定的。
