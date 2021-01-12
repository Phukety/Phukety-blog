---
title: Collections.unmodifiableList方法的使用与场景
date: 2021-01-11 21:50:58
cover: images/java.jpeg
photos: 
  -  images/java.jpeg
categories: 
  - java基础 
tags: 
  - java
  - 集合
---
本文主要简单介绍java中Collections.unmodifiableList方法的使用与场景
<!--more -->

# **问题引入**

我们现在假设这样一个场景：学校里，学生可以选择课程，而学生和课程是一对多的关系。换言之，学生实体维护了学生信息以及课程列表，代码如下：

```java
public class Student {
    // 学生姓名
    private String name;
    // 学生的课程
    private List<String> courses;
    public Student(String name, List<String> courses) {
        this.name = name;
        this.courses = courses;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public List<String> getCourses() {
        return courses;
    }
    public void setCourses(List<String> courses) {
        this.courses = courses;
    }
    public static void main(String[] args) {
        // 初始化学生课程
        Student student = new Student("张三", new ArrayList<String>() {{
            add("数学");
            add("语文");
            add("英语");
        }});
        System.out.println("当前课程数量为: " + student.getCourses().size());

        // 如果想要添加其它课程，则可以获取student的课程对象，进行添加
        List<String> courseList = student.getCourses();
        // 继续添加另外的课程
        courseList.add("物理");
        courseList.add("化学");
        System.out.println("当前课程数量为: " + student.getCourses().size());

        // 是否可能存在问题？
        // 如何解决？
    }
}
```

由代码中的main方法可以看到，我们通过getCourses()获取courses集合的引用后，则可以任意的给该学生添加课程。

并且setter和getter均可以设置课程，导致对象混乱，这并不是我们想看到的设计。

出现这种情况的原因核心就是getter方法拿到引用后依然能对集合进行操作，那怎样才能不让对集合操作呢？

# **只读集合**

Java Util包中的Collections类提供了一个静态方法unmodifiableList(List<T>list)，这个方法传入一个需要设置为只读的集合，返回一个UnmodifiableList对象

```java
    public static <T> List<T> unmodifiableList(List<? extends T> list) {
        return (list instanceof RandomAccess ?
                new UnmodifiableRandomAccessList<>(list) :
                new UnmodifiableList<>(list));
    }
```

这里根据传入的集合是否实现RandomAccess接口，来判断构建UnmodifiableRandomAccessList还是UnmodifiableList对象。

顺带提一下，在集合中ArrayList等实现了RandomAccess，LinkedList等未实现该接口。该接口是一个标志类接口，在对集合进行随机查找时，能支持快速随机访问。并且for循环遍历会比iterator遍历效率更高。

我们再看看具体实现

```java
public boolean equals(Object o) {return o == this || list.equals(o);}
        public int hashCode()           {return list.hashCode();}

        public E get(int index) {return list.get(index);}
        public E set(int index, E element) {
            throw new UnsupportedOperationException();
        }
        public void add(int index, E element) {
            throw new UnsupportedOperationException();
        }
        public E remove(int index) {
            throw new UnsupportedOperationException();
        }
        public int indexOf(Object o)            {return list.indexOf(o);}
        public int lastIndexOf(Object o)        {return list.lastIndexOf(o);}
        public boolean addAll(int index, Collection<? extends E> c) {
            throw new UnsupportedOperationException();
        }

        @Override
        public void replaceAll(UnaryOperator<E> operator) {
            throw new UnsupportedOperationException();
        }
        @Override
        public void sort(Comparator<? super E> c) {
            throw new UnsupportedOperationException();
        }
```

可以看到，集合中的与查询相关的方法，比如get(),indexOf()等，都是调用的集合本身的方法，没有做变动。但如果是做修改的相关操作，比如set(),add(),remove()等，则是直接抛出UnsupportedOperationException异常，这就是为什么获取的列表是只读的原因。

刚刚提到会根据是否实现RandomAccess接口，来构造UnmodifiableList或UnmodifiableRandomAccessList对象，那我们来看看这两个类的区别

UnmodifiableRandomAccessList：

```java
static class UnmodifiableRandomAccessList<E> extends UnmodifiableList<E>
    implements RandomAccess
{
    UnmodifiableRandomAccessList(List<? extends E> list) {
        super(list);
    }
    public List<E> subList(int fromIndex, int toIndex) {
        return new UnmodifiableRandomAccessList<>(
            list.subList(fromIndex, toIndex));
    }
    private static final long serialVersionUID = -2542308836966382001L;
    private Object writeReplace() {
        return new UnmodifiableList<>(list);
    }
}
```

UnmodifiableRandomAccessList是UnmodifiableList的子类，并且实现了RandomAccess接口，代表支持快速随机访问。

还单独实现了writeReplace()方法，这个方法主要是用于对象序列化时，将UnmodifiableRandomAccessList转为UnmodifiableList对象。

为什么这样做呢？这是因为RandomAccess是jdk1.4引入的，因此1.4之前还没有UnmodifiableRandomAccessList这个对象。说到底，就是为了兼容。

# **实际使用**

我们了解了Collections.unmodifiableList(List<T>list)可以生成一个只读列表后，那么可以稍微改造下上面所说的学生选课的类，改造后的代码如下：

```
public class StudentRefactor {
    // 学生姓名
    private String name;
    // 学生的课程
    private final List<String> courses;
    public StudentRefactor(String name, List<String> courses) {
        this.name = name;
        this.courses = courses;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public List<String> getCourses() {
        // 获取只读集合,若对该集合执行修改操作，会报UnsupportedOperationException异常
        return Collections.unmodifiableList(courses);
    }
    // 提供唯一添加课程的入口，专一职责
    public void addCourses(String course) {
        this.courses.add(course);
    }
    public static void main(String[] args) {
        // 初始化学生课程
        StudentRefactor student = new StudentRefactor("张三", new ArrayList<String>() {{
            add("数学");
            add("语文");
            add("英语");
        }});
        System.out.println("当前课程数量为: " + student.getCourses().size());

        // 继续添加另外的课程
        student.addCourses("物理");
        student.addCourses("化学");
        System.out.println("当前课程数量为: " + student.getCourses().size());
    }
}
```

修改getCourses()方法，不再直接返回courses的引用，而是通过调用Collections.unmodifiableList方法生成一个课程集合的只读列表，如果继续使用getCourses().add("XXX")则会抛出异常。对应的，新增一个专门添加课程的方法addCourses()，在方法内对课程集合进行添加，单一职责，方法隔离。

demo地址： https://github.com/Phukety/study-demo/tree/master/src/main/java/com/phukety/demo/collections/unmodifiableList



