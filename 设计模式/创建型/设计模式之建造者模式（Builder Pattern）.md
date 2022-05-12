
## 概念

> 在创建者模式中，客户端不再负责对象的创建与组装，而是把这个对象创建的责任交给其具体的创建者类，把组装的责任交给组装类，客户端只负责对象的调用，从而明确了各个类的职责。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200527202648675.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0MDI1ODk0,size_16,color_FFFFFF,t_70#pic_center)
假设要建造一栋建筑，就要有建筑的一个抽象类，然后让不同类型的建筑去继承这个类。
先创建出这个**建筑抽象类**：这个类有三个属性

```java
public class Building {
    private String base;//地基
    private int floors;//层数
    private int area;//面积

    public Building(){};
    
    public void setBase(String base) {
        this.base = base;
    }
    
    public void setFloors(int floors) {
        this.floors = floors;
    }

    public int getArea() {
        return area;
    }

    public void setArea(int area) {
        this.area = area;
    }
}
```
其次，由于建筑有不同种类，先创建出两个具体的产品来继承这个建筑抽象类：
**公寓类**
```java
public class Apartment extends Building{

    private String base = "公寓地基";
    private int floors = 20;
    private int area = 50000;

    public Apartment(){
        super.setArea(this.area);
        super.setBase(this.base);
        super.setFloors(this.floors);
    };

    @Override
    public String toString() {
        return "Apartment{" +
                "base='" + base + '\'' +
                ", floors=" + floors +
                ", area=" + area +
                '}';
    }
}
```
**别墅类**

```java
public class Villa extends Building {
    String base = "三层小洋楼";
    int area = 10000;
    int floors = 3;

    public Villa(){
        super.setArea(this.area);
        super.setBase(this.base);
        super.setFloors(this.floors);
    }

    @Override
    public String toString() {
        return "Villa{" +
                "base='" + base + '\'' +
                ", area=" + area +
                ", floors=" + floors +
                '}';
    }
}
```
有了产品以后就需要有工厂去生产它们，这里还要创建一个生产过程的抽象类，里面有建造建筑物的基本三个流程。
**生产过程抽象类**
```java
public abstract class BuildProcess {
    //打地基
    abstract void buildBase();
    //刷漆
    abstract void paintWall();
    //封顶
    abstract void buildRoof();
    //获得完工的建筑
    public abstract Building getBuilding();
}
```
**生产别墅过程的类**
```java
public class BuildVilla extends BuildProcess {
    @Override
    void buildBase() {
        System.out.println("别墅打地基");
    }

    @Override
    void paintWall() {
        System.out.println("别墅刷漆");
    }

    @Override
    void buildRoof() {
        System.out.println("别墅封顶");
    }

    /**
     *建造别墅并返回
     */
    @Override
    public Building getBuilding() {
        buildBase();
        paintWall();
        buildRoof();
        return new Villa();
    }
}
```
**生产公寓过程的类**

```java
public class BuildApartment extends BuildProcess {
    @Override
    void buildBase() {
        System.out.println("公寓打地基");
    }
    
    @Override
    void paintWall() {
        System.out.println("公寓刷漆");
    }

    @Override
    void buildRoof() {
        System.out.println("公寓封顶");
    }

    @Override
    public Building getBuilding() {
        buildBase();
        paintWall();
        buildRoof();
        return new Apartment();
    }
}
```
有了产品的类和生产的类及它们的父类，现在还需要一个指挥建筑师，建筑师不需要做建筑工作等脏活累活，他只需要接到客户的需求并返回一个实例就行了。
**建筑师类**
```java
public class BuildingDirector {
    BuildProcess process = null;
    /**
     * 构造器指定生产过程
     */
    public BuildingDirector(BuildProcess process) {
        this.process = process;
    }

    /**
     * 返回一个Building对象
     * @return
     */
    public Building build(){
        return process.getBuilding();
    }
}
```
**客户类**
客户类要先找到一个建筑师对象，然后给建筑师一个构造函数的参数，也就是他想要的建筑类型的生产过程的类就行了。
```java
public class Client {
    public static void main(String[] args) {
        BuildingDirector director = new BuildingDirector(new BuildApartment());
        Building villa = director.build();
        System.out.println(villa);

        System.out.println("---------------------------------");

        BuildingDirector director1 = new BuildingDirector(new BuildVilla());
        Building apartment = director1.build();
        System.out.println(apartment);
    }
}
```
**运行结果**
```java
公寓打地基
公寓刷漆
公寓封顶
Apartment{base='公寓地基', floors=20, area=50000}
---------------------------------
别墅打地基
别墅刷漆
别墅封顶
Villa{base='三层小洋楼', area=10000, floors=3}

Process finished with exit code 0
```

