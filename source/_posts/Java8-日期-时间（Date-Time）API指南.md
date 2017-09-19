---
title: Java8 日期/时间（Date Time）API指南
categories:
  - java
  - java8
date: 2017-09-19 16:39:51
tags: Java8 日期/时间工具类
---
# 为什么我们需要新的Java日期/时间API？

在开始研究Java 8日期/时间API之前，让我们先来看一下为什么我们需要这样一个新的API。在Java中，现有的与日期和时间相关的类存在诸多问题，其中有：
1、Java的日期/时间类的定义并不一致，在java.util和java.sql的包中都有日期类，此外用于格式化和解析的类在java.text包中定义。
2、java.util.Date同时包含日期和时间，而java.sql.Date仅包含日期，将其纳入java.sql包并不合理。另外这两个类都有相同的名字，这本身就是一个非常糟糕的设计。
3、对于时间、时间戳、格式化以及解析，并没有一些明确定义的类。对于格式化和解析的需求，我们有java.text.DateFormat抽象类，但通常情况下，SimpleDateFormat类被用于此类需求。
4、所有的日期类都是可变的，因此他们都不是线程安全的，这是Java日期类最大的问题之一。
5、日期类并不提供国际化，没有时区支持，因此Java引入了java.util.Calendar和java.util.TimeZone类，但他们同样存在上述所有的问题。
在现有的日期和日历类中定义的方法还存在一些其他的问题，但以上问题已经很清晰地表明：Java需要一个健壮的日期/时间类。这也是为什么Joda Time在Java日期/时间需求中扮演了高质量替换的重要角色。

<!--more-->
# Java 8日期/时间API设计原则
1、不变性：新的日期/时间API中，所有的类都是不可变的，这对多线程环境有好处。
2、关注点分离：新的API将人可读的日期时间和机器时间（unix timestamp）明确分离，它为日期（Date）、时间（Time）、日期时间（DateTime）、时间戳（unix timestamp）以及时区定义了不同的类。
3、清晰：在所有的类中，方法都被明确定义用以完成相同的行为。举个例子，要拿到当前实例我们可以使用now()方法，在所有的类中都定义了format()和parse()方法，而不是像以前那样专门有一个独立的类。为了更好的处理问题，所有的类都使用了工厂模式和策略模式，一旦你使用了其中某个类的方法，与其他类协同工作并不困难。
4、实用操作：所有新的日期/时间API类都实现了一系列方法用以完成通用的任务，如：加、减、格式化、解析、从日期/时间中提取单独部分，等等。
5、可扩展性：新的日期/时间API是工作在ISO-8601日历系统上的，但我们也可以将其应用在非IOS的日历上。

# Java日期/时间API包

Java日期/时间API包含以下相应的包。
1、java.time包：这是新的Java日期/时间API的基础包，所有的主要基础类都是这个包的一部分，如：LocalDate, LocalTime, LocalDateTime, Instant, Period, Duration等等。所有这些类都是不可变的和线程安全的，在绝大多数情况下，这些类能够有效地处理一些公共的需求。
2、java.time.chrono包：这个包为非ISO的日历系统定义了一些泛化的API，我们可以扩展AbstractChronology类来创建自己的日历系统。
3、java.time.format包：这个包包含能够格式化和解析日期时间对象的类，在绝大多数情况下，我们不应该直接使用它们，因为java.time包中相应的类已经提供了格式化和解析的方法。
4、java.time.temporal包：这个包包含一些时态对象，我们可以用其找出关于日期/时间对象的某个特定日期或时间，比如说，可以找到某月的第一天或最后一天。你可以非常容易地认出这些方法，因为它们都具有“withXXX”的格式。
5、java.time.zone包：这个包包含支持不同时区以及相关规则的类。

# Java日期/时间API示例

## 如何在java8中获取当天的日期
java8中有个叫LocalDate的类，能用来表示今天的日期。这个类与java.util.Date略有不同，因为它只包含日期，没有时间。

    LocalDate today = LocalDate.now();
    --
    2017-09-19
## 如何在java8中获取当前的年月日
LocalDate类中提供了一些很方便的方法可以用来提取年月日以及其他的日期属性,特别方便，只需要使用对应的getter方法就可以了，非常直观。

	LocalDate today = LocalDate.now();
    int year = today.getYear();
    int month = today.getMonthValue();
    int day = today.getDayOfMonth();
    System.out.println(year);
    System.out.println(month);
    System.out.println(day);
## 在java8中如何获取某个特定的日期
通过另一个方法，可以创建出任意一个日期，它接受年月日的参数，然后返回一个等价的LocalDate实例。在这个方法里，需要的日期你填写什么就是什么，不想之前的API中月份必须从0开始。

    LocalDate birthDay = LocalDate.of(2001, 1, 1);
    System.out.println(birthDay);
## 在java8中检查两个日期是否相等
LocalDate重写了equals方法来进行日期的比较。

	LocalDate todayEq = LocalDate.of(2017,9,19);
	System.out.println(todayEq.equals(today));
## 在java8中如何检查重复事件，比如生日
在java中还有一个与时间日期相关的任务就是检查重复事件，比如每月的账单日
如何在java中判断是否是某个节日或者重复事件，使用MonthDay类。这个类由月日组合，不包含年信息，可以用来代表每年重复出现的一些日期或其他组合。他和新的日期库中的其他类一样也都是不可变且线程安全的，并且它还是一个值类（value class）。

	MonthDay birth_Day = MonthDay.of(9,19);
    MonthDay to_Day = MonthDay.from(today);
    if(birth_Day.equals(to_Day)) {
        System.out.println("今天是你的生日");
    }else {
        System.out.println("今天不是你的生日");
    }
## 如何在java8中获取当前时间
这个与第一个例子获取当前日期非常相似，这里用的是LocalTime类，默认的格式是hh:mm:ss:nnn    

    LocalTime now = LocalTime.now();
    System.out.println(now);
    ---
    17:42:53.966
## 如何增加时间里面的小时数
很多时候需要对时间进行操作，比如加一个小时来计算之后的时间，java8提供了更方便的方法 如plusHours，这些方法返回的是一个新的LocalTime实例的引用，因为LocalTime是不可变的。

	LocalTime now = LocalTime.now();
    System.out.println("当前时间：" + now);
    LocalTime two = now.plusHours(2);
    System.out.println("2小时后的时间：" + two);
    ---
    当前时间：17:45:12.091
	2小时后的时间：19:45:12.091
## 在java8中使用时钟
java8自带了Clock类，可以用来获取某个时区下（所以对时区是敏感的）当前的瞬时时间、日期。用来代替System.currentTimelnMillis()与TimeZone.getDefault()方法。

    Clock clock = Clock.systemUTC();
    System.out.println(clock);
    --
    SystemClock[Z]
    
## 在java中如何判断某个日期在另一个日期的前面还是后面
如何判断某个日期在另一个日期的前面还是后面或者相等，在java8中，LocalDate类中使用isBefore()、isAfter()、equals()方法来比较两个日期。如果调用方法的那个日期比给定的日期要早的话，isBefore()方法会返回true。equals()方法在前面的例子中已经说明了。

    LocalTime now = LocalTime.now();
    System.out.println("当前时间：" + now);
    LocalTime two = now.plusHours(2);
    System.out.println("2小时后的时间：" + two);
    System.out.println(now.isBefore(two));
    ---
    当前时间：17:49:21.295
    2小时后的时间：19:49:21.295
    true
## 在java8中处理不同的时区
java8中不仅将日期和时间进行了分离，同时还有时区。比如ZonId代表的是某个特定时区，ZonedDateTime代表带时区的时间，等同于以前的GregorianCalendar类。使用该类，可以将本地时间转换成另一个时区中的对应时间。

    LocalDateTime localDateTime = LocalDateTime.now();
    ZoneId zone = ZoneId.of(ZoneId.SHORT_IDS.get("ACT"));
    ZonedDateTime zdt = ZonedDateTime.of(localDateTime, zone);
    System.out.println("特定时区时间为：" + zdt);
    ---
    特定时区时间为：2017-09-19T17:59:10.288+09:30[Australia/Darwin]
    
## 如何表示固定的日期，比如信用卡过期时间
正如MonthDay表示的是某个重复出现的日子，YearMonth是另外一个组合，代表的是像信用卡还款日，定期存款到期日，options到期日这类的日期。你可以用这个类找出这个月有多少天，LengthOfMonth()这个方法返回的是这个YearMonth实例有多少天，这对于检查2月是否润2月很有用。

    YearMonth currentYearMonth = YearMonth.now();
    System.out.printf("这个月的是%s有%d天",currentYearMonth,currentYearMonth.lengthOfMonth());
    System.out.println( "currentYearMonth.atEndOfMonth()：" + currentYearMonth.atEndOfMonth());
    YearMonth creditCartExpiry = YearMonth.of(2018, Month.OCTOBER);
    System.out.printf("你选择的年月日期是%s %n ",creditCartExpiry);
    ---
    这个月的是2017-09有30天
    currentYearMonth.atEndOfMonth()：2017-09-30
    你选择的年月日期是2018-10 
## 如何在java8中检查闰年
LocalDate类由一个isLeapYear()方法来返回当前LocalDate对应的那年是否是闰年。

	LocalDate today = LocalDate.now();
	System.out.println(today.isLeapYear());
    ---
     false
## 计算两个日期之间包含多少天，多少月
计算两个日期之间包含多少天、周、月、年。可以用java.time.Period类完成该功能。下面例子中将计算日期与将来的日期之间一共有几个月。

    LocalDate today = LocalDate.now();
    LocalDate date1 = LocalDate.of(2017, Month.JULY, 12);
    Period period= Period.between(today, date1);
    System.out.println(period.getDays());
    System.out.println(period.getMonths());
    ---
    -7
	-2
## 在java8中获取当前时间戳
java8获取时间戳特别简单。Instant类由一个静态的工厂方法now()可以返回当前时间戳

    Instant it = Instant.now();
    System.out.println(it);
    
可以看到，当前时间戳是包含日期和时间的，与java.util.Date很类似，事实上Instant就是java8以前的Date，可以使用这个两个类中的方法在这两个类型之间进行转换，比如Date.from(Instant)就是用来把Instant转换成java.util.date的，而Date。toInstant()就是将Date转换成Instant的
    
## 如何在java8中使用预定义的格式器来对日期进行解析/格式化
在java8之前，时间日期的格式化非常麻烦，经常使用SimpleDateFormat来进行格式化，但是SimpleDateFormat并不是线程安全的。在java8中，引入了一个全新的线程安全的日期与时间格式器。并且预定义好了格式。比如，本例中使用的BASICISODATE格式会将20160414格式化成2016-04-14

        String day = "20130923";
        LocalDate formatterd = LocalDate.parse(day, DateTimeFormatter.BASIC_ISO_DATE);
        System.out.println(formatterd);
        ---
        2013-09-23
## 如何在java中使用自定义的格式器来解析日期
 在上例中，我们使用了预置的时间日期格式器来解析日期字符串了，但是有时预置的不能满足的时候就需要我们自定义日期格式器了，下面的例子中的日期格式是"MM dd yyyy".你可以给DateTimeFormatter的ofPattern静态方法()传入任何的模式，它会返回一个实例，这个模式的字面量与前例中是相同的。比如M代表月，m仍代表分，无效的模式会抛异常DateTimeParseException。
 
 	String day1 = "2013 09 23";
    try {
        DateTimeFormatter df = DateTimeFormatter.ofPattern("yyyy MM dd");
        LocalDate holiday = LocalDate.parse(day1,df);
        System.out.println(holiday);
    } catch (DateTimeParseException e) {
        // TODO: handle exception
    }
    ---
    2013-09-23
## 如何在java8中对日期进行格式化，转换成字符串
 前面的两个例子中，我们主要是对日期字符串来进行解析转换成日期，在这个例子我们相反，是把日期转换成字符。这里我们有个LocalDateTime类的实例，我们要把他转换成一个格式化好的日期串，与前例相同的是，我们仍需要制定模式串去创建一个DateTimeFormatter类的实例，但调用的是LocalDate.format()。这个方法会返回一个代表当前日期的字符串，对应的模式就是传入的DateTimeFormatter实例中定义好的。
 
 	LocalDateTime aDate = LocalDateTime.now();
    try {
        DateTimeFormatter df = DateTimeFormatter.ofPattern("yyyy MM dd");
        System.out.println(aDate.format(df));
    } catch (Exception e) {
        // TODO: handle exception
    }
    ---
    2017 09 19
    
    
    
    
    
    
    
    
    
    
    
    