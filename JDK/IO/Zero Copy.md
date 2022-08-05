### 传统IO

**read**：将数据从磁盘通过DMA读取到内核缓存区中，在拷贝到用户缓冲区

**write**:先将数据写入到socket缓冲区中，经过DMA写入网卡设备

![在这里插入图片描述](https://img-blog.csdnimg.cn/7583346eb0c54538927bd5e4bcbe2e1f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LuO5pio5aSp,size_20,color_FFFFFF,t_70,g_se,x_16)

4次切换，4次拷贝

### 虚拟内存

**1.虚拟内存空间可以远远大于物理内存空间
2.多个虚拟内存可以指向同一个物理地址**
正是多个虚拟内存可以指向同一个物理地址，可以把内核空间和用户空间的虚拟地址映射到同一个物理地址，这样的话，就可以减少IO的数据拷贝次数。

#### mmap

用户态可以直接访问内核态的数据

4次切换，3次拷贝

#### sendFile

sendfile表示在两个文件描述符之间传输数据，它是在操作系统内核中操作的，避免了数据从内核缓冲区和用户缓冲区之间的拷贝操作

![在这里插入图片描述](https://img-blog.csdnimg.cn/96525842f35e4f7781ceabaa56fb8524.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LuO5pio5aSp,size_20,color_FFFFFF,t_70,g_se,x_16)

2次切换，3次拷贝

**sendfile +DMA scatter/gather实现的零拷贝**

linux2.4版本后，对sendfile做了优化升级，引入SG-DMA技术，其实就是对DMA拷贝加入了scatter/gather操作，它可以直接从内核空间缓冲区中将数据读取到网卡，这样的话还可以省去CPU拷贝。

![在这里插入图片描述](https://img-blog.csdnimg.cn/67daa8fad1ef47298782084b4fb36b56.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LuO5pio5aSp,size_20,color_FFFFFF,t_70,g_se,x_16)

2次切换，2次拷贝，CPU全程不参与数据搬运