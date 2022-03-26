**CAS**

cas需要和volatile配合，cas适合线程数少，cpu多核

AtomicStampedReference可以解决ABA问题

AtomicMarkableReference只关注有没有发生改变而不在乎改变的次数

**CPU缓存**

CPU个主存之间的缓存，以行为单位。

缓存行：64byte

缓存行伪共享，如果两个cell处于一个缓存行，其中cell[1]被改变，则另一个cpu的缓存行会失效，如果此时另一个cpu准备对cell[2]改变，则必须重新去内存获取值

@Contend可以解决，在注解的对象前后增加128byte的padding，则cpu将对象预读至缓存时会占用不同的缓存行，这样不会造成缓存行的失效

