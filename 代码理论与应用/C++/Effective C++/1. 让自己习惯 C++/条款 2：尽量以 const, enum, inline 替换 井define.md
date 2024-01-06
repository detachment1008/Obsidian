即宁可用编译器，也不要用预处理器

原因：
1. 难以 debug，因为该符号被替换了
2. 无法创建 class 的专属常量

常量都用 const 表示起来

如果是类中的常量，需要用 static
- 且如果不取它们的地址，就可以直接使用而无需定义
- 否则，就需要找个地方定义它了，如：`const int GamePlayer::NumTurns`，且不应该给他赋值

enum 和 `#define` 的行为很相似，不会引起额外的内存分配；直接就可以当作常量来使用(如果 enum 没有起名字的话)

而宏函数，使用 `inline` 的内联函数替代就可以了

# 总结

常量：用 `const` 或 `enum` 替换 `#define`

宏函数：用 `inline` 函数替代 `#define`