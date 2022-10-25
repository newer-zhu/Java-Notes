### Stream

###### 中间流

| `filter(Predicate p)` | 接收Lambda ，从流中排除某些元素                            |
| --------------------- | ---------------------------------------------------------- |
| `distinct()`          | 筛选，通过流所生成元素的hashCode() 和equals() 去除重复元素 |
| `limit(long maxSize)` | 截断流，使其元素不超过给定数量                             |
| `skip(long n)`        |                                                            |

方法	描述
map(Function f)	接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素。
mapToDouble(ToDoubleFunction f)	接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的DoubleStream。
mapToInt(ToIntFunction f)	接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的IntStream。
mapToLong(ToLongFunction f)	接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的LongStream。
flatMap(Function f)

| 方法                     | 描述                             |
| ------------------------ | -------------------------------- |
| `sorted()`               | 产生一个新流，其中按自然顺序排序 |
| `sorted(Comparator com)` |                                  |

###### 终止流

方法	描述
allMatch(Predicate p)	检查是否匹配所有元素
anyMatch(Predicate p)	检查是否至少匹配一个元素
noneMatch(Predicate p)	检查是否没有匹配所有元素
findFirst()	返回第一个元素
findAny()	返回当前流中的任意元素
count()	返回流中元素总数
max(Comparator c)	返回流中最大值
min(Comparator c)	返回流中最小值
forEach(Consumer c)

| 方法                               | 描述                                          |
| ---------------------------------- | --------------------------------------------- |
| `reduce(T iden, BinaryOperator b)` | 可以将流中元素反复结合起来，得到一个值。返回T |
| `reduce(BinaryOperator b)`         |                                               |

| `collect(Collector c)` | 将流转换为其他形式。接收一个Collector接口的实现，用于给Stream中元素做汇总的方法 |
| ---------------------- | ------------------------------------------------------------ |
|                        |                                                              |