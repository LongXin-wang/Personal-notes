- [hash 和 equal](#hash-和-equal)
- [HashMap](#hashmap)

# hash 和 equal

1.**hashCode()介绍:**

hashCode()的作用是获取哈希码，也称为散列码；它实际上是返回一个 int 整数。这个哈希码的作用是确定该对象在**哈希表**中的索引位置。hashCode()定义在JDK的Object 类中，这就意味着Java中的任何类都包含有hashCode()函数。另外需要注意的是：Object的hashCode()方法是本地方法，也就是用C语言或C+ 实现的，该方法通常用来将对象的内存地址转换为整数之后返回。

2.**为什么要有 hashCode？**

我们以“HashSet如何检查重复”为例子来说明为什么要有hashCode()方法？

当你把对象加入HashSet时，HashSet会先计算对象的hashcode值来判断对象加入的位置，同时也会与其他已经加入的对象的hashcode值作比较，如果没有相符的hashcode，HashSet会假设对象没有重复出现。但是如果发现有相同 hashcode 值的对象，这时会调用equals()方法来检查hashcode相等的对象是否真的相同。如果两者相同，HashSet就不会让其加入操作成功。如果不同的话，就会重新散列到其他位置。（摘自《Head First Java》第二版）。这样我们就大大减少了equals ()方法的次数，相应就大大提高了执行速度。

hashCode()的默认行为是对堆上的对象产生独特值。如果没有重写hashCode()，则该 class 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）。

**3.哈希冲突**

如果两个不同的元素，通过哈希函数得出的实际存储地址相同怎么办？也就是说，当我们对某个元素进行哈希运算，得到一个存储地址，然后要进行插入的时候，发现已经被其他元素占用了，其实这就是所谓的哈希冲突，也叫哈希碰撞。前面我们提到过，哈希函数的设计至关重要，好的哈希函数会尽可能地保证 计算简单和散列地址分布均匀,但是，我们需要清楚的是，数组是一块连续的固定长度的内存空间，再好的哈希函数也不能保证得到的存储地址绝对不发生冲突。那么哈希冲突如何解决呢？哈希冲突的解决方案有多种:开放定址法（发生冲突，继续寻找下一块未被占用的存储地址），再散列函数法，链地址法，而HashMap即是采用了链地址法，也就是数组+链表的方式。

```Java
 
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Pig pig = (Pig) o;
        boolean nameCheck=false;
        boolean ageCheck=false;
        
        if (this.name == pig.name) {
            nameCheck = true;
        } else if (this.name != null && this.name.equals(pig.name)) {
            nameCheck = true;
        }
 
        if (this.age == pig.age) {
            ageCheck = true;
        } else if (this.age != null && this.age.equals(pig.age)) {
            ageCheck = true;
        }
 
       if (nameCheck && ageCheck){
           return true;
       }
 
       return  false;
    }
    
    
    @Override
    public int hashCode() {
 
        int result = 17;
        result = 31 * result + name.hashCode();
        result = 31 * result + age;
        return result;
        
    }
    
    
    简易写法
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Pig pig = (Pig) o;
        return Objects.equals(name, pig.name) &&
                Objects.equals(age, pig.age);
    }
 
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
```

# HashMap

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202405221354451.png)