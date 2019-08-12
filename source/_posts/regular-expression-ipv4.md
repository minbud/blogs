---
title: regular_expression_ipv4
date: 2019-08-10 14:16:32
tags: [regExp, ipv4]
---
## 正则表达式匹配ipv4地址
　　最近面试的时候被问到了这个问题，正则表达式接触得不多，只看得懂简单的，没写得出来，后来查了一下：[参考](https://stackoverflow.com/questions/5284147/validating-ipv4-addresses-with-regexp)  

<!--more-->

```shell
grep -E '\b((25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)(\.|$)){4}\b' <<< 192.168.1.1
```
　　不过这个表达式会匹配‘192.168.1.1.1’，但是这个应该也是正常的。  

## grep -E '\b\b' word boundary
　　[参考](https://unix.stackexchange.com/questions/71537/confused-about-word-boundary)  
　　上面的参考链接已经说得比较清楚，我大概翻译一下意思。  
　　'grep -E'就是使用扩展的正则表达式。  
　　‘\bfoo\b’则表示匹配的行中的 *foo* 的前一个邻接字符和后一个邻接字符不能是[a-zA-Z_],也就是不能是大小写字母和下划线，所以 *word boundary* 也就是字符的前后边界，指的是。比如 ‘afoo’、'fooa'、'foo_'这种都是不匹配的，但是'foo'、'foo*'、'%foo'这种就是匹配的。试一下很容易就能明白了。  
