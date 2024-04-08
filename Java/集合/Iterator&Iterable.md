- [遍历方式](#遍历方式)
- [迭代器的实现](#迭代器的实现)

# 遍历方式

第一种：for 循环。

```java
for (int i = 0; i < list.size(); i++) {
    System.out.print(list.get(i) + "，");
}
```

第二种：迭代器。

```Java
Iterator it = list.iterator();
while (it.hasNext()) {
    System.out.print(it.next() + "，");
}
```

第三种：for-each。

for-each，其实背后也是 Iterator，看一下反编译后的代码（如下所示）就明白了。

```Java
for (String str : list) {
    System.out.print(str + "，");
}

# 反编译
Iterator var3 = list.iterator();

while(var3.hasNext()) {
    String str = (String)var3.next();
    System.out.print(str + "，");
}
```

# 迭代器的实现

```Java
public interface Iterable<T> {
    Iterator<T> iterator();

    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }

    default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
}


public interface Iterator<E> {
    boolean hasNext();

    E next();

    default void remove() {
        throw new UnsupportedOperationException("remove");
    }

    default void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (hasNext())
            action.accept(next());
    }
}

```

**都是实现Iterable接口，而不是直接实现Iterator接口。**

*为什么不直接将 Iterator 中的核心方法 hasNext、next 放到 Iterable 接口中呢？直接像下面这样使用不是更方便？*

**具体怎么遍历，要看怎么实现迭代器的，每种数据结构遍历的方式实现不同，**

```Java
// ArrayList 重写了 Iterable 接口的 iterator() 方法
public Iterator<E> iterator() {
    return new Itr();
}

/**
 * ArrayList 迭代器的实现，内部类。
 */
private class Itr implements Iterator<E> {

    /**
     * 游标位置，即下一个元素的索引。
     */
    int cursor;

    /**
     * 上一个元素的索引。
     */
    int lastRet = -1;

    /**
     * 预期的结构性修改次数。
     */
    int expectedModCount = modCount;

    /**
     * 判断是否还有下一个元素。
     *
     * @return 如果还有下一个元素，则返回 true，否则返回 false。
     */
    public boolean hasNext() {
        return cursor != size;
    }

    /**
     * 获取下一个元素。
     *
     * @return 列表中的下一个元素。
     * @throws NoSuchElementException 如果没有下一个元素，则抛出 NoSuchElementException 异常。
     */
    @SuppressWarnings("unchecked")
    public E next() {
        // 获取 ArrayList 对象的内部数组
        Object[] elementData = ArrayList.this.elementData;
        // 记录当前迭代器的位置
        int i = cursor;
        if (i >= size) {
            throw new NoSuchElementException();
        }
        // 将游标位置加 1，为下一次迭代做准备
        cursor = i + 1;
        // 记录上一个元素的索引
        return (E) elementData[lastRet = i];
    }

    /**
     * 删除最后一个返回的元素。
     * 迭代器只能删除最后一次调用 next 方法返回的元素。
     *
     * @throws ConcurrentModificationException 如果在最后一次调用 next 方法之后列表结构被修改，则抛出 ConcurrentModificationException 异常。
     * @throws IllegalStateException         如果在调用 next 方法之前没有调用 remove 方法，或者在同一次迭代中多次调用 remove 方法，则抛出 IllegalStateException 异常。
     */
    public void remove() {
        // 检查在最后一次调用 next 方法之后是否进行了结构性修改
        if (expectedModCount != modCount) {
            throw new ConcurrentModificationException();
        }
        // 如果上一次调用 next 方法之前没有调用 remove 方法，则抛出 IllegalStateException 异常
        if (lastRet < 0) {
            throw new IllegalStateException();
        }
        try {
            // 调用 ArrayList 对象的 remove(int index) 方法删除上一个元素
            ArrayList.this.remove(lastRet);
            // 将游标位置设置为上一个元素的位置
            cursor = lastRet;
            // 将上一个元素的索引设置为 -1，表示没有上一个元素
            lastRet = -1;
            // 更新预期的结构性修改次数
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }
}

```