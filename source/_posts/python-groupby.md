---
title: python_groupby
date: 2019-07-31 11:07:31
tags: [python, groupby]
---

## itertools
　　itertools提供groupby()函数，可以将输入按键值分成组，并返回一个迭代器，利用这个函数可以实现类似于SQL查询中的 *groupby* 功能。  
　　[参考](https://docs.python.org/zh-cn/3/library/itertools.html)  
<!--more-->

## groupby()
　　使用groupby()函数前，需要先对输入进行排序，因为为了效率，groupby()只是简单的将相邻的具有相同键值的输入分到同一个组。  

```python
from itertools import groupby
res_audit = [{"user":"a", "date":"2019-01-12", "mem":1024, "cpu":32},
       {"user":"a", "date":"2019-01-12", "mem":1024, "cpu":32},
       {"user":"b", "date":"2019-01-13", "mem":1024, "cpu":32},
       {"user":"b", "date":"2019-01-12", "mem":1024, "cpu":32},
       {"user":"c", "date":"2019-01-12", "mem":1024, "cpu":32}]

res_audit = sorted(res_audit, key = lambda x:(x['user'], x['date']))
res_audit = groupby(res_audit, key = lambda x:(x['user'], x['date']))

for k,v in res_audit:
    print k
    tmp = None
    for item in v:
        if tmp is None:
            tmp = item
        else:
            tmp['mem'] += item['mem']
            tmp['cpu'] += item['cpu']
    print tmp
```

　　输出是：
```
('a', '2019-01-12')
{'date': '2019-01-12', 'mem': 2048, 'user': 'a', 'cpu': 64}
('b', '2019-01-12')
{'date': '2019-01-12', 'mem': 1024, 'user': 'b', 'cpu': 32}
('b', '2019-01-13')
{'date': '2019-01-13', 'mem': 1024, 'user': 'b', 'cpu': 32}
('c', '2019-01-12')
{'date': '2019-01-12', 'mem': 1024, 'user': 'c', 'cpu': 32}
```
