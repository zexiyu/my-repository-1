

# java.lang.String类的使用
## 1.概述
String:字符串，使用一对""引起来表示。

1. String声明为final的，不可被继承
2. String实现了Serializable接口：表示字符串是支持序列化的，实现了Comparable接口：表示String可以比较大小
3. String内部定义了final char[] value用于存储字符串数据23
4. 通过字面量的方式赋值（区别于new一个字符串赋值)，此时的字符串值声明在字符串常量池中。
5. 字符串常量池中是不会存储相同内容的字符串。(字符串常量池中使用String类的equals()比较，返回true或false)

## 2.String的不可变性



### 2.1 说明

1. 当对字符串重新赋值时，需要重新指定内存区域赋值，不能使用原有的value进行赋值。

2. 当对现有的字符串进行连接操作时，需要重新指定内存区域赋值，不能使用原有的value进行赋值。

3. 当调用String的replace()方法修改指定字符或字符串时，需要重新指定内存区域赋值，不能使用原有的value进行赋值。 




### 2.2 代码举例
```java
 String s1 = "abc";//字面量的定义方式
 String s2 = "abc";
 s1 = "hello";

System.out.println(s1 == s2);//比较s1和s2的地址值

System.out.println(s1);//hello
System.out.println(s2);//abc

System.out.println("*****************");

String s3 = "abc";
s3 += "def";
System.out.println(s3);//abcdef
System.out.println(s2);//abc

System.out.println("*****************");

String s4 = "abc";
String s5 = s4.replace('a', 'm');
System.out.println(s4);//abc
System.out.println(s5);//mbc
```



图示：当两个String变量的值都为同一个时，栈中的两个变量都指向方法区常量池中的同一个常量的地址值，当改变String中一个变量的值时，变量会指向新的常量的地址值

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1608643340822.png" alt="1608643340822" style="zoom: 90%;" />



### 2.3String不可变的好处 

可以实现多个变量引用堆内存中的同一个字符串实例，避免创建的开销。 

我们的程序中大量使用了String字符串，有可能是出于安全性考虑。 

大家都知道HashMap中key为String类型，如果可变将变的多么可怕。 

当我们在传参的时候，使用不可变类不需要去考虑谁可能会修改其内部的值，如果使用可变类的话，可能需要每次记得重新拷贝出里面的值，性能会有一定的损失。 



## 3.String实例化的不同方式

1.  方式说明
   方式一：通过字面量定义的方式
   方式二：通过new + 构造器的方式

2. 代码举例

   ```java
     //通过字面量定义的方式：此时的s1和s2的数据javaEE声明在方法区中的字符串常量池中。
      String s1 = "javaEE";
      String s2 = "javaEE";
      //通过new + 构造器的方式:此时的s3和s4保存的地址值，是数据在堆空间中开辟空间以后对应的地址值。
      String s3 = new String("javaEE");
      String s4 = new String("javaEE");
   
      System.out.println(s1 == s2);//true
      System.out.println(s1 == s3);//false
      System.out.println(s1 == s4);//false
      System.out.println(s3 == s4);//false
      System.out.println(s1.equals(s3)); //true	
   ```
   
   **分析:**
   因为String太过常用，JAVA类库的设计者在实现时做了个小小的变化，即采用了享元模式,每当生成一个新内容的字符串时，他们都被添加到一个共享池中，当第二次再次生成同样内容的字符串实例时，就共享此对象，而不是创建一个新对象，但是这样的做法仅仅适合于通过=符号进行的初始化。需要说明一点的是，在object中，equals()是用来比较内存地址的，但是String重写了Object的equals()方法，用来比较内容了：即使是不同地址，只要内容一致，也会返回true，这也就是为什么s1.equals(s3)返回true的原因了。
   
   
   
3.  面试题:

    String s = new String("abc");方式创建对象，在内存中创建了几个对象？
    两个:一个是堆空间中new结构，另一个是char[]对应的常量池中的数据："abc"

    
    
    两种实例化方式的区别 ？
    
    1. 直接赋值（String str = "hello"）：只开辟一块堆内存空间，并且会自动入池，不会产生垃圾。 
    
    2. 构造方法（String str= new String("hello");）:会开辟两块堆内存空间，其中一块堆内存会变成垃圾 被系统回收，而且不能够自动入池，需要通过public String intern();方法进行手工入池。 
    
    3. 在开发的过程中不会采用构造方法进行字符串的实例化。 

 图示

![1608643361832](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1608643361832.png)



## 4.字符串拼接方式赋值的对比



说明:

- 常量与常量的拼接结果在常量池。且常量池中不会存在两个甚至多个相同内容的常量。

- 只要其中一个是变量，结果就会在堆中。 

- 如果拼接的结果调用intern()方法，返回值就在常量池中（intern方法是对String类的对象进行手工入池操作）

  

  代码举例

  ```java
  String s1 = "javaEE";
  String s2 = "hadoop";
  String s3 = "javaEEhadoop";
  String s4 = "javaEE" + "hadoop";
  String s5 = s1 + "hadoop";
  String s6 = "javaEE" + s2;
  String s7 = s1 + s2;
  
  //只要字符串中有变量，结果就会在堆中而不在常量池
  System.out.println(s3 == s4);//true
  System.out.println(s3 == s5);//false
  System.out.println(s3 == s6);//false
  System.out.println(s3 == s7);//false
  System.out.println(s5 == s6);//false
  System.out.println(s5 == s7);//false
  System.out.println(s6 == s7);//false
  
  String s8 = s6.intern();//返回值得到的s8使用的常量值中已经存在的“javaEEhadoop”拼接的结果调用intern()方法，返回值就在常量池中
  System.out.println(s3 == s8);//true
  
  ****************************
      
  String s1 = "javaEEhadoop";
  String s2 = "javaEE";
  String s3 = s2 + "hadoop";
  System.out.println(s1 == s3);//false
  final String s4 = "javaEE";//s4:常量
  String s5 = s4 + "hadoop";
  System.out.println(s1 == s5);//true
  ```





## 5.String的常用方法：

>int. length()：返回字符串的长度： return value.length
>char charAt(int index)： 返回某索引处的字符return value[index]
>boolean isEmpty()：判断是否是空字符串：return value.length == 0
>String toLowerCase()：使用默认语言环境，将 String 中的所字符转换为小写
>String toUpperCase()：使用默认语言环境，将 String 中的所字符转换为大写
>String trim()：返回字符串的副本，忽略前导空白和尾部空白
>boolean equals(Object obj)：比较字符串的内容是否相同
>boolean equalsIgnoreCase(String anotherString)：与equals方法类似，忽略大小写
>String concat(String str)：将指定字符串连接到此字符串的结尾。 等价于用“+”
>int compareTo(String anotherString)：比较两个字符串的大小
>String substring(int beginIndex)：返回一个新的字符串，它是此字符串的从beginIndex开始截取到最后的一个子字符串。
>String substring(int beginIndex, int endIndex) ：返回一个新字符串，它是此字符串从beginIndex开始截取到endIndex(不包含)的一个子字符串。
>
>boolean endsWith(String suffix)：测试此字符串是否以指定的后缀结束
>boolean startsWith(String prefix)：测试此字符串是否以指定的前缀开始
>boolean startsWith(String prefix, int toffset)：测试此字符串从指定索引开始的子字符串是否以指定前缀开始
>
>boolean contains(CharSequence s)：当且仅当此字符串包含指定的 char 值序列时，返回 true
>int indexOf(String str)：返回指定子字符串在此字符串中第一次出现处的索引
>int indexOf(String str, int fromIndex)：返回指定子字符串在此字符串中第一次出现处的索引，从指定的索引开始
>int lastIndexOf(String str)：返回指定子字符串在此字符串中最右边出现处的索引
>int lastIndexOf(String str, int fromIndex)：返回指定子字符串在此字符串中最后一次出现处的索引，从指定的索引开始反向搜索
>
>注：indexOf和lastIndexOf方法如果未找到都是返回-1
>
>替换：
>String replace(char oldChar, char newChar)：返回一个新的字符串，它是通过用 newChar 替换此字符串中出现的所 oldChar 得到的。
>String replace(CharSequence target, CharSequence replacement)：使用指定的字面值替换序列替换此字符串所匹配字面值目标序列的子字符串。
>String replaceAll(String regex, String replacement)：使用给定的 replacement 替换此字符串所匹配给定的正则表达式的子字符串。
>String replaceFirst(String regex, String replacement)：使用给定的 replacement 替换此字符串匹配给定的正则表达式的第一个子字符串。
>匹配:
>boolean matches(String regex)：告知此字符串是否匹配给定的正则表达式。
>切片：
>String[] split(String regex)：根据给定正则表达式的匹配拆分此字符串。
>String[] split(String regex, int limit)：根据匹配给定的正则表达式来拆分此字符串，最多不超过limit个，如果超过了，剩下的全部都放到最后一个元素中。



## 6.String与其它结构的转换	 
### 6.1 与基本数据类型、包装类之间的转换
 String --> 基本数据类型、包装类：调用包装类的静态方法：parseXxx(str)
 基本数据类型、包装类 --> String:调用String重载的valueOf(xxx)

```java
 @Test
 public void test1(){
     String str1 = "123";
//    int num = (int)str1;//错误的
     int num = Integer.parseInt(str1);

 String str2 = String.valueOf(num);//"123"
 String str3 = num + "";

 System.out.println(str1 == str3); //false

 }
```



### 6.2 与字符数组之间的转换
String --> char[]:调用String的toCharArray()
char[] --> String:调用String的构造器



```java
@Test
public void test2(){
    String str1 = "abc123";  

char[] charArray = str1.toCharArray();
for (int i = 0; i < charArray.length; i++) {
    System.out.println(charArray[i]);
}

char[] arr = new char[]{'h','e','l','l','o'};
String str2 = new String(arr);
System.out.println(str2);

}
```

### 6.3 与字节数组之间的转换
编码：String --> byte[]:调用String的getBytes()
解码：byte[] --> String:调用String的构造器

编码：字符串 -->字节  (看得懂 --->看不懂的二进制数据)
解码：编码的逆过程，字节 --> 字符串 （看不懂的二进制数据 ---> 看得懂

说明：解码时，要求解码使用的字符集必须与编码时使用的字符集一致，否则会出现乱码。



```java

@Test
public void test3() throws UnsupportedEncodingException {
    String str1 = "abc123中国";
    byte[] bytes = str1.getBytes();//使用默认的字符集，进行编码。
    System.out.println(Arrays.toString(bytes));
    byte[] gbks = str1.getBytes("gbk");//使用gbk字符集进行编码。
	System.out.println(Arrays.toString(gbks));
	
	System.out.println("******************");

	String str2 = new String(bytes);//使用默认的字符集，进行解码。
	System.out.println(str2);

	String str3 = new String(gbks);
	System.out.println(str3);//出现乱码。原因：编码集和解码集不一致！
	String str4 = new String(gbks, "gbk");
	System.out.println(str4);//没出现乱码。原因：编码集和解码集一致！
}
```





### 6.4 与StringBuffer、StringBuilder之间的转换

- String -->StringBuffer、StringBuilder:

  调用StringBuffer、StringBuilder构造器

  

- StringBuffer、StringBuilder -->String:

  1. 调用String构造器；
  2. StringBuffer、StringBuilder的toString()
  3. 

### 7.JVM中字符串常量池存放位置说明：
jdk 1.6 (jdk 6.0 ,java 6.0):字符串常量池存储在方法区（永久区）
jdk 1.7:字符串常量池存储在堆空间
jdk 1.8:字符串常量池存储在方法区（元空间）



### 常见算法题目的考查：
1）模拟一个trim方法，去除字符串两端的空格。

2）将一个字符串进行反转。将字符串中指定部分进行反转。比如“abcdefg”反转为”abfedcg”

3）获取一个字符串在另一个字符串中出现的次数。
      比如：获取“ ab”在 “abkkcadkabkebfkabkskab” 中出现的次数

4）获取两个字符串中最大相同子串。比如：
   str1 = "abcwerthelloyuiodef“;str2 = "cvhellobnm"
   提示：将短的那个串进行长度依次递减的子串与较长的串比较。

5）对字符串中字符进行自然顺序排序。
提示：
1字符串变成字符数组。
2对数组排序，择，冒泡，Arrays.sort();
3将排序后的数组变成字符串。



# StringBuffer、StringBuilder



## 1.String、StringBuffer、StringBuilder三者的对比
String:不可变的字符序列；底层使用char[]存储
StringBuffer:  可变的字符序列；线程安全的，   效率低；底层使用char[]存储
StringBuilder:可变的字符序列；线程不安全的，效率高；底层使用char[]存储

## 2.StringBuffer与StringBuilder的内存解析
以StringBuffer为例：

```java
String str = new String();//char[] value = new char[0];
String str1 = new String("abc");//char[] value = new char[]{'a','b','c'};

StringBuffer sb1 = new StringBuffer();//char[] value = new char[16];底层创建了一个长度是16的数组。
System.out.println(sb1.length());//
sb1.append('a');//value[0] = 'a';
sb1.append('b');//value[1] = 'b';

StringBuffer sb2 = new StringBuffer("abc");//char[] value = new char["abc".length() + 16];
```



//问题1. System.out.println(sb2.length());//3
//问题2. 扩容问题:如果要添加的数据底层数组盛不下了，那就需要扩容底层的数组。
    默认情况下，扩容为原来容量的**2倍 + 2**，同时将原数组中的元素复制到新的数组中。

​    指导意义：开发中建议大家使用：StringBuffer(int capacity) 或 StringBuilder(int capacity)

## 3.对比String、StringBuffer、StringBuilder三者的执行效率
从高到低排列：StringBuilder > StringBuffer > String



## 4.StringBuffer、StringBuilder中的常用方法

增：append(xxx)
删：delete(int start,int end)
改：setCharAt(int n ,char ch) / replace(int start, int end, String str)
查：charAt(int n )
插：insert(int offset, xxx)
长度：length();
*遍历：for() + charAt() / toString()



# JDK8之前的时间API

## 1.获取系统当前时间：System类中的currentTimeMillis()

```java
long time = System.currentTimeMillis();

System.out.println(time);

//返回当前时间与1970年1月1日0时0分0秒之间以毫秒为单位的时间差,称为时间戳
```



## 2.java.util.Date类与java.sql.Date类

 java.util.Date类
        |---java.sql.Date类

 1.两个构造器的使用

> 构造器一：Date()：创建一个对应当前时间的Date对象
> 构造器二：创建指定毫秒数的Date对象

  2.两个方法的使用

> toString():显示当前的年、月、日、时、分、秒
> getTime():获取当前Date对象对应的毫秒数。（时间戳）



java.sql.Date对应着数据库中的日期类型的变量
如何将java.util.Date对象转换为java.sql.Date对象

```java
 @Test
 public void test2(){
    //构造器一：Date()：创建一个对应当前时间的Date对象
    Date date1 = new Date();
    System.out.println(date1.toString());//Sat Feb 16 16:35:31 GMT+08:00 2019

    System.out.println(date1.getTime());//1550306204104

    //构造器二：创建指定毫秒数的Date对象
    Date date2 = new Date(155030620410L);
    System.out.println(date2.toString());

    //创建java.sql.Date对象
    java.sql.Date date3 = new java.sql.Date(35235325345L);
    System.out.println(date3);//1971-02-13

    //如何将java.util.Date对象转换为java.sql.Date对象
    //情况一：利用多态进行强转
    //        Date date4 = new java.sql.Date(2343243242323L);
    //        java.sql.Date date5 = (java.sql.Date) date4;
    //情况二：
    Date date6 = new Date();
    java.sql.Date date7 = new java.sql.Date(date6.getTime());

  }


```





## 3.java.text.SimpleDataFormat类

SimpleDateFormat对日期Date类的格式化和解析
1.两个操作：
1.1 格式化：日期 --->字符串
1.2 解析：格式化的逆过程，字符串 ---> 日期

2.SimpleDateFormat的实例化:new + 构造器



**照指定的方式格式化和解析：调用带参的构造器**

```java
//      SimpleDateFormat sdf1 = new SimpleDateFormat("yyyyy.MMMMM.dd GGG hh:mm aaa");
        SimpleDateFormat sdf1 = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        //格式化
        String format1 = sdf1.format(date);
        System.out.println(format1);//2019-02-18 11:48:27
        //解析:要求字符串必须是符合SimpleDateFormat识别的格式(通过构造器参数体现),
        //否则，抛异常
        Date date2 = sdf1.parse("2020-02-18 11:48:27");
        System.out.println(date2);
```

其中 yyyy 是完整的公元年，MM 是月份，dd 是日期，HH:mm:ss 是时、分、秒。 

**注意**:有的格式大写，有的格式小写，例如 MM 是月份，mm 是分；HH 是 24 小时制，而 hh 是 12 小时制。

小练习：

​    练习一：字符串"2020-09-08"转换为java.sql.Date

    练习二："天打渔两天晒网"   1990-01-01  xxxx-xx-xx 打渔？晒网？
    
    举例：2020-09-08 ？ 总天数
    
    总天数 % 5 == 1,2,3 : 打渔
    总天数 % 5 == 4,0 : 晒网
    
    总天数的计算？
    方式一：( date2.getTime() - date1.getTime()) / (1000 * 60 * 60 * 24) + 1
    方式二：1990-01-01  --> 2019-12-31  +  2020-01-01 -->2020-09-08
     */


练习一答案：

```java
      @Test
public void testExer() throws ParseException {
  String birth = "2020-09-08";
  SimpleDateFormat sdf1 = new SimpleDateFormat("yyyy-MM-dd");
    Date date = sdf1.parse(birth);
 //        System.out.println(date);
  java.sql.Date birthDate = new java.sql.Date(date.getTime());
    System.out.println(birthDate);
}
```


## 4.Calendar类：日历类、抽象类

```java
   
// 1.实例化
  //方式一：创建其子类（GregorianCalendar）的对象
  //方式二：调用其静态方法getInstance()
   Calendar calendar = Calendar.getInstance();
//   System.out.println(calendar.getClass());

//2.常用方法
    //get()
    int days = calendar.get(Calendar.DAY_OF_MONTH);
    System.out.println(days);
    System.out.println(calendar.get(Calendar.DAY_OF_YEAR));

    //set()
    //calendar可变性
    calendar.set(Calendar.DAY_OF_MONTH,22);
    days = calendar.get(Calendar.DAY_OF_MONTH);
    System.out.println(days);

    //add()
    calendar.add(Calendar.DAY_OF_MONTH,-3);
    days = calendar.get(Calendar.DAY_OF_MONTH);
    System.out.println(days);

    //getTime():日历类---> Date
    Date date = calendar.getTime();
    System.out.println(date);

    //setTime():Date ---> 日历类
    Date date1 = new Date();
    calendar.setTime(date1);
    days = calendar.get(Calendar.DAY_OF_MONTH);
    System.out.println(days);
```





# JDK8中新的时间API



## 1.日期时间API的迭代：
- 第一代：jdk 1.0 Date类
- 第二代：jdk 1.1 Calendar类，一定程度上替换Date类
- 第三代：jdk 1.8 提出了新的一套API



## 2.前两代存在的问题举例：

- 可变性：像日期和时间这样的类应该是不可变的。

- 偏移性：Date中的年份是从1900开始的，而月份都从0开始。

- 格式化：格式化只对Date用，Calendar则不行。

  

  此外，它们也不是线程安全的；不能处理闰秒等。



## 3.java 8 中新的日期时间API涉及到的包

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1608686136891.png" alt="1608686136891" style="zoom:80%;" />





## 4.本地日期、本地时间、本地日期时间的使用：LocalDate / LocalTime / LocalDateTime

### 4.1 说明：
① 分别表示使用 ISO-8601日历系统的日期、时间、日期和时间。它们提供了简单的本地日期或时间，并不包含当前的时间信息，也不包含与时区相关的信息。
② LocalDateTime相较于LocalDate、LocalTime，使用频率要高
③ 类似于Calendar

### 4.2 常用方法：

![1608686230989](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1608686230989.png)



## 5.时间点：Instant

### 5.1 说明：

① 时间线上的一个瞬时点。 概念上讲，它只是简单的表示自1970年1月1日0时0分0秒（UTC开始的秒数。）
② 类似于 java.util.Date类

### 5.2 常用方法：

![1608686428891](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1608686428891.png)



## 6.日期时间格式化类：DateTimeFormatter

### 6.1 说明：

格式化或解析日期、时间，类似于SimpleDateFormat
### 6.2 常用方法：
① 实例化方式：
预定义的标准格式。如：ISO_LOCAL_DATE_TIME;ISO_LOCAL_DATE;ISO_LOCAL_TIME
本地化相关的格式。如：ofLocalizedDateTime(FormatStyle.LONG)
自定义的格式。如：ofPattern(“yyyy-MM-dd hh:mm:ss”)

② 常用方法：

![1608686818819](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1608686818819.png)

特别的：自定义的格式。如：ofPattern(“yyyy-MM-dd hh:mm:ss”)

```java
//  重点：自定义的格式。如：ofPattern(“yyyy-MM-dd hh:mm:ss”)
DateTimeFormatter formatter3 = DateTimeFormatter.ofPattern("yyyy-MM-dd hh:mm:ss");
//格式化
String str4 = formatter3.format(LocalDateTime.now());
System.out.println(str4);//2019-02-18 03:52:09

//解析
TemporalAccessor accessor = formatter3.parse("2019-02-18 03:52:09");
System.out.println(accessor);
```





# Java比较器



## 1.Java比较器的使用背景：

Java中的对象，正常情况下，只能进行比较：==  或  != 。不能使用 > 或 < 的
但是在开发场景中，我们需要对多个对象进行排序，言外之意，就需要比较对象的大小。

如何实现？**使用两个接口中的任何一个：Comparable 或 Comparator**



## 2.自然排序：使用Comparable接口



### 2.1 说明

1. String、包装类等实现了Comparable接口，重写了compareTo(obj)方法，给出了比较两个对象大小的方式。

2. String、包装类重写compareTo()方法以后,进行了从小到大的排列

    

3. 在TreeSet中默认要求里面的元素进行自然排序，强制要求里面的所有元素必须按照Comparable中的compareTo方法进行比较。按别的方式比较（比如按定制排序conparator比较）会报错

    　　如果容器里面的对象（比如：new Student（参数）对象）不具备compareTo方法此时就会抛出异常报错（**ClassCastException**），所以**必须要让容器中的元素实现Comparable接口**，这样它才具备compareTo方法。

    ​			注意点：

    　　　　1．TreeSet实例在调用add方法时会调用容器对象的compareTo方法对元素进行比较

    　　　　2．TreeSet实例中对象必须是实现了Comparable接口

    

4. 重写compareTo(obj)的规则：
    - 如果当前对象this大于形参对象obj时，则返回正整数，

    - 如果当前对象this小于形参对象obj时，则返回负整数，

    - 如果当前对象this等于形参对象obj时，返回0

      

      

5. 对于自定义类来说，如果需要排序，我们可以让自定义类实现Comparable接口，重写compareTo(obj)方法。在compareTo(obj)方法中指明如何排序

### 2.2 自定义类代码举例：

```java
public class Person implements Comparable{
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public Person() {
    }

    public String print(){
        return "person-----";
    }

    public String printPerson(){
        return "printPerson";
    }


    @Override
    public int compareTo(Object o) {
        if (o instanceof Person){
            Person person = (Person) o;
            if (this.age > person.age){
                return 1;  //如果当前对象this大于形参对象obj，则返回正整数，
            }else if (this.age<person.age){
                return -1;  //如果当前对象this小于形参对象obj，则返回负整数
            }else {
//                return 0;
                return -this.name.compareTo(person.name);   //姓名从大到小排序
            }
        }
        return 0;  
    }

    @Override
    public String toString() {
        return "<" + name + "\t" + age +
                '>';
    }
}
```

测试

```java
 TreeSet<Person> treeSet = new TreeSet<>();
        treeSet.add(new Person("aa",23));
        treeSet.add(new Person("ss",45));
        treeSet.add(new Person("dd",21));
        treeSet.add(new Person("rr",21));
        treeSet.add(new Person("bb",55));
        treeSet.add(new Person("nn",11));
        treeSet.add(new Person("mm",21));


        System.out.println(treeSet);
```

![image-20201223111156088](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201223111156088.png)

## 3.定制排序：使用Comparator接口



### 3.1 说明

1.背景：
当元素的类型没有实现java.lang.Comparable接口而又不方便修改代码，或者实现了java.lang.Comparable接口的排序规则不适合当前的操作，那么可以考虑使用 Comparator 的对象来排序



2.重写compare(Object o1,Object o2)方法，比较o1和o2的大小：
如果方法返回正整数，则表示o1大于o2；
如果返回0，表示相等；
返回负整数，表示o1小于o2。

### 3.2 代码举例：

```java
 TreeSet<Person> treeSet = new TreeSet<>(new Comparator<Person>() {
            @Override
            public int compare(Person o1, Person o2) {
                Person p1 = (Person) o1;
                Person p2 = (Person) o2;

                if (p1 instanceof Person&&p2 instanceof Person){
                    if (p1.getAge() > p2.getAge()) {
                        return 1;
                    } else if (p1.getAge() < p2.getAge()) {
                        return -1;
                    } else {
                        return -p1.getName().compareTo(p2.getName()); //姓名从大到小排序
                    }
                }
                throw new RuntimeException("输入的数据类型不一致");

            }
        });
        treeSet.add(new Person("aa", 23));
        treeSet.add(new Person("ss", 45));
        treeSet.add(new Person("dd", 21));
        treeSet.add(new Person("rr", 21));
        treeSet.add(new Person("bb", 55));
        treeSet.add(new Person("nn", 11));
        treeSet.add(new Person("mm", 21));

        System.out.println(treeSet);
    }
```

![image-20201223114903231](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201223114903231.png)



使用：
Arrays.sort(goods,com);
Collections.sort(coll,com);
new TreeSet(com);

## 两种排序方式对比

*    Comparable接口的方式一旦一定，保证Comparable接口实现类的对象在任何位置都可以比较大小。
*    Comparator接口属于临时性的比较。



# 其他类

## 1.System类

System类代表系统，系统级的很多属性和控制方法都放置在该类的内部。该类位于java.lang包。
由于该类的构造器是private的，所以无法创建该类的对象，也就是无法实例化该类。其内部的成员变量和成员方法都是static的，所以也可以很方便的进行调用。
方法：
native long currentTimeMillis()
void exit(int status)
void gc()
String getProperty(String key)



## 2.Math类

java.lang.Math提供了一系列静态方法用于科学计算。其方法的参数和返回值类型一般为double型。

## 3.BigInteger类、BigDecimal类

说明：
① java.math包的BigInteger可以表示不可变的任意精度的整数。
② 要求数字精度比较高，要用到java.math.BigDecimal类

代码举例：

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1608688361535.png" alt="1608688361535" style="zoom:67%;" />

