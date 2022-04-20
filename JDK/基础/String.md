## String

一旦一个string对象在内存(堆)中被创建出来，他就无法被修改。特别要注意的是，String类的所有方法都没有改变字符串本身的值，都是返回了一个新的对象。

- String s   = "a" + "b"，编译器会进行常量折叠(因为两个都是编译期常量，编译期可知)，即代码变成 String s  = "ab"

- 对于能够进行优化的(String s  = "a" + 变量 等) 最后会调用 StringBuilder 的 append() 方法替代，最后调用 toString() 方法     (底层就是一个 new String())

  ```java
  //StringBuilder的toString调用的构造函数，不会放入常量池
  String(byte[] value, byte coder) {
          this.value = value;
          this.coder = coder;
      }
  ```

- Java中的+对字符串的拼接，其实现原理是使用StringBuilder.append，如果拼接前后出现变量，都是放入堆，相当于创建了新对象。

 1.int i = 5;
 2.String i1 = "" + i;
 3.String i2 = String.valueOf(i);
 4.String i3 = Integer.toString(i);

第三行和第四行没有任何区别，因为String.valueOf(i)也是调用Integer.toString(i)来实现的。

第二行代码其实是String i1 = (new StringBuilder()).append(i).toString();，首先创建一个StringBuilder对象，然后再调用append方法，再调用toString方法。

**new String("ab")创建了几个对象？**

2个，一个在堆中创建，一个在字符串常量池中，字节码指令为ldc，放入"ab"

**new String("a") + new String("b")呢？**

4+2=6个，别忘了用'+'拼接会有StringBuilder！最后调用StringBuilder.toString()时会new 一个 String("ab")，但"ab"不会放入常量池,而手动编写的new String()创建对象后会放入常量池

### 字符串长度有限制吗？

字符串有长度限制，在编译期，要求字符串常量池中的常量不能超过65535，并且在javac执行过程中控制了最大值为65534。

### 为什么不可变？

不可变类的实例一旦创建，其成员变量的值就不能被修改。这样设计有很多好处，比如可以缓存hashcode、使用更加便利以及更加安全等

*StringUtils.join更擅长处理字符串数组或者列表的拼接*

### 字符串常量池

**intern()**

在JDK 7以前的版本中，字符串常量池是放在永久代中的。

jdk6中，s.intern()检查到常量池中没有s，就会复制一份s，然后放入池中。而7后，如果池中没有s，会把s的引用复制一份放入池中。

所以jdk7后，调用intern()方法会在常量池中存放一个指向堆中字符串的指针。而不是字符串本身。

因为按照计划，JDK会在后续的版本中通过元空间来代替永久代，所以首先在JDK 7中，将字符串常量池先从永久代中移出，暂时放到了堆内存中。

在JDK 8中，彻底移除了永久代，使用元空间替代了永久代，于是字符串常量池再次从堆内存移动到永久代中

如果虚拟机始终将相同的字符串共享， 就可以使用运算符检测是否相等。**但实际上 只有字符串常量是共享的，而 + 或 substring 等操作产生的结果并不是共享的。**

到目前为止switch支持这样几种数据类型：byte short int char String

switch只支持一种数据类型，那就是整型，其他数据类型都是转换成整型之后在使用switch的

## Stringbuilder

底层数组未被final修饰，长度可变，占用空间少

初始容量为16个字符