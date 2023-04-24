---
title: Time
tags: [java, util, time]
sidebar_label: Time
sidebar_position: 1
---

# 第一代时间类库

## java.util.Date

`java.util.Date` 不能直接表示日期，以毫秒的精度表示时间。年份的起始年为1900年，月份起始年为0，如下例子，2023年用123表示，12月用11表示。时间的默认时区是JVM默认的CST。

```java
Date now = new Date();// 等价 new Date(System.currentTimeMillis());
Date date = new Date(123, 11, 10); // 2023-12-10
date.setHours(23);
date.setMinutes(59);
long millisSeconds = date.getTime();
System.out.println(date);// Sun Dec 10 23:59:00 CST 2023
```

如下是 `java.util.Date` 的其中一个构造方法，可以看出为什么年要减去1900，月要减去1：

```java
public class Date {
    // ...
    @Deprecated
    public Date(int year, int month, int date, int hrs, int min, int sec) {
        int y = year + 1900;
        // month is 0-based. So we have to normalize month to support Long.MAX_VALUE.
        if (month >= 12) {
            y += month / 12;
            month %= 12;
        } else if (month < 0) {
            y += CalendarUtils.floorDivide(month, 12);
            month = CalendarUtils.mod(month, 12);
        }
        BaseCalendar cal = getCalendarSystem(y);
        cdate = (BaseCalendar.Date) cal.newCalendarDate(TimeZone.getDefaultRef());
        cdate.setNormalizedDate(y, month + 1, date).setTimeOfDay(hrs, min, sec, 0);
        getTimeImpl();
        cdate = null;
    }
}
```

## java.sql.Date

`java.sql.Date` 继承了 `java.util.Date` 类，并重写了构造方法，把时分秒毫秒等全部设置为0，精度只能表示到天，所以一般来说**基本不用该类**。

```java
/**
 * <P>A thin wrapper around a millisecond value that allows
 * JDBC to identify this as an SQL <code>DATE</code> value.  A
 * milliseconds value represents the number of milliseconds that
 * have passed since January 1, 1970 00:00:00.000 GMT.
 * <p>
 * To conform with the definition of SQL <code>DATE</code>, the
 * millisecond values wrapped by a <code>java.sql.Date</code> instance
 * must be 'normalized' by setting the
 * hours, minutes, seconds, and milliseconds to zero in the particular
 * time zone with which the instance is associated.
 */
public class Date extends java.util.Date {
    @Deprecated
    public Date(int year, int month, int day) {
        super(year, month, day);
    }

    public Date(long date) {
        // If the millisecond date value contains time info, mask it out.
        super(date);
    }
```

# 第二代时间类库

## java.util.Calendar

`java.util.Calendar` 类中的年和月可以直接表示，不像 `java.util.Date` 那样需要减去1900年和1个月。同时该类是一个可修改的类，并不是一个稳定的时间。此时的JDK保留了 `java.util.Date` 和 `java.util.Calendar` 两个类，只用一个类有时候功能是残缺的。

```java
// 获取日历类，获取当前时间
Calendar calendar = Calendar.getInstance();
// 转换为Date类
Date time = calendar.getTime();
// 输出 格式化后的 时间
DateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
System.out.println(dateFormat.format(time));

// 变更时间，由当前时间变更为 2025年12月12日
calendar.set(2025,12,12);
Date time2 = calendar.getTime();
System.out.println("我改时间了：");
System.out.println(dateFormat.format(time2));
```

## java.text.DateFormat

`DateFormat` 是时间格式化的抽象类，我们经常使用它的子类 `SimpleDateFormat` 来格式化和解析日期与时间。但是它也有一个臭名昭著的问题，非线程安全的，多线程如果尝试使用同一个formatter解析日期，可能会得到无法预期的结果。

```java
DateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
// 开5个线程，方便截图
for (int i = 0; i < 5; i++) {
    // Lambda表达式创建线程
    new Thread(() -> {
        try {
            // 将字符串时间转换为Date日期对象
            Date date = dateFormat.parse("2022-12-10");
            System.out.println(Thread.currentThread().getName() + "====>" + date);
        } catch (ParseException e) {
            throw new RuntimeException(e);
        }
    }).start();
}
```

总结：
* 早期的日期时间类库设计不统一，分布在不同的包中。
* 日期设计不合理，`Date` 中的年从 `1900` 年算起，包括 `Calendar` 中的月份从 `0` 开始。
*  `Date` 和 `Calendar` 的时间可变，会造成很大的安全隐患。
*  `DateFormat` 线程不安全，多线程场景需要手动加锁解决。

# 第三代时间类库

前面两代的时间类库的缺陷导致了很多用户转向了第三方类库，比如Joda-Time等。所以在java 8开始在 `java.time` 包下增加了很多新类，优化了这些缺陷，并增加了一些新的功能。

java 8中常见的日期时间类有：

* LocalDate：表示日期，精度到天，无时区属性。
* LocalTime：表示时间，无时区属性。
* LocalDateTime：表示日期和时间，无时区属性。
* Instant：表示时间戳，也就是1970年1月1日0点【格林威治时间】到现在的毫秒数。
* ZonedDateTime：表示日期和时间，有时区属性。
* Duration：表示一段时间。
* Period：表示一段日期，基于日历系统的日期区间。
* DateTimeFormatter：日期时间格式化器，线程安全。
* ZoneId/ZoneOff：表示时区。
* TemporalAdjuster：支持调整日期。

## LocalDate

该类表示的是不可修改的日期，精度到天，不包含时间和时区信息。

```java
LocalDate now = LocalDate.now();
LocalDate localDate = LocalDate.of(2022, 12, 31); // 2022-12-31
int year = localDate.getYear();// 2022
Month month = localDate.getMonth();// Month枚举
int monthValue = localDate.getMonthValue();// 12
int dayOfYear = localDate.getDayOfYear(); // 365
int dayOfMonth = localDate.getDayOfMonth(); // 31
DayOfWeek dayOfWeek = localDate.getDayOfWeek(); // DayOfWeek枚举 SATURDAY
boolean leapYear = localDate.isLeapYear(); // true 是否闰年

LocalDate date = LocalDate.ofYearDay(2022, 32);// 2022-02-01
int year2 = date.get(ChronoField.YEAR);// 2022
int month2 = date.get(ChronoField.MONTH_OF_YEAR);// 2
int dayOfMonth2 = date.get(ChronoField.DAY_OF_MONTH); // 1
int dayOfYear2 = date.get(ChronoField.DAY_OF_YEAR);// 32
int dayOfWeek2 = date.get(ChronoField.DAY_OF_WEEK);//2

LocalDate localDate2 = LocalDate.parse("2022-12-31");
```

一些常用运算样例：

```java
// 日期的加法运算
LocalDate localDate1 = LocalDate.now();
// 1、添加15天
LocalDate localDate2 = localDate1.plus(15, ChronoUnit.DAYS);
// 2、添加15天
LocalDate localDate3 = localDate1.plus(Period.ofDays(15));
// 3、增加15天
LocalDate localDate4 = localDate1.plusDays(15);
// 4、增加15周
LocalDate localDate5 = localDate1.plusWeeks(15);
// 5、增加15月
LocalDate localDate6 = localDate1.plusMonths(15);
// 6、增加15年
LocalDate localDate7 = localDate1.plusYears(15);

// 日期的减法运算
LocalDate localDate1 = LocalDate.now();
// 1、减去 15天
LocalDate localDate2 = localDate1.minus(15, ChronoUnit.DAYS);
// 2、减去15天
LocalDate localDate3 = localDate1.minus(Period.ofDays(15));
// 3、减去15天
LocalDate localDate4 = localDate1.minusDays(15);
// 4、减去15周
LocalDate localDate5 = localDate1.minusWeeks(15);
// 5、减去15月
LocalDate localDate6 = localDate1.minusMonths(15);
// 6、减去15年
LocalDate localDate7 = localDate1.minusYears(15);

// 日期的调整方法
LocalDate localDate1 = LocalDate.now();
// 1、设置为周二
LocalDate localDate2 = localDate1.with(DayOfWeek.THURSDAY);
// 2、设置到2050年
LocalDate localDate3 = localDate1.with(ChronoField.YEAR, 2050);
// 3、设置到10号
LocalDate localDate4 = localDate1.withDayOfMonth(10);
// 4、设置到一年的第364天：有效值为 1 到 365，闰年的有效值为 1 到 366
LocalDate localDate5 = localDate1.withDayOfYear(364);
// 5、设置到10月份: 有效值是1到12
LocalDate localDate6 = localDate1.withMonth(10);
// 6、设置到2050年
LocalDate localDate7 = localDate1.withYear(2050);


// 日期的比较运算
LocalDate localDate1 = LocalDate.parse("2021-02-14");
LocalDate localDate2 = LocalDate.parse("2022-05-20");
localDate1.isAfter(localDate2);//false
localDate1.isBefore(localDate2);//true
localDate1.isEqual(localDate2);// false
localDate1.isLeapYear();// false
localDate1.isSupported(ChronoField.DAY_OF_MONTH);// true
localDate1.isSupported(ChronoUnit.HOURS);// false，不支持小时字段
localDate1.equals(localDate2);// false
localDate1.compareTo(localDate2);// 1:大于，0：等于，-1：小于
localDate1.lengthOfMonth();// 28天
localDate1.lengthOfYear();// 365天
```

## LocalTime

该类表示的是时间，精度到纳秒，不包含日期和时区信息。

```java
LocalTime now = LocalTime.now();
LocalTime time = LocalTime.parse("13:14:52");
LocalTime localTime = LocalTime.of(23, 59, 59);//23:59:59
int hour = localTime.getHour();//23
int minute = localTime.getMinute();//59
int second = localTime.getSecond();//59
int nano = localTime.getNano();//0
int minuteOfHour = localTime.get(ChronoField.MINUTE_OF_HOUR);//59
```

## LocalDateTime

该类的功能是 `LocalDate` 和 `LocalTime` 的合并。

```java
LocalDate date = LocalDate.now();
LocalTime time = LocalTime.now();
LocalDateTime dateTime1 = LocalDateTime.of(date, time);
LocalDateTime dateTime2 = date.atTime(time);
LocalDateTime dateTime3 = time.atDate(date);
LocalDateTime dateTime4 = LocalDateTime.of(2022, 12, 31, 13, 14, 52);
LocalDateTime now = LocalDateTime.now();
LocalDate date2 = dateTime4.toLocalDate();
LocalTime time2 = dateTime4.toLocalTime();
```

## Instant

该类是java 8补充的表示时间戳的类，不同于 `System.currentTimeMillis()` 获取到的是纳秒。以Unix元年时间，也就是1970年1月1日0点开始所经历的时间。

```java
Instant instant1 = Instant.now();

// 1.Instant对象的一些属性
long epochSecond = instant1.getEpochSecond();//纪元秒
long millis = System.currentTimeMillis();//时间戳
long epochMilli = instant1.toEpochMilli();//毫秒
int nano = instant1.getNano();//纳秒

// 2.获取指定时间的Instant对象
Instant instant2 = Instant.ofEpochSecond(100);
long epochSecond2 = instant2.getEpochSecond();
long epochMilli2 = instant2.toEpochMilli();
int nano2 = instant2.getNano();

//3.指定时间戳创建 带时区的日期时间对象 ZoneDateTime
Instant instant3= Instant.ofEpochSecond(1670679915);// 2022-12-10 21:45:15
ZonedDateTime zonedDateTime = instant3.atZone(ZoneId.of("Asia/Shanghai"));

// 4.指定时间戳创建  默认时区的日期时间对象 LocalDateTime
Instant instant4 = Instant.ofEpochSecond(1670679915); // 2022-12-10 21:45:15
LocalDateTime localDateTime = LocalDateTime.ofInstant(instant4, ZoneId.systemDefault());
```

Instant的设计初衷是为了便于机器使用。它包含的是由秒及纳秒所构成的数字。所以，它无法处理那些我们非常容易理解的时间单位。

## Period和Duration

* `Period` ：用于计算日期的间隔，对应使用 `LocalDate` ，它们的作用范围域都是日期(年/月/日)。
*  `Duration` ：用于计算时间的间隔，对应使用 `Instant`、`LocalTime`、`LocalDateTime`，它们的作用范围域都是时间(天/时/分/秒/毫秒/纳秒)。

`Period` 类的常见用法如下：
```java
// 传入年月日创建Period对象
Period p = Period.of(2022, 12, 31);
// 传入年构造
p = Period.ofYears(2022);
// 传入月份构造
p = Period.ofMonths(11);
// 传入周构建，1周为7天
p = Period.ofWeeks(1);
// 传入天数构建
p = Period.ofDays(12);
// 传入负数日期
Period period = Period.of(2022,-12,6);
// 判断日期中是否包含负数，有返回true
boolean negative = period.isNegative();

int years = p.getYears();
int months = p.getMonths();
int days = p.getDays();

// 计算日期差
LocalDate date1 = LocalDate.of(2021,11,11);
LocalDate date2 = LocalDate.of(2022,12,12);
Period period2 = Period.between(date1, date2);
long years = ChronoUnit.YEARS.between(date1, date2);//获取相差几年 结果：1
long months = ChronoUnit.MONTHS.between(date1, date2);// 获取相差几月  结果：13
long days = ChronoUnit.DAYS.between(date1, date2);// 获取相差几天  结果：396
long days2 = date2.toEpochDay() - date1.toEpochDay();// 通过toEpochDay将时间转换为距离1970年1月1日0时的时间，相减获取相差天数
```

`Duration` 类的常见用法如下：
```java
// 创建两个时间
LocalDateTime start = LocalDateTime.of(2021,11,11,00,00,00);
LocalDateTime end = LocalDateTime.of(2022,12,12,12,12,12);

// between的用法是end-start的时间，若start的时间大于end的时间，则所有的值是负的
Duration duration = Duration.between(start, end);// PT9516H12M12S-->9516小时12分12秒

System.out.println("相差的天数="+duration.toDays());
System.out.println("相差的小时="+ duration.toHours());
System.out.println("相差的分钟="+duration.toMinutes());
// 获取秒，在JDK9之上才可调用，JDK8中为私有方法
System.out.println("相差的秒数="+duration.toSeconds());
// JDK8可以调用 getSeconds获取秒
System.out.println("相差的秒数="+duration.getSeconds());
System.out.println("相差的毫秒="+duration.toMillis());
System.out.println("相差的纳秒="+duration.toNanos());

//isNegative返回Duration实例对象是否为负
System.out.println(Duration.between(start, end).isNegative());//false  end-start为正，所以此处返回false
System.out.println(Duration.between(end, start).isNegative());//true   start-end为负，所以此处返回true
System.out.println(Duration.between(start, start).isNegative());//false start-start为0，所以此处为false

// 计算时间间隔
LocalTime start = LocalTime.of(11,11,10);
LocalTime end = LocalTime.of(12,12,30);
Duration duration = Duration.between(start, end);// 结果：两个时间相差：3680秒，相差：1小时，相差：61分钟
long hour = ChronoUnit.HOURS.between(start , end );// 计算小时: 1
long minute = ChronoUnit.MINUTES.between(start , end );// 计算分钟：61
long seconds = ChronoUnit.SECONDS.between(start , end );// 计算秒: 3680
int time = end.toSecondOfDay() - start.toSecondOfDay();// 结果：3680

// 计算时间戳间隔
long todayTimeMillis = System.currentTimeMillis();// 获取当前时间
long yesterdayTimeMillis = todayTimeMillis - 24 * 60 * 60 * 1000;// 设置昨天时间戳，可以是任意两个时间戳
Instant yesterday = Instant.ofEpochMilli(yesterdayTimeMillis);//通过Instant类，可以直接将毫秒值转换为Instant对象
Instant today = Instant.ofEpochMilli(todayTimeMillis);
Duration duration = Duration.between(yesterday, today);//天数 = 1
```

## TemporalAdjuster

`TemporalAdjuster` 类是调整 `Temporal` 对象的策略。 `LocalDate` 和 `LocalTime` 等都是 `Temporal` 的实现类。

`TemporalAdjuster` 是函数接口，仅有一个带 `Temporal` 对象参数的抽象方法 `adjustInto()`。在 `TemporalAdjusters` 类中有很多预定义的实现。常见的用法如下：

```java
LocalDateTime now = LocalDateTime.now();
now.with(TemporalAdjusters.firstDayOfMonth());//获取当月第一天
now.with(TemporalAdjusters.firstDayOfNextMonth());//获取下月第一天
now.with(TemporalAdjusters.firstDayOfNextYear());//获取明年第一天
now.with(TemporalAdjusters.firstDayOfYear());//获取本年第一天
now.with(TemporalAdjusters.lastDayOfMonth());//获取当月最后一天
now.with(TemporalAdjusters.lastDayOfYear());//获取本年最后一天
now.with(TemporalAdjusters.dayOfWeekInMonth(3, DayOfWeek.FRIDAY));//获取当月第三周星期五
now.with(TemporalAdjusters.previous(DayOfWeek.MONDAY));//获取上周一
now.with(TemporalAdjusters.next(DayOfWeek.SUNDAY));//获取下周日
```

## 时间格式化

Java 8 的 `java.time.format` 包中提供了 `DateTimeFormatter` 和 `DateTimeFormatterBuilder` 来以不同的方式格式化日期、时间或两者。

`DateTimeFormatter` 具有可直接用于解析字符序列的内置格式，且是线程安全的。

```java
LocalDate localDate = LocalDate.now();
// 通过 LocalDate 的 format方法根据传入的 DateTimeFormatter类中的常量【也就是时间格式】进行格式化
String format1 = localDate.format(DateTimeFormatter.ISO_LOCAL_DATE);// 2022-12-11
String format2 = localDate.format(DateTimeFormatter.BASIC_ISO_DATE);// 20221211

LocalDate localDate2 = LocalDate.parse("20220520",DateTimeFormatter.BASIC_ISO_DATE);// 2022-05-20
```

`DateTimeFormatterBuilder` 提供了更复杂的格式器，可以自定义格式器。另外，它还提供了非常强大的解析功能，比如区分大小写的解析、柔性解析（允许解析器使用启发式的机制去解析输入，不精确地匹配指定的模式）、填充，所有的格式化器都是用 `DateTimeFormatterBuilder` 创建的，可以通过 `appendValue` 、 `appendLiteral` 和 `appendText` 等，用于生成一个格式化器。

```java
// 创建对象
DateTimeFormatterBuilder builder = new DateTimeFormatterBuilder();
// 设置格式化
DateTimeFormatter formatter = builder.appendLiteral("今天是:")
    .appendValue(ChronoField.YEAR)
    .appendLiteral("年,")
    .appendValue(ChronoField.MONTH_OF_YEAR)
    .appendLiteral("月,")
    .appendValue(ChronoField.DAY_OF_MONTH)
    .appendLiteral("日,周")
    .appendValue(ChronoField.DAY_OF_WEEK)
    .toFormatter();
LocalDateTime dateTime  = LocalDateTime.now();
String str =  dateTime.format(formatter);// 今天是:2022年,12月,11日,周7
```

## 时区

之前的日期和时间都不包含时区信息。时区的处理是新版日期和时间API新增的重要功能，使用新版日期和时间API时区的处理被极大地简化。

新的 `java.time.ZoneId` 类是老版 `java.util.TimeZone` 的替代品。它的设计目标就是要让你无需为时区处理的复杂和繁琐而操心。跟其他日期和时间类一样，`ZoneId` 类也是无法修改的。时区是按照一定的规则将区域划分成的标准时间相同的区间。在 `ZoneRules` 这个类中包含了40个这样的实例。你可以简单地通过调用 `ZoneId` 的`getRules()` 得到指定时区的规则。每个特定的 `ZoneId` 对象都由一个地区ID标识。

```java
// 创建时区对象
ZoneId zoneId = ZoneId.of("Asia/Shanghai");
```

地区ID都为{区域}/{城市}的格式，这些地区集合的设定都由英特网编号分配机构（IANA）的时区数据库提供。你可以通过Java 8的新方法 `toZoneId` 将一个老的时区对象转换为 `ZoneId` 。

```java
// 通过TimeZone的toZoneId转换为新的时区对象
ZoneId zoneId1 = TimeZone.getDefault().toZoneId();
```

一旦得到 `ZoneId` 对象，就可以将它与 `LocalDate` 、`LocalDateTime` 或者是 `Instant` 对象整合起来，构造为一个 `ZonedDateTime` 实例，它代表了相对于指定时区的时间点。

```java
// 创建时区
ZoneId zoneId = ZoneId.of("Asia/Shanghai");
LocalDate date=LocalDate.of(2022, 12, 12);
// 根据 LocalDate 获取 ZonedDateTime
ZonedDateTime zdt1=date.atStartOfDay(zoneId);

LocalDateTime dateTime=LocalDateTime.of(2022, 12, 12, 13, 14);
// 根据 LocalDateTime 获取 ZonedDateTime
ZonedDateTime zdt2=dateTime.atZone(zoneId);

Instant instant=Instant.now();
// 根据 Instant 获取 ZonedDateTime
ZonedDateTime zdt3=instant.atZone(zoneId);
```

# 总结

* Java 8之前老版的 `java.util.Date` 类以及其他用于日期时间的类在设计上有缺陷，比如可变性，尴尬的起始时间，默认值和包名。
* 新版的日期和时间API中，日期-时间对象是不可变的。
* 新的API提供了两种不同的时间表示方式，有效地区分了运行时人 `LocalDate` 和机器 `Instant` 的不同需求。
* 操纵时间时钟返回一个全新的日期和时间，不会改变原本的日期和时间。
* 使用 `TemporalAdjuster` 可以更精细的方式操纵日期，不再局限于一次只能改变它的一个值，并且你还可按照需求定义自己的日期转换器。
* 格式化时间也变得线程安全，而且可以根据自己的意愿自定义格式化风格。

# 参考资料

* [51CTO - Java8全新日期、时间API在这全明白了](https://www.51cto.com/article/744423.html)