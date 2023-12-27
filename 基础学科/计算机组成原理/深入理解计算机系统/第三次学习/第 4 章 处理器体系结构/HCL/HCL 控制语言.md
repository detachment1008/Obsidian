特点：
1. 没有短路求值，都会求值
2. 用 C 的逻辑符号表示位级的逻辑运算
3. 情况表达式：
```
Word Out = [
	select1: expr1;
    select2: expr2;
	...
];
```
4. 集合表示法：`iexpr in {iexpr1, iexpr2, ...};`