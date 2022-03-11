把一个任务分为子任务然后合并，这里以从1加到n举例

```java

class newTask extends RecursiveTask<Integer>{
    int begin = 0,end = 0;

    public newTask(int begin, int end){
        this.begin = begin;
        this.end = end;
    }
    @Override
    protected Integer compute() {
        if (begin == end){
            return begin;
        }
        if (end - begin == 1)
            return end + begin;

        int mid = (end + begin) / 2;

        newTask t1 = new newTask(begin,mid);
        t1.fork();
        newTask t2 = new newTask(mid + 1, end);
        int result = t1.join() + t2.join();
        return result;
    }
}
```

```java
public static void main(String[] args) {
        ForkJoinPool newPool = new ForkJoinPool(4);
        System.out.println(newPool.invoke(new newTask(1,10)));
    }
```

