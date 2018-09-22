# List相关实现类的源码解析（JDK1.8）

> 2018.9.22-

## List的架构图

![List的架构图](https://images0.cnblogs.com/blog/497634/201309/08214705-fbbcdd2d785f40f5b773041801bd74f1.jpg)

## ArrayList

* 继承关系：ArrayList -> AbstractList 实现 List接口

* ArrayList 是一个数组队列，相当于 动态数组。与Java中的数组相比，它的容量能动态增长。不是线程安全的。ArrayList包含了两个重要的对象：elementData`(Object[]类型的数组)` 和 size

* 遍历ArrayList时，使用随机访问(即，通过索引序号访问)效率最高

* 转数组：Integer[] newText = v.toArray(new Integer[v.size()])

## Fail-Fast机制

* fail-fast 机制是java集合(Collection)中的一种错误机制。当多个线程对同一个集合的内容进行操作时，就可能会产生fail-fast事件。

* 通过modCount的值来判断是否多线程同时操作，modCount用来记录List修改的次数：每修改一次(添加/删除等操作)，将modCount+1

## LinkedList

* 继承关系：LinkedList -> AbstractSequentialList -> AbstractList 实现 List接口

* LinkedList的本质是`双向链表`，包含两个重要的成员：header 和 size

header是双向链表的表头，它是双向链表节点所对应的类Entry的实例。Entry中包含成员变量： previous, next, element。其中，previous是该节点的上一个节点，next是该节点的下一个节点，element是该节点所包含的值

* 特点：顺序访问会非常高效，而随机访问效率比较低，因此不要通过随机访问去遍历LinkedList

* `双向链表和索引值联系起来`

通过一个计数索引值来实现的。例如，当我们调用get(int location)时，首先会比较“location”和“双向链表长度的1/2”；若前者大，则从链表头开始往后查找，直到location位置；否则，从链表末尾开始先前查找，直到location位置

* LinkedList可以作为`FIFO(先进先出)的队列`或`LIFO(后进先出)的栈`

## Vector：矢量队列（已经被淘汰，很少用到）

* 继承关系：Stack -> AbstractList 实现 List接口

* Vector本质是数组

* 和ArrayList不同，Vector中的操作是线程安全的

## Stack：栈

* 继承关系：Stack -> Vector -> AbstractList 实现 List接口

* 本质是数组

* 特性：先进后出

* 执行push时(即，将元素推入栈中)，执行peek时(即，取出栈顶元素，不执行删除)，执行pop时(即，取出栈顶元素，并将该元素从栈中删除)

## 其他知识点

* ArrayList内部多次调用方法：`Arrays.copyOf`(U[] original, int newLength, Class<? extends T[]> newType)

复制数组数据到返回结果数组，内部调用System.arraycopy

* `System.arraycopy`(Object src,int srcPos,Object dest, int destPos,int length)

native方法，复制数组数据到目标数组

* RandomAccess接口

RandomAccess接口是一个空接口， 是一个`标记接口`，用于标明实现该接口的List支持快速随机访问，主要目的是使算法能够在随机和顺序访问的list中表现的更加高效。部分方法中会判断当前对象是否实现了RandomAccess接口接口，若实现了则采用随机访问的方式获取数据

```java
public static <T> int binarySearch(List<? extends Comparable<? super T>> list, T key) {
    if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
        return Collections.indexedBinarySearch(list, key);
    else
        return Collections.iteratorBinarySearch(list, key);
}
```

* 求交集：retainAll

* List中有一个subList方法，用来返回一个list的一部分的视图

称为视图，是因为返回的list是靠原来的list支持的，做的操作会影响到彼此。如果你在调用了sublist返回了子list之后，如果修改了原list的大小，那么之前产生的子list将会失效，变得不可使用。`用法`：删除一个list的某个区段，list.subList(from, to).clear();

* `(java8)`ArrayList.replaceAll：使用对该元素应用运算符的结果来替换列表中的每个元素

* `(java8)`ArrayList.removeIf：使用函数式接口来移除匹配的元素

* `在同一个包里`的子类中实例化NewObject类获得对象，然后可用该对象访问`protected`修饰的方法或者属性

## 总结⭐

* 对于需要快速插入，删除元素，应该使用LinkedList。

* 对于需要快速随机访问元素，应该使用ArrayList。

* 对于“单线程环境” 或者 “多线程环境，但List仅仅只会被单个线程操作”，此时应该使用非同步的类(如ArrayList、LinkedList)。

* 对于“多线程环境，且List可能同时被多个线程操作”，此时，应该使用同步的类(如Vector或`CopyOnWriteArrayList`)