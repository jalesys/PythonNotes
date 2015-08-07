# PythonNotes
My Python Notes from real world projects



##1. lambda表达式
Python 的lambda表达式不是正在意义上的lambda表达式，Python要求lambda表达式不超过一行。在很多时候使用lambda表达式非常简洁易读。
   
```Python
lambda p : p * 2`
```
 

这相当于一个函数    
```Python
def func(p):        
    return p*2
```

lambda表达式甚至可以赋值给别的变量,有一个非常有意思的例子：
```Python
    processFunc = collapse and (lambda s: " ".join(s.split())) or (lambda s: s)
```
这段代码首先判断变量collapse的值，如果collapse为真(实际上and后面也要判断lambda表达式是否为True，由于lambda表达式就是为真)，则processFunc则为第一个lambda函数，否则为第二个。

lambda表达式实际上是一种更加高阶的抽象。在Python很多时候使用lambda表达式是一种非常好的习惯。

Python中list的排序使用sort函数，sort函数原型如下：
```Python
sorted(data, cmp=None, key=None, reverse=False)
```
> 其中，data是待排序数据，可以使List或者iterator, cmp和key都是函数，这两个函数作用与data的元素上产生一个结果，sorted方法根据这个结果来排序。
cmp(e1, e2) 是带两个参数的比较函数, 返回值: 负数: e1 < e2, 0: e1 == e2, 正数: e1 > e2. 默认为 None, 即用内建的比较函数.
key 是带一个参数的函数, 用来为每个元素提取比较值. 默认为 None, 即直接比较每个元素.
通常, key 和 reverse 比 cmp 快很多, 因为对每个元素它们只处理一次; 而 cmp 会处理多次. 

其中，cmp和key这两个参数函数经常可以使用lambda函数来代替：
```Python
l = [1, 2, 3, 4, 5]

#如果我们要根据l中元素对10取模排序
l = sorted(l, key=lambda x: x % 10)
#也可以这样写
l = sorted(l, cmp=lambda a,b: a%10 < b%10)
```

##2. bisect

假如一个list已经排序，但是下一次插入操作还希望保持顺序，可以这么做:

```Python
l = sorted(l)  
l.append(item)
l = sorted(l)
```
这样每次append都需要重新sort一次，于是bisect就派上用场了，bisect能使**已排序**list在插入后保持原有顺序。
```Python
l = sorted(l)  
bitsect.insort(l,item)
```

bitsect模块的bitsect*方法用于返回元素插入的index，但是不执行插入操作。
```Python
l = [1, 2, 5, 7, 10]  
bitsect.bitsect(l, 6)  #返回3，即如果插入，则index=3
#如果要插入重复数字，则index应该返回啥呢，这个时候就需要
#bitsect_left和bitsect_right函数了
bitsect.bitsect_left(l, 5) #返回2,左边的index
bitsect.bitsect_right(l, 5) #返回3,右边的index
#这两个函数对应的插入函数为:
#bitsect.insort_left和bitsect.insort_right
```

##3. Useful Decorators
###3.1 functools

##4. yield/contextmanager/with 
###4.1 yield
以前总是搞不清楚yield关键字和return的区别，现在回头看来那时候真是学的不好。

yield实际上的作用就是返回一个生成器(generator)：
```Python
 def createGenerator() :
    for i in range(3) :
        yield i*i
```

> yield i*i返回一个生成器，这个生成器中的代码不会立即执行. 第一次迭代中你的函数会执行，从开始到达 yield 关键字，然后返回
> yield 后的值作为第一次迭代的返回值.
> 然后，每次执行这个函数都会继续执行你在函数内部定义的那个循环的下一次，再返回那个值，直到没有可以返回的。

换句话说， 一个函数里面只要包含yield关键字，这个函数的返回值就是一个生成器，而这个函数里面的任何代码都不会立刻执行，代码执行只会发生在对生成器进行迭代的时候(因此生成器有个外号叫`惰性序列`)，验证如下：
```Python
def test():
    print 'before yield'
    for j in range(5):
        yield j*j

    print 'after yield'

for i in test():
    print i
    
#输出信息：
#before yield
#0
#1
#4
#9
#16
#after yield
```
显然， test函数中的代码在迭代时执行，并且yield之前的代码在迭代之前执行一次，迭代完成之后再执行yield之后的代码。再换句话说，yield返回的生成器把计算方法(包括相关数据)放到生成器中，每次迭代都会去执行一次代码。

经常看到有人问xrange和range函数的区别，大家都知道xrange节约内存，但是为啥呢？我们普遍的用法是：
```Python
for i in range(10):
    print i

for i in xrange(10):
    print i
```
这两种从使用方式和使用效果上来看基本没有区别，但是从本质上看，区别很大。range函数是生成一个list，xrange函数是生成一个生成器(二者都是可迭代对象)，如果range的范围很大， 那么生成的list会非常庞大，而xrange返回的是生成器，当中并不包含大量的数据，每一次迭代只包含当前的数据，因此节约内存。

###4.2 with statement 和 contextmanager
从Python 2.6开始支持with 语句，常用到的是：
```Python
with open('file', 'w') as fd:
    fd.write('Hello World')
```
这样我们就不需要关心fd的关闭问题了。实际上with的这种用法是通过实现两个函数：\_\_enter\_\_()和\_\_exit\_\_()分别在执行之前和执行之后调用，并且将值赋给as后面的变量。实际上close的调用放在了\_\_exit\_\_()里面。

那么如果自己也想让自己的函数使用with语句的话就需要自己实现\_\_enter\_\_()和\_\_exit\_\_()，当然Python中的contextmanager为我们做了这件事情。
```Python
@contextmanager
def myfunc(d):
    data = get_data(d)
    yield data
    clean_up()
    
with myfunc(d) as m:
    m.update()
```
其实，yield之前的代码是在\_\_enter\_\_()中，yield之后的代码在\_\_exit\_\_()中执行。


##5. 重载\_\_getattr\_\_和\_\_setattr\_\_

##6. map/reduce/filter
map是一个非常强大的工具。

```Python
map(...)
    map(function, sequence[, sequence, ...]) -> list
```
map第一个参数为一个函数，第二个参数是一个序列，最后返回的是序列作为函数参数的结果的list（有点绕）。
```Python
def func(i):
    return i*i
result = map(func, [i for i in range(10)])
#或者使用lambda函数
result = map(lambda i: i*i, [i for i in range(10)])
```

reduce的用法：
```Python
reduce(...)
    reduce(function, sequence[, initial]) -> value
```
第一个参数也是函数，第二个参数是一个序列，第三个参数是初始值。
reduce函数的作用是，按照序列的顺序对序列中的元素调用function，其中function必须有两个参数；如果有initial的话，从initial开始。
```Python
#1+2+3+4+5+6+7+8+9
result = reduce(lambda x,y: x+y, [i for i in range(10)])
```

filter的用法：
```Python
filter(...)
    filter(function or None, sequence) -> list, tuple, or string
```
顾名思义，filter做的是筛选，第一个参数是一个函数返回bool值，第二个参数是一个序列，filter的作用是把序列中item使function为True的元素返回到一个序列中。

##8. 设计模式
### 8.1 观察者模式
### 8.2 单例模式

##9. Daemon实现
有一个别人实现的Daemon方法，地址：http://www.jejik.com/articles/2007/02/a_simple_unix_linux_daemon_in_python/
 
Linux下Daemonize方法：

 1. fork一个进程出来，然后主进程退出。
 2. 修改工作目录到根
 3. 将子进程设置为会话组长（setsid()），脱离控制终端
 4. 修改umask
 5. 再fork一次，主进程退出，子进程成为真正的Daemon，避免进程再次获得控制终端。
 6. 将fork出来的标准输入输出错误都重定向到/dev/null


 

