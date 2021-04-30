# 过程三：Initialization（初始化）阶段

## 1. static与final的搭配问题

**说明**：使用static+ final修饰的字段的显式赋值的操作，到底是在哪个阶段进行的赋值？ 

- 情况1：在链接阶段的准备环节赋值

- 情况2：在初始化阶段*<*clinit*>*()中赋值 

**结论**： 在链接阶段的准备环节赋值的情况： 

- 对于基本数据类型的字段来说，如果使用static final修饰，则显式赋值(直接赋值常量，而非调用方法通常是在链接阶段的准备环节进行 

- 对于String来说，如果使用字面量的方式赋值，使用static final修饰的话，则显式赋值通常是在链接阶段的准备环节进行 
- 在初始化阶段*<*clinit*>*()中赋值的情况： 排除上述的在准备环节赋值的情况之外的情况。

**最终结论**：使用static+final修饰，且显示赋值中不涉及到方法或构造器调用的基本数据类到或String类型的显式财值，是在链接阶段的准备环节进行。

```java
public static final int INT_CONSTANT = 10;                                // 在链接阶段的准备环节赋值
public static final int NUM1 = new Random().nextInt(10);                  // 在初始化阶段clinit>()中赋值
public static int a = 1;                                                  // 在初始化阶段<clinit>()中赋值

public static final Integer INTEGER_CONSTANT1 = Integer.valueOf(100);     // 在初始化阶段<clinit>()中赋值
public static Integer INTEGER_CONSTANT2 = Integer.valueOf(100);           // 在初始化阶段<clinit>()中概值

public static final String s0 = "helloworld0";                            // 在链接阶段的准备环节赋值
public static final String s1 = new String("helloworld1");                // 在初始化阶段<clinit>()中赋值
public static String s2 = "hellowrold2";                                  // 在初始化阶段<clinit>()中赋值
```

## 2. *<*clinit*>*()的线程安全性

对于*<*clinit*>*()方法的调用，也就是类的初始化，虚拟机会在内部确保其多线程环境中的安全性。

虚拟机会保证一个类的()方法在多线程环境中被正确地加锁、同步，如果多个线程同时去初始化一个类，那么只会有一个线程去执行这个类的*<*clinit*>*()()方法，其他线程都需要阻塞等待，直到活动线程执行*<*clinit*>*()()方法完毕。

正是因为<font color='red'>函数*<*clinit*>*()()带锁线程安全的</font>，因此，如果在一个类的*<*clinit*>*()()方法中有耗时很长的操作，就可能造成多个线程阻塞，引发死锁。并且这种死锁是很难发现的，因为看起来它们并没有可用的锁信息。

如果之前的线程成功加载了类，则等在队列中的线程就没有机会再执行*<*clinit*>*()()方法了。那么，当需要使用这个类时，虚拟机会直接返回给它已经准备好的信息。

## 3. 类的初始化情况：主动使用vs被动使用

Java程序对类的使用分为两种：主动使用和被动使用。

**主动使用**

Class只有在必须要首次使用的时候才会被装载，Java虚拟机不会无条件地装载Class类型。Java虚拟机规定，一个类或接口在初次使用前，必须要进行初始化。这里指的“使用”，是指主动使用，主动使用只有下列几种情况：（即：如果出现如下的情况，则会对类进行初始化操作。而初始化操作之前的加载、验证、准备已经完成。

1. ==实例化==：当创建一个类的实例时，比如使用new关键字，或者通过反射、克隆、反序列化。

   ```java
   /**
    * 反序列化
    */
   Class Order implements Serializable {
       static {
           System.out.println("Order类的初始化");
       }
   }
   
   public void test() {
       ObjectOutputStream oos = null;
       ObjectInputStream ois = null;
       try {
           // 序列化
           oos = new ObjectOutputStream(new FileOutputStream("order.dat"));
           oos.writeObject(new Order());
           // 反序列化
           ois = new ObjectInputStream(new FileOutputStream("order.dat"));
           Order order = ois.readObject();
       }
       catch (IOException e){
           e.printStackTrace();
       }
       catch (ClassNotFoundException e){
           e.printStackTrace();
       }
       finally {
           try {
               if (oos != null) {
                   oos.close();
               }
               if (ois != null) {
                   ois.close();
               }
           }
           catch (IOException e){
               e.printStackTrace();
           }
       }
   }
   ```

2. ==静态方法==：当调用类的静态方法时，即当使用了字节码invokestatic指令。

3. ==静态字段==：当使用类、接口的静态字段时（final修饰特殊考虑），比如，使用getstatic或者putstatic指令。（对应访问变量、赋值变量操作）

   ```java
   public class ActiveUse {
   	@Test
       public void test() {
           System.out.println(User.num);
       }
   }
   
   class User {
       static {
           System.out.println("User类的初始化");
       }
       public static final int num = 1;
   }
   ```

4. ==反射==：当使用java.lang.reflect包中的方法反射类的方法时。比如：Class.forName("com.atguigu.java.Test")

5. ==继承==：当初始化子类时，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。

   > 当Java虚拟机初始化一个类时，要求它的所有父类都已经被初始化，但是这条规则并不适用于接口。
   >
   > - 在初始化一个类时，并不会先初始化它所实现的接口
   > - 在初始化一个接口时，并不会先初始化它的父接口
   > - 因此，一个父接口并不会因为它的子接口或者实现类的初始化而初始化。只有当程序首次使用特定接口的静态字段时，才会导致该接口的初始化。

6. ==default方法==：如果一个接口定义了default方法，那么直接实现或者间接实现该接口的类的初始化，该接口要在其之前被初始化。

   ```java
   interface Compare {
   	public static final Thread t = new Thread() {
           {
               System.out.println("Compare接口的初始化");
           }
       }   
   }
   ```

7. ==main方法==：当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类。

   > VM启动的时候通过引导类加载器加载一个初始类。这个类在调用public static void main(String[])方法之前被链接和初始化。这个方法的执行将依次导致所需的类的加载，链接和初始化。

8. ==MethodHandle==：当初次调用MethodHandle实例时，初始化该MethodHandle指向的方法所在的类。（涉及解析REF getStatic、REF_putStatic、REF invokeStatic方法句柄对应的类）

**被动使用**

除了以上的情况属于主动使用，其他的情况均属于被动使用。<font color='red'>被动使用不会引起类的初始化。</font>

也就是说：<font color='red'>并不是在代码中出现的类，就一定会被加载或者初始化。</font>如果不符合主动使用的条件，类就不会初始化。

1. ==静态字段==：当通过子类引用父类的静态变量，不会导致子类初始化，只有真正声明这个字段的类才会被初始化。

   ```java
   public class PassiveUse {
    	@Test
       public void test() {
           System.out.println(Child.num);
       }
   }
   
   class Child extends Parent {
       static {
           System.out.println("Child类的初始化");
       }
   }
   
   class Parent {
       static {
           System.out.println("Parent类的初始化");
       }
       
       public static int num = 1;
   }
   ```

2. ==数组定义==：通过数组定义类引用，不会触发此类的初始化

   ```java
   Parent[] parents= new Parent[10];
   System.out.println(parents.getClass()); 
   // new的话才会初始化
   parents[0] = new Parent();
   ```

3. ==引用常量==：引用常量不会触发此类或接口的初始化。因为常量在链接阶段就已经被显式赋值了。

   ```java
   public class PassiveUse {
       public static void main(String[] args) {
           System.out.println(Serival.num);
           // 但引用其他类的话还是会初始化
           System.out.println(Serival.num2);
       }
   }
   
   interface Serival {
       public static final Thread t = new Thread() {
           {
               System.out.println("Serival初始化");
           }
       };
   
       public static int num = 10; 
       public static final int num2 = new Random().nextInt(10);
   }
   ```

4. ==loadClass方法==：调用ClassLoader类的loadClass()方法加载一个类，并不是对类的主动使用，不会导致类的初始化。

   ```java
   Class clazz = ClassLoader.getSystemClassLoader().loadClass("com.test.java.Person");
   ```



**扩展**

> -XX:+TraceClassLoading：追踪打印类的加载信息

