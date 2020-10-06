---
weight: 1000
title: "Java8 new语法特性"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Java8 new语法特性"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Java, Note]
categories: [Java-MicroServices]

lightgallery: true

toc:
  auto: false
---

Java8 new语法特性

![image](/images/Java/Java-base.jpg)

## 总结

发现很多小伙伴还没有用上Java的新特性，这其实也和Java比较标准化的原因有关吧，稳定才是企业的诉求，像lambda，函数式等其它语言都过的东西，却很少在Java中看见，不过会还是要会的，跟上时代的步伐，尤其是函数式编程的思想

## Lambda表达式和方法引用


## Stream流对象

流分类:

1. 顺序流: 按照顺序对集合中的元素进行处理
2. 并行流: 使用多线程同时对集合中多个元素进行处理(在使用并行流的时候就要注意线程安全的问题)

流的过程: 元素流在管道中经过中间操作（intermediate operation）的处理，最后由终端操作 (terminal operation) 得到前面处理的结果。

中间操作(intermediate operation):  中间操作会产生另一个流，(流是一种惰性操作，所有对源数据的计算只在终止操作被初始化的时候才会执行)，而且中间操作还分无状态操作和有状态操作两种:

1. 无状态操作 : 在处理流中的元素时，会对当前的元素进行单独处理。 (例如:过滤操作).
2. 有状态操作 : 某个元素的处理可能依赖于其他元素.( 例如:查找最小值，最大值，和排序 ).


终止操作 (terminal operation):消费 Stream 流，并且会产生一个结果 . `如果一个 Stream 流被消费过了，那它就不能被重用的`

Stream流一般的执行过程可概括为:

1. 源(Stream)
2. 零个或多个中间操作(intermediate operation)
3. 终止操作 （到这一步才会执行整个stream pipeline计算） (terminal operation)

源的创建方式

1. 使用Collection下的 stream() 和 parallelStream() 方法
2. 使用Stream中的静态方法：of()

// 顺序流
Stream< String> stream = createStream.stream();
// 并行流
Stream< String> parallelStream = createStream.parallelStream();
// of()方法创建
Stream< String> stringStream = Stream.of(createStream.toArray(new String[createStream.size()]));

Intermediate操作

中间操作包括map (mapToInt, flatMap 等)、 filter、 distinct、 sorted、 peek、 limit、 skip、 parallel、 sequential、 unordered等.

常用操作解析:

filter : 筛选符合条件的元素后重新生成一个新的流。
map : 接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素。
flatMap:   接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流。
distinct: 去重操作，将 Stream 流中的元素去重后，返回一个新的流。
sorted: 产生一个自然顺序排序或者指定排序条件的新流。
skip:跳过n元素，配合limit(n)可实现分页
peek: 生成一个包含原Stream的所有元素的新Stream，同时会提供一个消费函数（Consumer实例），新Stream每个元素被消费的时候都会执行给定的消费函数(一般用于重赋值那些)；
limit:  对一个Stream进行截断操作，获取其前N个元素，如果原Stream中包含的元素个数小于N，那就获取其所有的元素；

terminal操作:

终止操作包括:forEach、 forEachOrdered、 toArray、 reduce、 collect、 min、 max、 count、 anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 iterator等

常用操作解析:

forEach: 遍历了流中的元素。(终端操作)
collect: 接收一个Collector实例，将流中元素收集成另外一个数据结构
max:获得流中最大值，比较器可以由自己定义。(终端操作)
min: 获得流中最小值，比较器可以由自己定义。(终端操作)
anyMatch : 判断 Stream 流中是否有任何符合要求的元素，如果有则返回 ture,没有返回 false。（终端操作）



## Optional

1、什么是Optional？

一句话概括: 它是一个容器，用来包装对象，解决NullPointerException异常的

2、如何创建Optional对象？或者说如何用Optional来包装对象或null值？

- static Optional empty() ：用来创建一个空的Optional

- static Optional of(T value) ：用来创建一个非空的Optional

- static Optional ofNullable(T value) ：用来创建一个可能是空，也可能非空的Optional

其实上面这三个方法，看一下源码就很清晰了，比如of

```java
// 方法
public static <T> Optional<T> of(T value) {
        return new Optional<>(value);
    }
// 构造器
private Optional(T value) {
        this.value = Objects.requireNonNull(value);
    }
// Objects对象的requireNonNull方法
public static <T> T requireNonNull(T obj) {
        if (obj == null)
            throw new NullPointerException();
        return obj;
    }
```

of方法创建Optional对象，并对传入值做NullPointerException异常判断，所以，当你传递一个null值的时候，就会触发异常了

再看看empty方法

```java
// 方法
public static<T> Optional<T> empty() {
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }
// 静态常量
private static final Optional<?> EMPTY = new Optional<>();

// 常量
private final T value;

// 构造器
private Optional() {
    this.value = null;
    }
```

一开始就定义了一个`Optional<?> EMPTY`的对象，并且构造函数使用默认的，value为空。empty只是做了泛型的转换

剩下的ofNullable就更简单了，通过传递的值决定使用of还是empty

```java
public static <T> Optional<T> ofNullable(T value) {
        return value == null ? empty() : of(value);
    }
```

Optional类的源码还是比较简单的，代码很少，通过成员变量和构造方法的解读，相信你已经理解了of，empty，ofNullable

3、如何使用Optional类？

最正确的做法就是对需要做NullPointerException异常判断的对象，把它包装成Optional类，然后指明对象不存在的时候，应该怎么做，下面来看一下几种常见的用法

1. orElse

```java
public class OptionalTest {
    public static void main(String[] args) {
        HashMap<Integer, User> userHashMap = new HashMap<>();
        userHashMap.put(1, new User(1, "Xiao Ming"));
        userHashMap.put(2, new User(2, "Xiao Zhi"));
        userHashMap.put(3, null);

        UserUtil userUtil = new UserUtil(userHashMap); // 这个工具类只是为了填充数据的，不要在意这些细节，getUserByUserId方法能返回User对象

        // 包装了User对象，并且使用orElse指明了对象不存在的时候应该返回指定的值，也就是new User(1, "Xiao Bai")
        User user = Optional
                .ofNullable(UserUtil.getUserByUserId(2))
                .orElse(new User(1, "Xiao Bai"));

    }
}

// 工具类，随便写的，通过hashMap模拟查询用户
public class User {
    public Integer id;
    public String name;

    public User(Integer id, String name) {
        this.id = id;
        this.name = name;
    }
}

class UserUtil {

    public static HashMap<Integer, User> hashMap;

    UserUtil(HashMap<Integer, User> hashMap) {
        UserUtil.hashMap = hashMap;
    }

    public static User getUserByUserId(Integer id) {
        User user = hashMap.get(id);
        System.out.println(user);
        return user;
    }
}
```

2. orElseGet

和 orElse 不同，它的参数接受一个lambda表达式

```java
User user = Optional
                .ofNullable(UserUtil.getUserByUserId(2))
                .orElseGet(() -> new User(1, "Xiao Bai"));
```

3. orElseThrow

同理，传递一个lambda表达式异常(注意啊，方法是限定了参数的，触发异常就用orElseThrow，不能用orElseGet)

```java
User user = Optional
                .ofNullable(UserUtil.getUserByUserId(2))
                .orElseThrow(()-> new AssertionError("AssertionError"));
```

4. map

调用map后，如果当前 Optional 为 Optional.empty，则依旧返回 Optional.empty；否则返回一个新的 Optional，该 Optional 包含的是：函数 mapper 在以 value 作为输入时的输出值

比如下面的例子，第一次调用map后，获取的是name，传递给下一个map的值相当于Optional.ofNullable(name)

```java
String user = Optional.ofNullable(UserUtil.getUserByUserId(1))
                .map(user1 -> user1.name)
                .map(String::toLowerCase)
                .orElse("123");
        System.out.println(user);

// 方法
public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Optional.ofNullable(mapper.apply(value));
        }
    }
```

这个操作可以用在对对象做多次操作的场景下，并且保证为空的时候返回指定的值，比如先获取用户的名称，再获取用户的电话，做一下判断后，再通过电话查询到其它的数据，然后继续...，最后如果某一环节出现异常，那就返回orElse定义的对象

5. flatMap

flatMap 方法与 map 方法的区别在于，map 方法参数中的函数 mapper 输出的是值，然后 map 方法会使用 Optional.ofNullable 将其包装为 Optional；而 flatMap 要求参数中的函数 mapper 输出的就是 Optional

即 `.flatMap(user -> Optional.of(user.name()))`

6. filter

都是差不多的套路，这次我们先看源码，可以发现同样是接受lambda表达式，并且要是一个Boolean返回值的，如果本次操作的optional是empty的，就返回本身

```java
public Optional<T> filter(Predicate<? super T> predicate) {
        Objects.requireNonNull(predicate);
        if (!isPresent())
            return this;
        else
            return predicate.test(value) ? this : empty();
    }

// 函数式接口部分代码，可以看到test是要求返回Boolean类型的

@FunctionalInterface
public interface Predicate<T> {

    /**
     * Evaluates this predicate on the given argument.
     *
     * @param t the input argument
     * @return {@code true} if the input argument matches the predicate,
     * otherwise {@code false}
     */
    boolean test(T t);
}

// 例子
String name = Optional.ofNullable(UserUtil.getUserByUserId(1))
                .filter(user -> user.id == 2)
                .map(user -> user.name)
                .orElse("abc");
        System.out.println(name);
```

通过观察源码，我们可以看到，很多为空的都会返回empty，这让各个操作都能够互相调用

## 总结

Optional是一个比较简单的类，推荐直接阅读源码，通过简单的包装，很优雅的解决的空指针问题，以前谷歌Guava库就实现了，后面Java8正式把规范加到 `java.util.Optional` 类中

PS: 如果你看源码有压力，快去补习一下lambda，函数式编程

另外，上面的做法是对于程序内的，如果是web开发，参数校验，请使用Hibernate-Validator即可，作为一个合格的后端，我不会让前端挑刺的