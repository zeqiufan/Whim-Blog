# equals & hashcode
## 前言
非常经典的问题，如何定义一个实例对象“相等”。java中一般提供两种形式：
```java
a == b
a.equals(b)
```
其中 == 是在比较a和b在jvm中的内存地址，而equals()是所有类的父类Object中定义的方法。
## equals
java中所有对象，都是Object类的子类，当然这里面不包括基础变量类型，就是int、dobule之类的，以下只讨论引用类型，比如Integer、Double、String之类。  
### 解析
首先看一眼Object类中equals函数的源码：
```java
public boolean equals(Object obj) {
    return (this == obj);
}
```
相当简单粗暴，就是 == 的逻辑。  
但是这种比较其实容易有问题，比如：
```java
String a = new String("123");
String b = new String("123");

System.out.println(a == b); //(1)
System.out.println(a.equals(b)); //(2)
```
代码(1)输出false，代码(2)输出true。
这里 == 输出false是因为a和b指向了内存中两块不同的地址，而equals输出true是因为String重写了equals方法：
```java
public boolean equals(Object anObject) {
    if (this == anObject) { // 先看引用是否相同，引用相同那么一定相同（指向同一个地方）
        return true;
    }
    if (anObject instanceof String) { // 如果输入也是String，那么进一步判断（否则不用比了，类型都不同肯定不同）
        String anotherString = (String)anObject; // 转成String
        int n = value.length;
        if (n == anotherString.value.length) { // 字符长度是否相等，不等则肯定不同
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) { // 挨个比较字符
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```
也是很简单的一个逻辑，可以粗略理解为将两个字符串中的字符挨个比较。  
相似的，Integer也重写了自己的equals函数：
```java
public boolean equals(Object obj) {
    if (obj instanceof Integer) {
        return value == ((Integer)obj).intValue();
    }
    return false;
}
```
其中intValue函数为：
```java
public int intValue() {
    return value;
}
```
Integer的这个逻辑更好理解，其实就是在比较两个整数大小。  

### 结论
看到这里应该就明白了，equals方法默认比较两个对象的引用地址，如果你觉得这个逻辑无法满足你的业务场景，那么就请重写该类的equals方法。  
相似的，可以去看看Long和Double类，Double里面还涉及一个精度的问题，稍微复杂一点点。

## hashcode
### 用途
hashCode()也是Object类的一个函数，返回值为int，可以理解为，“返回当前对象的一个哈希值”。举个使用场景的例子就是HashMap<K, V>中的K，HashMap的实现离不开hashcode。
### 源码解析
不过hashCode()是一个native函数，native回头再讲，这里可以粗略理解为“调用了其他语言写好了的功能接口”，理解为一个黑盒子也许会方便一点，毕竟源码是这样的：
```java
public native int hashCode();
```
对，没有常见的函数体。具体效果我看到的解释是，返回该对象在jvm中的内存地址，也就是上文中 == 用到的那个。  
相似的，在String等类中hashCode()函数也被重写了，如下是String中的源码：
```java
/**
 * s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]
 */
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```
具体返回值看注释里的公式就行了，如此实现了一个从字符串到数字的映射，因此上文中 a.equals(b) 的返回值为true，因为"123"和"123"用该方法算出来的哈希值均为48690。
```
49*31^2 + 50*31^1 + 51*31^0 = 48690
```
因此如果是自己定义的类，比如自定义了一个Student类：
```java
public class Student {
    private String sno;
    private String name;
    ……

    public String getSno() {
        return sno;
    }
    ……
}
```
那么如果直接将Student类放到HashMap的Key中（个人一般不使用这种写法），这种写法一般需要自己去重写hashCode()函数，如果不重写那估计就等着出bug吧：
```java
Map<Student, Object> map = new HashMap<>();

Student stu = new Student();
map.put(stu, "xxx");
```
如果业务逻辑是使用学号来定义是不是同一个学生，那么可以将hashCode()函数重写为：
```java
@Override
public int hashCode() {
    return this.sno.hashCode();
}
```
个人一般使用String作为Key，这样相当于是在使用String类重写好的hashCode()函数：
```java
Map<String, Object> map = new HashMap<>();

Student stu = new Student();
map.put(stu.getSno(), "xxx");
```

## 后记
写的多或少完全看心情，不过考虑到个人忙的程度，像这样详细的写法应该不会多哈哈哈。