---
weight: 4400
title: "Java stream"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Java stream"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [Java, Note]
categories: [Java-MicroServices]

lightgallery: true

toc:
  auto: false
---

1. 使用stream 把 Arrays.asList() 的数组转换成 ArrayList

Integer [] myArray = { 1, 2, 3 };
List myList = Arrays.stream(myArray).collect(Collectors.toList());
// 基本类型也可以实现转换（依赖boxed的装箱操作）
int [] myArray2 = { 1, 2, 3 };
List myList = Arrays.stream(myArray2).boxed().collect(Collectors.toList());

2. 去重，去除重复对象（每个属性的值都一样的），需要注意的是要先重写对象User的equals和hashCode方法

List<User> distinctList = list.stream().distinct().collect(Collectors.toList());

3. 排序，按id升续排列，如果要降续则改成：(a, b) -> b.getId() - a.getId(); a和b都是变量名（可以按自己意愿取名字），都是list中的对象的实例

List<User> sortList = list.stream().sorted((a, b) -> a.getId() - b.getId()).collect(Collectors.toList());
 
4. 过滤，按照自己的需求来筛选list中的数据，比如我筛选出不及格的（小于60分）的人,t为实例

List<User> filterList = list.stream().filter(t -> t.getScore() < 60).collect(Collectors.toList());
 
5. map, 提取对象中的某一元素，例子中我取的是每个人的name，注意list中类型对应，如果取的是id或者班级，就应该是integer类型

List<String> mapList = list.stream().map(t -> t.getName()).collect(Collectors.toList());
 
6. 统计，统计所有人分数的和, 主要我设置的分数属性是double类型的，所以用mapToDouble，如果是int类型的，则需要用mapToInt

double sum = list.stream().mapToDouble(t -> t.getScore()).sum();
int count = list.stream().mapToInt(t -> t.getId()).sum();
 
7. 分组， 按照字段中某个属性将list分组

Map<Integer, List<User>> map = list.stream().collect(Collectors.groupingBy(t -> t.getGrade()));

    System.out.println("按年级分组"+map);
    /*然后再对map处理，这样就方便取出自己要的数据*/
    for(Map.Entry<Integer, List<User>> entry : map.entrySet()){
        System.out.println("key:"+entry.getKey());
        System.out.println("value:"+entry.getValue());
    }
 
8. 多重分组，先按年级分组，再按班级分组
    
Map<Integer/*年级id*/, Map<Integer/*班级id*/, List<User>>> groupMap = list.stream().collect(Collectors.groupingBy(t -> t.getGrade(), Collectors.groupingBy(t -> t.getClasses())));
    
    System.out.println("按照年级再按班级分组："+groupMap);
    System.out.println("取出一年级一班的list："+groupMap.get(1).get(1));

9. 多重分组，一般多重分组后都是为了统计，比如说统计每个年级，每个班的总分数
    
Map<Integer/*年级id*/, Map<Integer/*班级id*/, Double>> sumMap = list.stream().collect(Collectors.groupingBy(t -> t.getGrade(), Collectors.groupingBy(t -> t.getClasses(), Collectors.summingDouble(t -> t.getScore()))));

    System.out.println(sumMap);
    System.out.println("取出一年级一班的总分："+sumMap.get(1).get(1));

10. stream是链式的，这些功能是可以一起使用的，例如：计算每个年级每个班的及格人数

Map<Integer/*年级*/, Map<Integer/*班级*/, Long/*人数*/>> integerMap = list.stream().filter(t -> t.getScore() >= 60).collect(Collectors.groupingBy(t -> t.getGrade(), Collectors.groupingBy(t -> t.getClasses(), Collectors.counting())));

    System.out.println("取出一年级一班及格人数："+integerMap.get(1).get(1));


分组补充：

https://blog.csdn.net/daobuxinzi/article/details/100190366
https://blog.csdn.net/u014231523/article/details/102535902

一般分组都是 Map 的，但是可以对结果做处理，比如重载集合收集器，使用自定义的结果集

11. collectingAndThen --根据对象的属性进行去重操作
先进行结果集的收集，然后将收集到的结果集进行下一步的处理，这里做个记录，具体使用参考文档
