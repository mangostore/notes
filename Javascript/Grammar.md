# 记录一些没有引起注意的语法细节

## 实现长度为10内容分别为0-9的数组

```js
Array(10).fill().map((item, index) => index);
Array.from({ length: 10 }).map((item, index) => index);
```
说明：Array(10)生成长度为10的空数组[empty × 10]，map循环的时候会跳过空项；而Array.from({ length: 10 })是10个undefined组成的数组。
