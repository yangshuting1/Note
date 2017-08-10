## 注解
### 1. 注解概述
注解起源：是JDK5.0及以后版本引入的，在java.lang.annotation包中。
注解作用：它可以用于创建文档，跟踪代码中的依赖性，甚至执行基本编译时检查。
注解格式：注解是以‘@注解名’在代码中存在的
注解分类：标记注解、单值注解、完整注解三类

### 2. java中常见的注解
#### 标准注解（JDK自带注解）

@Deprecated 意思是“废弃的，过时的”

@Override 意思是“重写、覆盖”

@SuppressWarnings 意思是“压缩,警告”


注解标识 | 作用|
--------- | -------------  | 
@Override | 表示当前的方法定义覆盖超类中的方法，如果一不小心拼写错误，或者签名方法对不上被覆盖的方法，编译器就会发生错误。
@Deprecated | 如果使用了注解为它的元素，编译器会发出警告信息
@SuppressWarnings | 关闭不当的编译器警告信息


[@override注解演示](http://note.youdao.com/noteshare?id=c8da0880776152f9407778359a08e0a3)（具体代码见test1/Person.java)

[@Deprecated  && @SuppressWarnings注解演示](http://note.youdao.com/noteshare?id=609fd2bede99847a8d95bba813b0614e)(具体代码见test1/Test.java)

#### 常见第三方注解

Spring的注解 | 
--------- |  
@Autowired | 
@Service |
@Repository |



Mabits的注解 | 
--------- |  
@InsertProvider | 
@UpdateProvider|
@Options|


### 3.注解的分类

注解可以分成三类：

1. 源码注解 
当javac把.java源文件编译成.class时，就将相应的注解去掉。这种注解的生命周期就维持到源文件阶段。
2. 编译时注解
在JVM通过ClassLoader向内存中加载字节码文件时候，JVM会去掉 相应的注解。这种注解的生命周期就维持到字节码文件阶段。无法使用反射来访问相应的注解。
3. 运行时注解
JVM运行内存的字节码时候，仍然可能会保留并且执行的某些注解。这种注解的生命周期就维持到内存字节码阶段。
注意：这个阶段，程序可以通过反射访问生命周期到内存字节码阶段的注解。


### 自定义注解

定义一个Description注解：

```java
@Documented
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD.ElementType.TYPE)
public @interface Description {
    String desc();
    int age() default 11;
    String author() default "absfree";
        
}

```
#### 自定义语法的语法要求
1. 注解类型是通过”@interface“关键字定义的
2. 在”注解体“中，所有的方法均没有方法体且只允许public和abstract这两种修饰符号（不加修饰符缺省为public），注解方法不允许有throws子句
3.可以通过default关键字给成员变量制定默认值
4.	注解方法的返回值只能为以下几种：原始数据类型, String, Class, Enum, Annotation它们的一维数组，可以为方法指定默认返回值 
5. 注解可以没有成员，没有成员的注解标注为标识注解
上面的四行就属于元注解，那么什么是元注解呢？
元注解：专职负责其他的注解
**@Target作用：用于描述注解的使用范围**

ElementType | 描述 |
--------- | -------------  | 
CONSTRUCTOR | 描述构造器
FIELD | 用于描述域
LOCAL_VARIABLE| 描述局部变量
METHOD| 描述方法
PACKAGE| 描述包
PARAMETER| 用于描述参数
TYPE| 描述类、接口(包括注解类型) 或enum声明

我们自定义了两个自定义注解
eg1:Table用于类、接口或enum声明

```java
@Target(ElementType.TYPE)
public @interface Table {
    
    public String tableName() default "className";
}

```

eg2:@NoDBColumn用于域

```java
@Target(ElementType.FIELD)
public @interface NoDBColumn {
}

```


**@Retention :对自定义注解的生命周期的管理(保留)**

```java

   @Retention(RetentionPolicy.SOURCE)   //源码注解
   @Retention(RetentionPolicy.CLASS)     // 编译时注解
   @Retention(RetentionPolicy.RUNTIME)  // 运行时注解

```

  在运行时去动态获取注解信息，那只能用 RUNTIME 注解；
  在编译时进行一些预处理操作，如生成一些辅助代码，就用 CLASS注解；
  做一些检查性的操作，比如 @Override 和 @SuppressWarnings，则可选用 SOURCE 注解
**@Documented:标记注解，生成javadoc时会包含注解信息**
例如javadoc此类的工具文档化，没有成员.
（idea演示:tool->Generate JavaDoc->填Output directory ->ok）
eg:

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Documented {
}
```
**@Inherited 解释：标记注解，这个允许子注解继承的**
被它修饰的annotation类型用在一个类上面,这个annotation 将被用在子类中
先抛出一个问题：到底子类继承是继承类上的注解还是方法上的注解？
在解析注解中会讲解

### 使用自定义注解
自定义注解的语法要求：
@<注解名>(<成员名1>=<成员值1>,<成员名2> =<成员值2>,....)
参照下面的例子来看

```java

@Description(desc="I am a annotation",author = "aa")
 public String color() {
 
     return “red”;
     }
     
```

### 解析注解

通过反射获取类、方法、成员上的运行时注解信息，从而实现动态控制程序运行的逻辑
演示：
1. @Retention(RetentionPolicy.SOURCE) 什么都打印不出来
2. @Inherited在父类类上加注解，和父类方法上加注解。什么都打印不出来,为什么呢?(没起作用，必须用在类上，在实现接口的时候是没有作用的)实现一个接口的时候是不起任何作用的，必须用在继承上；
3. 改过之后，发现打印出类上的注解，方法上的不显示。（Demo3代码详情，演示）

```java
public class AnnotationTest{
pulic static void main(String args[]){
//使用类加载器加载类
   try{
        Class c=Class.forName("com.ann.test.Child");
        //2.找到类上面的注解
        boolean isExist=c.isAnnotationPresent(Description.class);
          if(isExist){
          //3.拿到注解实例
        Description d=(Description) c.getAnnotation(Description.class);
        System.out.println(d.desc());
             }
         }
         //4.找到方法上的注解
         Method[] ms=c.getMethods();
         for(Method m:ms){
         boolean isMExist=m.isAnnotationPresent(Description.class);
         if(isMExist){
             Description d1=(Description)m.getAnnotation(Description.class);
             System.out.println(d1.desc());
             Method[] ms=c.getMethods();
             for(Method m:ms){
                boolean isMExist=m.isAnnotationPresent(Description.class);
             if(isMExist){
                Description d1=(Description)m.getAnnotation(Description.class);
                System.out.println(d1.desc());
              }
            }
     catch (ClassNotFoundException e) {
     e.printStackTrace();
     }
 
```
•	使用类加载器加载类，获取类对象;
•	判断是否存在类注解;
•	取出类注解，做逻辑处理;
•	方法注解类似


