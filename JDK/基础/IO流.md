![图片](https://mmbiz.qpic.cn/mmbiz_jpg/rAMaszgAyWpblibxHNficQAaicBURw5uxaY0c1KTiaia1oCJn3CMSict7ZOCET2GwvxkMl8WnH9eCEobicoDkuEAOTsmw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://img-blog.csdn.net/20160421004327228?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

**字节流**

![img](https://img-blog.csdn.net/20160421005454478?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

**字符流**

![img](https://img-blog.csdn.net/20160421005558870?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

**转换流**

1. InputStreamReader:*字节到字符的桥梁*
2. OutputStreamWriter:*字符到字节的桥梁*

**区别**

字节流没有缓冲区，是直接输出的，而字符流是输出到缓冲区的。因此在输出时，字节流不调用colse()方法时，信息已经输出了，而字符流只有在调用close()方法关闭缓冲区时，信息才输出。要想字符流在未关闭时输出信息，则需要手动调用flush()方法。

## BIO

## NIO

原来的 I/O 以流的方式处理数据，而 NIO 以块的方式处理数据
