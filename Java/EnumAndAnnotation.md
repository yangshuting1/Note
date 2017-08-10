## 枚举和注解
### 初识
 Java1.5发行版本增加了两个引用类型：1.新的类——“枚举”2.新的接口——“注解”
 枚举类型（enum type）:是由一组固定的常量组成合法值得类型。比如说一年四季;一副牌中的花色;

 eg1：一年四季
 ```java
    public enum Year {
    SPRING,
    SUMMER,
    AUTUMN,
    WINTER
    }
 ```

### 用enum代替int常量
在没有引入枚举之前，表示枚举类型的常用模式是声明一组具名的int常量。
✨科普一下位运算符
演示：Text.class
总结：**int枚举模式**
 1. 安全性和使用方便性较低,你必须确保是int型，保证范围是0和1;
 2. 打印出来的时候只有0和1，没意义;



### 用实例域代替序数
应该给enum添加int域，而不是使用ordinal方法来导出与枚举关联的序数值。

**wrong:**
```java
public enum Fruit {
    APPLE,PEAL,ORANGE;
    public int numberOfFruit(){
    //ordinal()用来返回枚举常量在类型中的数字位置
    return ordinal()+1;
    }
} 
```
**right:**
```java
public enum Fruit{
    APPLE(1), PEAR(2), ORANGE(3);
    private final int number;
    Fruit(int num) {
       number  = num;
    }
    public int numberOfFruit(){
        return number;
    }
}
```

### 用EnumSet代替位域
如果一个枚举类型的元素主要设置在集合中，一般使用int枚举模式，用不同倍数赋予每个常量：
eg:

**wrong:**
```java
public class Text {
    public static final int STYLE_BOLD      = 1 << 0; // 1 加粗    0001
    public static final int STYLE_ITALIC    = 1 << 1; // 2 斜体    0010
    public static final int STYLE_UNDERLINE = 1 << 2; // 4 下划线   0100
    private int styles;
    public void applyStyles(int styles) {
    ...
    }
    //用OR位运算符将几个常量合并到一个集合中
    text.applyStyles(Text.STYLE_BOLD | Text.STYLE_UNDERLINE);
}

```
 **位域：**用OR位运算符将几个常量合并到一个集合中
java.util包提供了EnumSet类来有效的表示从单个枚举中提取的多个值的多个集合，实现了Set接口
**用枚举代替位域:**
```java
class Text{
    public enum Style{BOLE,ITALIC,UNDERLINE}
    public void applyStyles(Set<Style> styles){
      }
      //将EnumSet实例传递给applyStyles方法
   text.applyStyles(EnumSet.of(Style.BOLD,Style.UNDERLINE));
}
```
### 用enumMap代替序数索引
假设有一个香草的数组，表示一座花园中的植物，
```java
public class Herb {
    public enum Type { ANNUAL, PERENNIAL, BIENNIAL };
    private final String name;
    private final Type type;

    Herb(String name, Type type) {
        this.name = name;
        this.type = type;
    }

    @Override public String toString() {
        return name;
    }
``` 
    
你想要按照类型（一年生、多年生或者两年生植物）进行组织之后再将这些植物列出来。需要构建三个集合，每种类型一个，并且遍历整座花园，将每种香草放到相应的集合中。有些程序员会将这些集合放到一个按照类型的序数进行索引的数组来实现这一点。
com.demo.three.Herb演示
    
```java
    public static void main(String args[]){
    //构造数据
    Herb[] garden= { new Herb("Basil", Type.ANNUAL), 
    new Herb("Carroway", Type.BIENNIAL),
    new Herb("Dill", Type.ANNUAL),        
    new Herb("Lavendar", Type.PERENNIAL), 
    new Herb("Parsley", Type.BIENNIAL),        
    new Herb("Rosemary", Type.PERENNIAL) };
         //构建三个集合
    Set<Herb>[] herbsByType = (Set<Herb>[])new Set[Herb.Type.values().length];
    
    //每种类型一个
    for(int i=0; i<herbsByType.length; i++) {
    herbsByType[i] = new HashSet<Herb>();
    }
    
    //遍历整座花园
    for(Herb h : garden) {
         herbsByType[h.type.ordinal()].add(h);
     }
    for(int i=0; i<herbsByType.length; i++) {
    System.out.println(Herb.Type.values()[i]+":"+herbsByType[i]);
    }
}
```

这种方法的确可行，但是隐藏着许多问题。因为数组不能与泛型兼容。程序需要进行未受检的转换，并且不能正确无误地进行编译。因为数组不知道它的索引代表着什么，你必须手工标注这些索引的输出。但是这种方法最严重的问题在于，当你访问一个按照枚举的序数进行索引的数组时，使用正确的int值就是你的职责了；int不能提供枚举的类型安全。你如果使用了错误的值，程序就会悄然地完成错误的工作，或者幸运的话就会抛出ArrayIndexOutOfBoundException异常。


java.util.EnumMap是一种非常快速的Map实现专门用于枚举的键。
com.demo.four演示：
```java
Map<Herb.Type, Set<Herb>> herbsByType = new EnumMap<Herb.Type, Set<Herb>>(Herb.Type.class);

for(Herb.Type t : Herb.Type.values())
    herbsByType.put(t, new HashSet<Herb>());

for(Herb h : garden)
    herbsByType.get(h.type).add(h);

System.out.println(herbsByType);
```
这段程序更简短，更清楚，也更安全，运行速度方面可以与使用序数的程序相媲美。它没有不安全的转换；不必手工标注出这些索引的输出，因为映射键知道如何将自身翻译成可打印的字符串的枚举；计算数组索引时也不可能出错。EnumMap在运行速度方面之所以能与通过序数索引的数组相媲美，是因为EnumMap在内部使用了这种数组。但是它对程序员隐藏了这种思想细节，集Map的丰富功能和类型安全与数组的快速于一身。注意EnumMap构造器采用键类型的Class对象：这是一个有限制的类型令牌（bounded type token），它提供了运行时的泛型信息。

### 用接口模拟可伸缩的枚举

枚举类型是不可扩展的，但是接口类型是可扩展的。使用接口，可以模拟可伸缩的枚举。
实现接口Operation，里面只有一个apply方法
```java
public interface Operation {
    double apply(double x, double y);
}
```

基本的运算:
```java
public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) {return x + y;}
    },
    MINUS("-") {
        public double apply(double x, double y) {return x - y;}
    },
    TIMES("*") {
        public double apply(double x, double y) {return x - y;}
    },
    DIVIDE("/") {
        public double apply(double x, double y) {return x - y;}
    };
    private final String symbol;
    BasicOperation(String symbol) {
        this.symbol = symbol;
    }
    public String toString() {
        return symbol;
    }
    
}
```
扩展该计算器：
```java
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {return Math.pow(x, y);}
    },
    REMAINDER("%") {
        public double apply(double x, double y) {return x % y;}
    };
    
    private final String symbol;
    private ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
    public String toString() {
        return symbol;
    }
}
```
使用：
```java
public class Main {

    private static <T extends Enum<T> & Operation> void test(Class<T> opSet, double x, double y) {
        
        for(Operation op : opSet.getEnumConstants())
            System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
    }
    
    public static void main(String[] args) {
        double x = 2.0;
        double y = 4.0;
        test(ExtendedOperation.class, x, y);
    }

}
```
接口模拟可伸缩枚举的不足：无法实现从一个枚举继承到另一个枚举，所以有些公共的功能是在每个枚举类中重复的。如果公共的功能比较多，将它封装到一个辅助类或静态辅助方法中，来避免代码的复制工作。

### 注解优先于命名模式
一般命名模式表明有些程序元素需要通过某种工具进行特殊处理。
例如：JUnit测试框架要求用户一定要用test作为测试方法名称的开头。这种方法有几个严重的缺陷：
1. 文字拼写错误会导致失败，但是没有任何提示。比如，把方法名称命名写成tsetSafety而不是testSafety时。JUnit不错报错，但是也不会执行测试。
2. 无法确保它们只用于相应的程序元素上。比如，将每个类命名为testSafety，是希望JUnit会自动的测试这个类的所有方法，而不管什么方法名称， JUnit不会报错，也不会执行测试，因为它针对的是方法名，不是类名。
3. 它们没有提供将参数值与程序元素关联的好方法。比如，想要支持一种测试类别，它只在抛出特殊异常时才会成功。异常类型本质是测试的一个参数，如果命名类不存在，或者不是一个异常，你只有通过运行后才能发现。

对于注解类型可以很好的解决命名模式的缺点。不过只能应用于无参static方法中：
```java
		public class Sample{  
		   @Test 
		   public static void m1(){}  
		   public static void m2(){}  
		   @Test 
		   public static void m3({
		    throw new RuntimeException("boom")}   
		        public static void m4(){}   
		   @Test 
		   public void m5(){}  
		   public static void m6(){}  
		   @Test 
		   public static void m7({ 
		   throw new RuntimeException("carsh")}  
		        public static void m8(){}   
		}  
```
Sample类有8个静态方法，其中是个是被注解为测试，这4个中有2个抛出异常，m3和m7

还有一个是实例方法m5，不属于有效注解。总之，4个测试方法一项通过，两项失败，另外一项无效。

### 坚持使用Override注解
对于传统程序员，注解里面最重要的就是Override注解了。这里的注解，都是指只能用在方法中的声明，
她表示被注解的方法用于覆盖了父类的一个声明，如果坚持使用这个注解，可以防止一大类的非法错误
```java
public class Bigram{
	private final char first；
	private final char second；
	public Bigram （char first ，char second）{
		this.first=first;
		this.second=second;
	}
	
	public boolean equals(Bigram b){
		return b.first==first && b.second==second;
	}
	public int hashCode（）{
		return 31*first+second;
	}

	public static void main (String args[]){
		Set<Bigram> s=new HashSet<Bigram>();
		for(int i=0;i<10;i++){
		  for(char ch='a',ch<='z',ch++){
		  s.add(new Bigram(ch,ch));
		  System.out.println(s.size);
		  }
		}	
	}
}
```
将26个字母反复的添加到集合里，随后打印集合的大小，因为集合不能重复，会发现他打印出来的是26.
Bigram类创建者原本想要覆盖equals方法，同时覆盖hashCode方法
但是却不幸的重载了equals方法，（equals必须参数传入Object类），这个equals方法测试对象的同一性，就像==操作符一样了。这就是得不到想要结果的原因。
在现在的IDE中都提供了Override注解检查功能，这就可以让你在编译时候就会发现这个方法是有问题的。
```java
@override
public boolean equals(Bigram b){
	return b.first==first && b.second==second;
	}	
```
在你想要覆盖超类声明的每个方法声明中使用Override注解。
例外：在编写一个没有标注为抽象的类，并确信它覆盖了抽象的方法，就没有必要讲注解放到该方法上。
### 用标记接口定义类型
**标记接口：**是没有包含方法声明的接口，而指明一个类实现了某种属性的接口。
标记接口的优势：
1. 标记接口定义的类型是由被标记类的实例实现的；标记注解则没有这样的类型。
2. 可以被更精确地进行锁定。如果注解类型利用@Target（ElementType.TYPE）声明，它就可以被应用到任何类或者接口，假设有一个标记只是适用于特殊的接口实现，但它却可以被应用到类，如果定义成一个标记接口，就可以用它将唯一的接口扩展成适用的接口。
**什么时候用标记注解？**
想要标记程序元素而非类和接口，考虑到未来可能要给标记添加更多的信息，或者标记要适合于已经广泛使用了注解类型的框架。
**什么时候用标记接口？**
定义一个任何新方法都不会与之关联的类型。

 


