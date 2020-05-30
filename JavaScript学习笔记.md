JavaScript的语法和Java语言类似。
网景公司为静态HTML上添加动态效果，设计出JavaScript语言，并借Java名气取名为JavaScript。
ECMAScript是一种语言标准，而JavaScript是网景公司对ECMAScript标准的一种实现。
最新版ECMAScript 6标准，简称ES6，在2015年6月正式发布。


嵌入式JavaScript：
```js
<head>
  <script>
  </script>
</head>
```
引入式JavaScript：
```
<head>
  <script src="/static/js/xxx.js"></script>
</head>
```


|Number|解释|
|----|----|
|NaN;|NaN表示Not a Number，当无法计算结果时用NaN表示 0/0|
|Infinity;|Infinity表示无限大，当数值超过了JavaScript的Number所能表示的最大值时，就表示为Infinity 2/0|

|相等运算符|解释|
|----|----|
|==|自动转换数据类型再比较，不推荐使用|
|===|不会自动转换数据类型，如果数据类型不一致返回false，如果一致再比较，推荐使用|
|NaN|NaN这个特殊的Number与所有其他值都不相等，包括它自己|


