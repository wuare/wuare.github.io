---
title: 编译原理-正则表达式转换为DFA
date: 2020-07-12 21:19:05
tags: [编译原理]
---

转换关系
<!--more-->
r=ab对应的DFA
![r=ab](../../../../img/re-dfa-and.jpg)
r=a|b对应的DFA
![r=a|b](../../../../img/re-dfa-or.jpg)
r=(a)\*
![r=a\*](../../../../img/re-dfa-some.jpg)
eg: r=(a|b)\*abb对应的NFA
![r=a\*](../../../../img/re-dfa-convert.jpg)
