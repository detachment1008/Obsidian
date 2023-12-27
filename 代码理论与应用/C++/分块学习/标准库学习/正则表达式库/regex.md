# `regex`
---

在 C++ 11 引入，是一个正则表达式处理相关的库

# 重要类
---

## `basic_regex`

```
template <class CharT, class Traits = std::regex_traits<CharT>>
class basic_regex;
```

提供正则表达式的通用框架

一些常用字符的 `typedef`:
1. `regex`: `basic_regex<char>`
2. `wregex`: `basic_regex<wchar_t>`

可以用构造函数，`=` 或 `assign` 为其赋值

相当于这个就是创建了一个正则表达式的匹配模式

## `match_results`

保存了正则表达式匹配结果

常用特化：
1. `cmatch`: `match_results<const char*>`
2. `smatch`: `match_results<std::string::const_iterator>`

# 重要函数
---

## `regex_search`

确定正则表达式和字符序列中是否有匹配

1. 判断是否存在: `std::regex_search(std::string, std::regex)`
2. 判断是否存在，并存储匹配结果: `std::regex_search(std::string, std::smatch, std::regex)`

`match` 中保存了匹配项的末尾，利用这个重新设置匹配目标字符串，可以实现重复搜索