---
title: c++_priority_queue_lambda
date: 2019-04-26 14:17:23
tags:  c++
---

### priority_queue自定义比较函数
在使用sort()函数时，可以如下：
```
sort(input.begin(), input.end(), [](int a, int b){return a<b;});
```
但是这样使用却不行：
```
priority_queue<int, vector<int>, [](int a, int b){return a<b;}> queue;
```
<!--more-->
原因在于使用模板生成类时，需要指定参数类型，但是lambda表达式是一个对象，因此上面相当于传了一个对象给一个类型。
但是模板生成函数时，可以根据传递给函数的参数推断出参数的类型，因而可行。

### 使用方式
```
auto myCmp = [](int a, int b){return a<b;};
priority_queue<int, vector<int>, decltype(myCmp)> queue(myCmp);
```

### list的sort()
由于list是链表，因而不能直接利用sort()库函数，但是list提供了自己的sort()
```
myList.sort([](int a, int b){return a<b;});
```

### [参考资料](https://stackoverflow.com/questions/5807735/c-priority-queue-with-lambda-comparator-error)
