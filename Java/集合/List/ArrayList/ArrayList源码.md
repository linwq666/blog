## ArrayList源码解析

1.  整体架构
    *   基于数组实现
    *   查询速度快，是根据索引查询数据快：查询数据通过地址值和索引定位。
    *   删除效率低：可能需要把后面的很多数据进行迁移。
    *   添加效率低：需要把后面数据往后移，或者可能需要进行数组扩容。
    *   是非线程安全的。
    *   增强for循环或者使用迭代器过程中，如果数组大小被改变，会快速失败，抛出异常。
2.  原理
    *   利用无参构造器创建的集合，会在底层创建一个默认长度为0的数组。
    *   添加第一个元素时，底层会创建一个新的长度为10的数组。
    *   存满时，会扩容1.5倍。
    *   如果一次添加多个元素，1.5倍放不下，则新创建数组的长度以实际为准。
3.  源码解析
    *   初始化

        ```java
        public class ArrayList<E> extends AbstractList<E>
                implements List<E>, RandomAccess, Cloneable, java.io.Serializable
        {
            private static final long serialVersionUID = 8683452581122892189L;

            /**
             * 默认初始化容量.
             */
            private static final int DEFAULT_CAPACITY = 10;

            /**
             * 初始化一个长度为0的数组.
             */
            private static final Object[] EMPTY_ELEMENTDATA = {};

            /**
             * 默认大小的空数组
             */
            private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA ={};

            /**
             * 实际元素对象数据数组
             */
            transient Object[] elementData; // non-private to simplify nested class access

            /**
             * 当前集合的大小.
             *
             * @serial
             */
            private int size;

            /**
             * 指定容量大小进行创建ArrayList 
             * 如果指定容量大小等于0则直接用空数组进行初始化
             *
             * @param  initialCapacity  容量大小
             * @throws IllegalArgumentException if the specified initial capacity
             *         is negative
             */
            public ArrayList(int initialCapacity) {
                if (initialCapacity > 0) {
                    this.elementData = new Object[initialCapacity];
                } else if (initialCapacity == 0) {
                    this.elementData = EMPTY_ELEMENTDATA;
                } else {
                    throw new IllegalArgumentException("Illegal Capacity: "+
                                                       initialCapacity);
                }
            }

            /**
             * 默认构造，采用默认数组进行初始化.
             */
            public ArrayList() {
                this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
            }

            /**
             * 外部传入一个集合
             *
             * @param c the collection whose elements are to be placed into this list
             * @throws NullPointerException if the specified collection is null
             */
            public ArrayList(Collection<? extends E> c) {
                //将外部传入进来的集合数据转化为数组赋值给 元素对象数组
                elementData = c.toArray();
                //更新集合大小
                if ((size = elementData.length) != 0) {
                    // c.toArray might (incorrectly) not return Object[] (see 6260652)
                    if (elementData.getClass() != Object[].class)
                        elementData = Arrays.copyOf(elementData, size, Object[].class);
                } else {
                    // // 如果传入进来的是一个空的集合，那么用空数组替换.
                    this.elementData = EMPTY_ELEMENTDATA;
                }
            }
        }
        ```
    *   添加

        ```java
        /**
             * 将元素e添加到当前elementData中.
             *
             * @param e element to be appended to this list
             * @return 返回是否添加成功)
             */
            public boolean add(E e) {
                //每次添加数组的时候都会检查数组容量是否需要扩容
                ensureCapacityInternal(size + 1);  // Increments modCount!!
                //赋值
                elementData[size++] = e;
                return true;
            }

            /**
             * 在指定索引位置index上插入元素.
             *
             * @param index 插入指定元素的索引
             * @param element 待插入的元素
             * @throws IndexOutOfBoundsException {@inheritDoc}
             */
            public void add(int index, E element) {
                //校验索引是否合理
                rangeCheckForAdd(index);
                //检查是否需要扩容
                ensureCapacityInternal(size + 1);  // Increments modCount!!
                //将需要插入的index位置往后移动一位
                System.arraycopy(elementData, index, elementData, index + 1,size - index);
                elementData[index] = element;
                size++;
            }
        ```
    *   删除

        ```java
        /**
             * 删除此集合中指定位置的元素.
             * @param index the index of the element to be removed
             * @return 返回被删除的元素
             * @throws IndexOutOfBoundsException {@inheritDoc}
             */
            public E remove(int index) {
                //检查索引是否合理
                rangeCheck(index);
                //数据修改次数
                modCount++;
                E oldValue = elementData(index);

                int numMoved = size - index - 1;
                if (numMoved > 0)
                    //将index后面的元素向前移动一位
                    System.arraycopy(elementData, index+1, elementData, index,numMoved);
                elementData[--size] = null; // clear to let GC do its work

                return oldValue;
            }

            /**
             * 从集合删除指定元素第一次出现的元素.
             *
             * @param o 要删除的元素
             * @return 返回是否删除成功
             */
            public boolean remove(Object o) {
                if (o == null) {
                    for (int index = 0; index < size; index++)
                        if (elementData[index] == null) {
                            fastRemove(index);
                            return true;
                        }
                } else {
                    // 遍历元素 对象元素相等的话就直接删除
                    for (int index = 0; index < size; index++)
                        if (o.equals(elementData[index])) {
                            fastRemove(index);
                            return true;
                        }
                }
                return false;
            }

            /**
             * 删除所有元素.
             */
            public void clear() {
                modCount++;

                //直接把容量空间全部置为 NULL 交于 GC
                for (int i = 0; i < size; i++)
                    elementData[i] = null;
                size = 0;
            }
        ```
    *   改变元素

        ```java
            /**
             * 将指定位置的元素替换成新的元素.
             *
             * @param index index of the element to replace
             * @param element element to be stored at the specified position
             * @return 返回替换前的元素
             * @throws IndexOutOfBoundsException {@inheritDoc}
             */
            public E set(int index, E element) {
                //检查索引是否合理
                rangeCheck(index);
                //取出旧元素
                E oldValue = elementData(index);
                //更新新元素
                elementData[index] = element;
                return oldValue;
            }
        ```
    *   查询数据

        ```java
            /**
             * Returns the element at the specified position in this list.
             *
             * @param  index index of the element to return
             * @return the element at the specified position in this list
             * @throws IndexOutOfBoundsException {@inheritDoc}
             */
            public E get(int index) {
                rangeCheck(index);
                return elementData(index);
            }
        ```
    *   动态扩容

        ```java
             /**
             * 计算容量.
             *
             * @param elementData 当前元素数组
             * @param minCapacity 最小容量
             * @return 返回容量
             * @throws IndexOutOfBoundsException {@inheritDoc}
             */
            private static int calculateCapacity(Object[] elementData, int minCapacity) {
                //当前集合为空时
                if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
                    //和默认容量比较，取最大值
                    return Math.max(DEFAULT_CAPACITY, minCapacity);
                }
                return minCapacity;
                }

            private void ensureCapacityInternal(int minCapacity) {
                ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
            }

            private void ensureExplicitCapacity(int minCapacity) {
                modCount++;

                //如果最小的扩容容量比当前的元素数据还要大的时候进行扩容
                if (minCapacity - elementData.length > 0)
                    //扩容
                    grow(minCapacity);
            }

            /**
             * 扩容.
             *
             * @param minCapacity the desired minimum capacity
             */
            private void grow(int minCapacity) {
                // overflow-conscious code
                int oldCapacity = elementData.length;
                //扩容到当前容量的1.5倍
                int newCapacity = oldCapacity + (oldCapacity >> 1);
                //如果最小容量大于新容量，则按最小容量计算
                if (newCapacity - minCapacity < 0)
                    newCapacity = minCapacity;
                //检查是否超过最大容量
                if (newCapacity - MAX_ARRAY_SIZE > 0)
                    newCapacity = hugeCapacity(minCapacity);
                // minCapacity is usually close to size, so this is a win:
                elementData = Arrays.copyOf(elementData, newCapacity);
            }

            private static int hugeCapacity(int minCapacity) {
                if (minCapacity < 0) // overflow
                    throw new OutOfMemoryError();
                return (minCapacity > MAX_ARRAY_SIZE) ?
                    Integer.MAX_VALUE :
                    MAX_ARRAY_SIZE;
            }
        ```
    *

