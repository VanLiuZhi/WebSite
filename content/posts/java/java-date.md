---
weight: 2202
title: "Java 时间处理"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Java 时间处理"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [Java, Note]
categories: [Java-MicroServices]

lightgallery: true

toc:
  auto: false
---

## 概述

java的日期处理，在旧版本中是存在很多问题的。也引申出了一些第三方类库 joda-time。 推荐使用java8的新API就行了

## simpleDateFormat 的线程安全问题

`static SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyyMMdd HH");`

当我们在一个工具类中定义simpleDateFormat之后，如果多个线程调用

线程1

`simpleDateFormat.setTimeZone(TimeZone.getTimeZone("+7"))`
`simpleDateFormat.format(new Date())`

线程2

`simpleDateFormat.format(new Date())` 

由于线程1修改了时区，导致线程2调用的时候也是按照线程1的时区来的，所以SimpleDateFormat有很严重的线程安全问题，除非每次都实例化一个新的对象

## LocalDate

LocalDate类实例是一个不可变对象，只包含日期，不含有时间信息，也不带时区信息

`LocalDate now = LocalDate.now()` 创建实例
`LocalDate.of(2020, 12, 12);` 通过of方法构造实例

## LocalTime

LocalTime只包含时间信息，不包含日期信息

`LocalTime time = LocalTime.now()` 或者调用 `LocalTime.parse("12:30:20")`

## LocalDateTime

LocalDate和LocalTime的结合体，既有时间信息，又有日期信息。可以通过LocalDate加时间或者LocalTime加日期构造LocalDateTime

```java
//今天当前时间
LocalDateTime ltime1 = LocalDateTime.now();

//指定年、月、日、时、分、秒
LocalDateTime time = LocalDateTime.of(2019, 12, 26, 12, 11, 34);

//LocalDate + 时间信息
LocalDateTime ltime2 = LocalDate.now().atTime(12,11,23);

//LocalTime + 日期信息
LocalDateTime ltime3 = LocalTime.now().atDate(LocalDate.now());
```

## Duration、Period

这个两个类是为了计算时间间隔的

Duration 表示秒/纳秒的时间间隔
Period 表示年、月、日时间间隔

```java
// 几天前的实现
LocalDate date1 = LocalDate.now();
LocalDate date2 = LocalDate.of(2019, 7, 27);
Period period = Period.between(date2, date1);

LocalTime now = LocalTime.now();
LocalTime of = LocalTime.of(15, 52, 1, 22);
Duration between = Duration.between(now, of);

// date1是2020年8月28，只会返回相差的天数1天，不是从2019年到2028年的天数。可以用获取月份，年份的方法来操作
System.out.println(period.getDays());
// 这个会计算全部的，也就是两个之间相差的秒数
System.out.println(between.getSeconds());
```

## TemporalAdjuster

一个内置的工具类，基于函数式的

```java
@FunctionInterface
public function TemporalAdjuster{
  Temporal adjustInto(Temporal temporal);
}
```

LocalDate、LoclaTime、LocalDateTime等都实现了Temporal接口

用法是 调用 with，传递TemporalAdjuster

```
dayOfWeekInMonth 返回同一个月中每周第几天

firstDayOfMonth 返回当月第一天

firstDayOfNextMonth 返回下月第一天

firstDayOfNextYear 返回下一年的第一天

firstDayOfYear 返回本年的第一天

firstInMonth 返回同一个月中第一个星期几

lastDayOfMonth 返回当月的最后一天

lastDayOfNextMonth 返回下月的最后一天

lastDayOfNextYear 返回下一年的最后一天

nextOrSame / previousOrSame 返回后一个/前一个给定的星期几，如果这个值满足条件，直接返回
```

## DateTimeFormatter

用于日期格式化操作，format()/parse()

```Java
//format 格式化
LocalDateTime dateTime = LocalDateTime.now();
String strDate1 = dateTime.format(DateTimeFormatter.BASIC_ISO_DATE);    // 20200828
String strDate2 = dateTime.format(DateTimeFormatter.ISO_LOCAL_DATE);    // 2020-08-28
String strDate3 = dateTime.format(DateTimeFormatter.ISO_LOCAL_TIME);    // 14:20:16.998
String strDate4 = dateTime.format(DateTimeFormatter.ofPattern("yyyy-MM-dd"));// 2020-08-28


//parse 解析
String strDate6 = "2020-08-28";
String strDate7 = "2020-08-28 12:30:05";

LocalDate date = LocalDate.parse(strDate6, DateTimeFormatter.ofPattern("yyyy-MM-dd"));
LocalDateTime dateTime1 = LocalDateTime.parse(strDate7, DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
```

## 常用操作

```java
// 获取今天12：00：00时间戳
LocalDate.now().atTime(12,0,0).toInstant(ZoneOffset.of("+8")).getEpochSecond());

//5天前
LocalDate.now().minusDays(5);

//1天后
LocalDate.now().plusDays(1);

//2小时前
LocalTime.now().minusHours(2);

//5分钟后
LocalTime.now().plusMinutes(5);



//前端传递过来的字符串20191212，后端需要 2019-12-12
//除了replace()，更优雅的方式
DateTimeFormatter formatter = DateTimeFormatter.BASIC_ISO_DATE;
LocalDate formatted = LocalDate.parse("20191212",formatter);
//2019-12-12
System.out.println(formatted);

//前端传递过来的字符串2019-12-12，后端需要 20191212
LocalDate.parse("2019-12-12").format(DateTimeFormatter.BASIC_ISO_DATE));

//前端传递过来的字符串2019年12月12日，后端需要 2019-12-12
System.out.println(LocalDate.parse("2019年12月12日",DateTimeFormatter.ofPattern("yyyy年MM月dd日")));



//场景中需要使用距当前最近的周一
//返回上一个周一，如果今天是周一则返回今天
LocalDate date1 =LocalDate.now().with(previousOrSame(DayOfWeek.MONDAY));

//下一个周一/当天
LocalDate date1 =LocalDate.now().with(nextOrSame(DayOfWeek.MONDAY));

//本月最后一天
System.out.println(LocalDate.now().with(TemporalAdjusters.lastDayOfMonth()));

//明年第一天
System.out.println(LocalDate.now().with(TemporalAdjusters.firstDayOfNextYear()));
```

## 对象转换

```java
//Date转LocalDateTime
Date date = new Date();
LocalDateTime time1 = LocalDateTime.ofInstant(date.toInstant(),ZoneOffset.of("+8"))				
//Date转LocalDate
LocalDate date1 = LocalDateTime.ofInstant(date.toInstant(),ZoneOffset.of("+8")).toLocalate()

//LocalDate无法直接转Date，因为LocalDate不含时间信息

//LocalDateTime转Date
LocalDateTime localDateTime3 = LocalDateTime.now();
Instant instant3 = LocalDateTime3.atZone(ZoneId.systemDefault()).toInstant();
Date date3 = Date.from(instant);
```