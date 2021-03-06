1. python切片细节

   https://www.jianshu.com/p/15715d6f4dad

   ```csharp
   object[start_index:end_index:step] 
   ```

   **很重要记住step>0就是从左往右**

   - a[-1]是最后一个，所以a[-2]是倒数第2个
   - 切取完整的对象a[:]
   - a[::]从左往右，arr[::,-1]从右往左
   - a[:6:-1]从右往左直到第6个结束如果a从0~9，那么这个就是9,8,7
   - a[6::-1]  从第六个位置开始，从右往左取值
   - a[:-6:-1]   从终点开始，从右往左，直到第六个（不包括，相当于右开，所以也相当于取5个值）：[9,8,7,6,5]
   - **多层切片操作**                   a[:8] [2:5] [-1:]，可以一层一层分开来计算
   - 在某个位置插入元素，比如arr=[1,2,3]，那么arr[1:1]=[9]，那么就会变成arr[1,9,2,3]，插入只能是可迭代对象

   **原理**

   https://zhuanlan.zhihu.com/p/79752359

   1. python的可迭代对象比如集合[]，做切片操作[1:2]，就会调用___getitem___()方法：

      ```python
       class A():
           def __getitem__(self, key):
               print('__getitem__ is called')
               print('key={}'.format(key))
           def __setitem__(self, key, val):
               print('__setitem__ is called')
               print('key={}, val={}'.format(key, val))
      ```

      做切片的时候这个key值就是一个slice函数

      ```python
       >>> mySlice[0:3]
       __getitem__ is called
       key=slice(0, 3, None)
       >>> mySlice[:5]
       __getitem__ is called
       key=slice(None, 5, None)
       >>> mySlice[:100:5]
       __getitem__ is called
       key=slice(None, 100, 5)
      ```

      所以上面说的start、end、step都是slice函数的参数，默认值是None。

      **slice**

      `slice`类只有四个成员，其中`start`, `stop`, `step`是成员变量，`indices`类是成员函数。

      这个indices是这么用的：

      ```python
      >>> a = slice(5, 10, 2)
       >>> b = a.indices(20)
       >>> b
       (5, 10, 2)
       >>> list(range(*b))
       [5, 7, 9]
      ```

      a只是一个元祖，没有任何意义，当a调用了indices方法，传入一个length值，那么这个元祖就是为这个length服务了.所以range函数可以配合上这个元祖，返回长为20的数组进行这些切片操作得出来的值的真正索引

      

2. 手写一个打印程序运行时间的装饰器

   ```python
   import time
   def gettime(func):
       def inner(*args,**kwargs):
           start_time=time.time()
           time.sleep(2)
           func(*args,**kwargs)
           dur=time.time()-start_time
           print(dur)
       return inner
   
   @gettime
   def main():
       sum=1
       for i in range(1000):
           sum+=1
   main()
   ```

   如果装饰器需要接受参数，那么需要在原来的基础上再增加一层，见自己的另一篇。

   如果需要拿到原函数的返回值，那么就要在装饰器中调用原函数时接受返回值。

   **装饰器的原理**

   实现一个闭包，闭包就是将函数作为一个返回值传递给对象，它可以将本地变量脱离它的作用域而存在。通过@这个语法糖将函数作为参数传递到装饰器，装饰器内部去调用这个函数。

3. Python中的多继承

   每个类都有一个属性/      (__mro__)它的值是一个元祖，里面记录了当前类的方法解析顺序，知道object。

   class A(B,C)   如果B和C中都有一个相同的方法，那么在A中调用这个方法只会调用B的，因为在mro元祖中B的顺序在C之前。如果需要调用C，那么需要显示调用C.test(self)

   

4. GIL全局解释器锁。这个只有在Cpython解释器中才有，因为Cpython不是线程安全的，因为Cpython使用的是系统原生的线程，在Linux的pthread完全由操作系统调度执行。pthread本身不是线程安全的，需要使用者通过锁来实现多线程的安全运行，因此CPython解释器下的Python实现多线程也必然存在线程不安全的问题。所以在解释器层面实现了一把全局互斥锁，来保护Python对象从而实现对单核CPU的使用率

   https://zhuanlan.zhihu.com/p/87307313

   **GIL隐患**

   - 在多核CPU的主机上，由于存在GIL导致无论有多少线程开启，同一时刻只有一个线程在执行，这导致了在多核CPU情况下，效率还不如单线程执行效率高
   - 对于CPU密集型的计算类程序GIL就有比较大的问题，因为CPU密集型的程序本身没有太多等待，不需要解释器介入并且所有任务只能等待1个核心，其他核心空闲也无法使用，这么看对多核的使用确实很糟糕。

   **优势**

   - 在单核CPU上执行多线程可以由解释器实现有效的调度。在IO密集型程序上，GIL的存在也不会让效率很低下

   **GIL缺陷的解决方案**

   python作为生命力极强的热门语言，绝对不会在多核时代坐以待毙。即便有GIL的限制，仍然有许多方法让程序拥抱多核。

   - **多进程**:Python2.6引入了MultiProcess库来弥补Threading库中GIL带来的缺陷，基于此开发多进程程序，每个进程有单独的GIL,避免多进程之间对GIL的竞争，从而实现多核的利用，但是也带来一些同步和通信问题，这也是必然会出现的。
   - **Ctypes**:CPython的优势就是与C模块的结合，因此可以借助Ctypes调用C的动态库来实现将计算转移，C动态库没有GIL可以实现对多核的利用。（也就是python去调用C，实现多核利用）
   - **协程**:协程也是一个很好的手段，在Python3.4之前，官方没有对协程的支持，存在一些三方库的实现，比如gevent和Tornado。3.4之后就内置了asyncio标准库，官方真正实现了协程这一特性。

   GIL问题的并不是编程语言的本身问题，换做其他语言只是将问题转移到了用户层面，需要用户自己实现线程安全。

5. Python如何判断变量的类型

   - isinstance
   - type

   **区别**

   是否考虑继承关系

   - isinstance() 会认为子类是一种父类类型，考虑继承关系。Class A(B)  认为A就是B
   - type() 不会认为子类是一种父类类型，不考虑继承关系。相反

6. 元类 type

   实例是由类创建的，而类是由元类创建的。也就是元类创建的对象叫做类，类创建实例

7. 一个星（*）：表示接收的参数作为元组来处理

   两个星（**）：表示接收的参数作为字典来处理

8. xrange和range区别

   - Xrange生成的是一个生成器，而range是直接开辟了一个List，但是Python3已经取消了xrange
   - Xrange在生成很大序列的时候性能更好
   
9. Python中的dict实现

   内部是一个哈希表，处理哈希冲突的时候是采用了开放寻址的方法。当出现哈希冲突的时候，利用探测函数往后找到一个空的位置插入。

   