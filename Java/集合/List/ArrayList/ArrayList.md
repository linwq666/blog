ArrayList是一种有序、可重复、有索引的容器，底层数据结构基于数组实现，数组内存地址是连续的。

*   优点：根据索引查询比较快
*   缺点：删除和插入比较慢

1.  ArrayList提供了三种构造器用于创建对象

    *   ArrayList() 默认构造器
    *   ArrayList(int initialCapacity) 指定容量构造器
    *   ArrayList(Collection\<? extends E> c) 指定集合构造器
2.  ArrayList提供了添加、移除、修改、获取等常用方法

    *   public boolean add(E e)   将指定的元素添加到此集合的末尾
    *   public void add(int index,E element) 在此集合中指定的位置插入指定的元素
    *   public E get(int index)  返回指定索引处的元素
    *   public int size() 返回集合中的元素的个数
    *   public E remove(int index) 删除指定索引处的元素，返回被删除的元素
    *   public boolean remove(Object o) 删除指定的元素，返回删除是否成功
    *   public E set(int index,E element) 修改指定索引处的元素

```java
		ArrayList<String> list = new ArrayList<>();
		list.add("张三");
		list.add("李四");
		list.add("王五");
		System.out.println(list);//[张三, 李四, 王五]
		//在此集合中索引2位置插入指定的元素
		list.add(2,"老六");
		System.out.println(list);//[张三, 李四, 老六, 王五]
		//获取索引1位置的元素
		String element = list.get(1);
		System.out.println(element);//李四
```

## 遍历方式

```java
        List<Integer> list = new ArrayList();
        list.add(1);
        list.add(2);
        list.add(3);
        //1、for循环
        for (int i = 0; i < list.size(); i++) {
            int e = list.get(i);
            System.out.println(e);
        }
        //2、迭代器
        Iterator<Integer> it = list.iterator();
        while (it.hasNext()){
            System.out.println(it.next());
        }
        //3、增强for循环（foreach遍历）
        for (Integer i:list) {
            System.out.println(i);
        }
        //4、JDK 1.8开始之后的Lambda表达式
        list.forEach(i->{
            System.out.println(i);
        });
```

