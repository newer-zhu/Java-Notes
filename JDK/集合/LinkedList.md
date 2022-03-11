![image-20210815143700675](E:\学习笔记\typora\img\image-20210815143700675.png)

 LinkedList 实现了 Queue 接口,该接口比List提供了更多的方法

## 重要变量

```java
transient int size;
transient LinkedList.Node<E> first;
transient LinkedList.Node<E> last;
```

```java
//节点内部类，是一个双向链表
private static class Node<E> {
    E item;
    LinkedList.Node<E> next;
    LinkedList.Node<E> prev;

    Node(LinkedList.Node<E> prev, E element, LinkedList.Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

## 重要函数

```java
public boolean add(E e) {
    this.linkLast(e);
    return true;
}
//尾部添加节点
void linkLast(E e) {
        LinkedList.Node<E> l = this.last;
        LinkedList.Node<E> newNode = new LinkedList.Node(l, e, (LinkedList.Node)null);
        this.last = newNode;
        if (l == null) {
            this.first = newNode;
        } else {
            l.next = newNode;
        }

        ++this.size;
        ++this.modCount;
    }
//在指定节点前插入一个节点
void linkBefore(E e, LinkedList.Node<E> succ) {
        LinkedList.Node<E> pred = succ.prev;
        LinkedList.Node<E> newNode = new LinkedList.Node(pred, e, succ);
        succ.prev = newNode;
        if (pred == null) {
            this.first = newNode;
        } else {
            pred.next = newNode;
        }

        ++this.size;
        ++this.modCount;
    }

public E remove(int index) {
        this.checkElementIndex(index);
        return this.unlink(this.node(index));
    }

public E remove() {
        return this.removeFirst();
    }

public boolean remove(Object o) {
        LinkedList.Node x;
        if (o == null) {//删除第一个元素为空的j
            for(x = this.first; x != null; x = x.next) {
                if (x.item == null) {
                    this.unlink(x);
                    return true;
                }
            }
        } else {
            for(x = this.first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    this.unlink(x);
                    return true;
                }
            }
        }

        return false;
    }
//把指定节点从当前链表删除
E unlink(LinkedList.Node<E> x) {
        E element = x.item;
        LinkedList.Node<E> next = x.next;
        LinkedList.Node<E> prev = x.prev;
        if (prev == null) {
            this.first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }

        if (next == null) {
            this.last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        --this.size;
        ++this.modCount;
        return element;
    }
```

l

