## 结构

1. 魔数（4Bytes）

   - 识别Class文件，安全保障

2. 版本号（4Bytes）

   - 高版本兼容低版本

3. 常量池

   计数器（2Bytes），从1开始

   常量池表cp_info，存放编译时尝生的字面量和符号引用，类加载后会放进运行时常量池
   
4. 访问标识（2Bytes）

   ACC_开头，有PUBLIC、FINAL、SUPER、INTERFACE、ABSTRACT、ENUM、ANNOTATION、SYNTHETIC
   
5. 类索引、父类索引、接口索引集合

6. 字段表集合

   不会出现父类的字段

7. 方法表

   不包括父类继承的方法

