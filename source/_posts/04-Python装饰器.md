---
title: Python装饰器
date: 2017-03-24 19:00
tags: [Python]
---
## 1.什么是装饰器
装饰器是可以实现在**不修改原函数**的前提下对函数进行功能扩展的语法糖，符合“**封闭开放**”的开发原则。其原理是使用**闭包**的方法对函数进行“装饰”。
比如现在要对一个函数添加验证权限的功能，且不能修改原函数，那么可以按如下方法实现：
<!-- more -->
```python
# 装饰器原理
def w1(func):
    def inner():
        print("---正在验证权限----")
        func()
    return inner

def f1():
    print("---f1---")

f1 = w1(f1)
f1()
```
执行结果：
```
---正在验证权限----
---f1---
```
将f1=w1(f1)改成@w1并放到需要“装饰”的函数上面就是所谓的装饰器了：
```python
@w1
def f1():
    print("---f1---")
```
## 2.装饰器什么时候进行装饰
```python
def w1(func):
    print("---正在装饰1----")
    def inner():
        print("---正在验证权限1----")
        func()
    return inner

def w2(func):
    print("---正在装饰2----")
    def inner():
        print("---正在验证权限2----")
        func()
    return inner

# 只要python解释器执行到了这个代码,那么就会自动的进行装饰,而不是等到调用的时候才装饰的
@w1
@w2
def f1():
    print("---f1---")

# 在调用f1之前,已经进行装饰了
f1()

# 执行顺序：
# 1.@w2 等价于 f1 = w2(f1)
# 2.@w1 等价于 f1 = w1(f1)
# 3.执行f1()，此时f1 -> w1 inner，即执行w1里的inner函数
# 4.w1 inner func -> w2 inner，即执行w2函数内的inner函数
# 5.执行w2函数内inner函数里的func函数，即执行原f1函数
```
执行结果：
```
---正在装饰2----
---正在装饰1----
---正在验证权限1----
---正在验证权限2----
---f1---
```
## 3.对带参数的函数进行装饰
### 带定长参数的情况：
```python
def func(functionName):
    print("---func---1---")
    def func_in(a, b):  #如果a,b 没有定义,那么会导致16行的调用失败
        print("---func_in---1---")
        functionName(a, b)  #如果没有把a,b当做实参进行传递,那么会导致调用12行的函数失败
        print("---func_in---2---")

    print("---func---2---")
    return func_in

# test = func(test)
@func
def test(a, b):
    print("----test-a=%d,b=%d---"%(a,b))

# test指向func_in
test(11, 22)
```
### 带不定长参数的情况：
```python
def func(functionName):
    print("---func---1---")
    def func_in(*args, **kwargs):   #采用不定长参数的方式满足所有函数需要参数以及不需要参数的情况
        print("---func_in---1---")
        functionName(*args, **kwargs)   #这个地方,需要写*以及**,如果不写的话,那么args是元祖,而kwargs是字典
        print("---func_in---2---")

    print("---func---2---")
    return func_in

@func
def test(a, b, c):
    print("----test-a=%d,b=%d,c=%d---"%(a,b,c))

@func
def test2(a, b, c, d):
    print("----test-a=%d,b=%d,c=%d,d=%d---"%(a,b,c,d))

test(11,22,33)
test2(44,55,66,77)
```
## 4.使用装饰器对有返回值的函数进行装饰
```python
def func(functionName):
    print("---func---1---")
    def func_in():
        print("---func_in---1---")
        ret = functionName()  #保存 返回来的haha
        print("---func_in---2---")
        return ret  #把haha返回到17行处的调用

    print("---func---2---")
    return func_in

@func
def test():
    print("----test----")
    return "haha"

ret = test()
print("test return value is %s"%ret)
```
## 5.通用的装饰器
```python
def func(functionName):
    def func_in(*args, **kwargs):
        print("-----记录日志-----")
        ret = functionName(*args, **kwargs)
        return ret

    return func_in

@func
def test():
    print("----test----")
    return "haha"

@func
def test2():
    print("----test2---")

@func
def test3(a):
    print("-----test3--a=%d--"%a)

ret = test()
print("test return value is %s"%ret)

a = test2()
print("test2 return value is %s"%a)

test3(11)
```
## 6.带参数的装饰器
```python
def func_arg(arg):
    def func(functionName):
        def func_in():
            print("---记录日志-arg=%s--"%arg)
            if arg=="heihei":
                functionName()
                functionName()
            else:
                functionName()
        return func_in
    return func

# 1. 先执行func_arg("heihei")函数,这个函数return 的结果是func这个函数的引用
# 2. @func
# 3. 使用@func对test进行装饰
@func_arg("heihei")
def test():
    print("--test--")

# 带有参数的装饰器,能够起到在运行时,有不同的功能
@func_arg("haha")
def test2():
    print("--test2--")

test()
test2()

```