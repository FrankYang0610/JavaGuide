# JavaGuide
#### FRANK PUFFIN, JUL 2024

## 一、Java运行环境
Java的运行环境（Java Runtime Environment, JRE）是支持Java程序运行的必要组件。它包含Java虚拟机（JVM）、核心类库和其他支持文件。

* **Java虚拟机（JVM）** 是Java程序的运行时**引擎**，它负责将Java字节码（`.class`文件）翻译成机器码，并在计算机上执行。 JVM使Java具有跨平台的特性，只要有相应平台的JVM，Java程序就能运行在不同的操作系统上；
* **核心类库** 提供Java程序运行所需的基本类，如数据结构、网络处理、文件I/O、图形界面等。这些类库通常以包（package）的形式组织，例如`java.lang`、`java.util`、`java.io`等；
* **其他支持文件** 如动态链接库（如`.dll`文件等）用于实现一些底层功能。

JVM和核心类库的关系在**一定程度上**类似C++编译器和STL的关系。

### 1.1 JVM与操作系统的交互
* **内存管理：** JVM通过系统调用向操作系统申请和释放内存。例如，在Linux系统上，可能使用`mmap()`和`munmap()`系统调用。
* **文件操作：** JVM通过操作系统的文件系统API读写文件，如`open()`, `read()`, `write()`, `close()`等。
* **网络操作：** 使用操作系统提供的网络套接字API进行网络通信。
* **线程管理：** JVM的线程通常是映射到操作系统的原生线程上，使用如`pthread_create()`等系统调用。
* **本地方法调用（JNI）：** Java Native Interface (JNI) 允许Java代码调用其他语言（如C/C++）编写的本地代码。
* JVM需要管理本地方法的加载、卸载，以及Java和本地代码之间的数据转换。
* JVM的垃圾回收器需要与操作系统交互以管理内存。 某些垃圾回收算法（如并发标记清除）可能需要操作系统的支持来实现线程挂起和恢复。
* JVM需要从文件系统加载类文件，这涉及到与文件系统的交互。自定义类加载器可能涉及网络操作，从远程服务器加载类。
* 即时编译器（JIT）将字节码编译为本地机器码，这个过程涉及到与CPU架构的直接交互。JIT可能需要使用特定的CPU指令集来优化代码执行。
* JVM提供了如JMX（Java Management Extensions）这样的工具，允许外部程序监控和管理JVM的运行状态。
* 现代JVM实现可能直接使用特定的硬件特性，如CPU的`SIMD`指令集，以提高性能。
* JVM的`synchronized`关键字和`java.util.concurrent`包中的同步工具通常是基于操作系统的底层同步原语实现的。
* JVM需要与操作系统交互以获取系统时间，实现`Thread.sleep()`等功能。
* JVM可能需要与操作系统的动态链接器交互，特别是在加载本地库时。

### 1.2 Java程序的编译和运行
* **源代码编写：** 开发者使用`.java`文件编写Java源代码。
* **编译：** **使用Java编译器（通常是javac）将源代码编译成字节码（bytecode）。** 这个过程包括几个阶段：
  * 词法分析：将源代码分解成一系列标记（tokens）。识别关键字、标识符、字面量等。
  * 语法分析：根据Java语言规范检查代码的语法结构。构建抽象语法树（AST）。
  * 语义分析：进行类型检查。检查变量声明和使用。进行常量折叠等简单优化。
  * **字节码生成：将AST转换为Java字节码。生成`.class`文件。**
* **字节码验证：** JVM在加载类时会验证字节码，确保其符合JVM规范，不会危及JVM的安全。
* **类加载：** 当Java程序运行时，JVM的类加载器会动态加载、链接和初始化类。
* **即时编译（JIT）：** JVM会监控代码的执行，识别热点代码；对频繁执行的代码进行即时编译，**将字节码转换为本地机器码**。
* **执行：** JVM解释执行字节码或运行编译后的本地代码。

**【Java程序的最终状态】** Java代码最终在JVM (Java虚拟机)上运行。JVM本身是作为操作系统的一个进程运行的。JVM可以直接解释执行字节码。JIT也会在运行时将热点代码（频繁执行的代码）编译成本地机器码。最准确的说法是，**Java程序不会在编译时直接被编译为汇编或机器码。但在运行时，部分Java代码（尤其是热点代码）会被JIT编译器编译成汇编代码，然后转换为机器码以提高性能。** 

### 1.3 JVM的虚拟内存结构
JVM (Java虚拟机) 的虚拟内存结构是Java程序运行的基础。它定义了Java程序在运行时如何使用内存。**Java程序直接与JVM交互，使用JVM提供的内存结构。** 而操作系统仅为JVM分配物理内存、管理JVM进程、提供底层系统调用支持，不涉及具体Java程序。JVM内存结构包括：
* 方法区（Method Area）： 
  * 存储类信息、常量、静态变量、即时编译器编译后的代码等。
  * 在JDK 8之前，这个区域也被称为"永久代"（PermGen）； **JDK 8及以后，被元空间（Metaspace）取代，并使用本地内存**。
* 堆（Heap）：
  * 和操作系统虚拟内存中的堆相似，都用于**动态内存分配**。
  * 存储对象实例和数组。
  * 是垃圾收集器管理的主要区域。
  * 可以物理上不连续，但逻辑上应该连续。
  * 通常分为新生代和老年代。
* 虚拟机栈（VM Stack）：
  * 每个线程都有一个私有的虚拟机栈。
  * 存储局部变量表、操作数栈、动态链接、方法出口等信息。
  * 每个方法调用都会创建一个栈帧（Stack Frame）。
* 本地方法栈（Native Method Stack）：
  * 为虚拟机使用到的本地（Native）方法服务。
  * 有些虚拟机（如 HotSpot）将本地方法栈和虚拟机栈合并实现。
* 程序计数器（Program Counter Register）：
  * 记录当前线程执行的字节码指令地址。
  * 每个线程都有一个独立的程序计数器。
  * 是唯一一个在JVM规范中没有规定任何OutOfMemoryError情况的区域。
* 直接内存（Direct Memory）：
  * 不是虚拟机运行时数据区的一部分，也不是 Java 虚拟机规范中定义的内存区域。
  * 在JDK 1.4中新加入了NIO，引入了一种基于通道（Channel）与缓冲区（Buffer）的 I/O 方式，它可以使用Native函数库直接分配堆外内存。

JVM内存结构示意图如下：
```
+-------------------+
|    Method Area    |
|    (Metaspace)    |
+-------------------+
|                   |
|       Heap        |
|                   |
+-------------------+
| VM Stack | Native |
|          | Method |
|          | Stack  |
+-------------------+
| Program Counter   |
+-------------------+
|  Direct Memory    |
+-------------------+
```

### 1.4 `native`关键字
`native`这个关键字对理解实现至关重要。它表示该方法是在平台特定的代码中实现的，通常用C或C++等其他编程语言编写。

例如在`public static native Thread currentThread()`方法中，当Java虚拟机(JVM)启动时，它会加载包含本地方法实现的本地库，`currentThread()`的本地实现直接与底层操作系统交互，以获取有关当前线程的信息。以下是本地代码可能的简化伪代码表示：

```c++
JNIEXPORT jobject JNICALL Java_java_lang_Thread_currentThread(JNIEnv *env, jclass cls) {
    // 获取本地线程 ID
    native_thread_t native_thread = GET_CURRENT_NATIVE_THREAD();
    
    // 查找对应的 Java Thread 对象
    jobject thread_obj = FIND_JAVA_THREAD(native_thread);
    
    return thread_obj;
}
```

### 1.5 JVM语言
JVM语言是指能够在Java虚拟机(JVM)上运行的编程语言。**这些语言的源代码被编译成Java字节码**，然后可以在任何支持JVM的平台上运行。

常见的JVM语言包括：
* Java：最原始和广泛使用的JVM语言。
* Kotlin：由JetBrains开发，被Google推荐用于Android开发。
* Scala：结合了面向对象和函数式编程特性。
* Groovy：动态类型语言，常用于脚本编写。
* Clojure：Lisp方言，强调函数式编程。
* JRuby：Ruby语言的JVM实现。
* Jython：Python语言的JVM实现。

## 二、基本数据类型
### 2.1 基本数据类型
* `byte`(1 byte)，`short`(2 byte)，`int`(4 byte)，`long`(8 byte)。
* `float`(4 byte)，`double`(8 byte)。
* `char`(2 byte, Unicode)。
* `boolean`(JVM无明确定义，取值只可以是`true`或`false`)。
* `null`是一个特殊的值，用于表示引用类型变量没有指向任何对象。`null`是Java的关键字，可以被赋值给任何引用类型变量（对象、数组、接口等）。
* 字符串（`String`）是一个类，不是基本类型。
```java
int i;
float f = 3.14f;
char c = 'A';
boolean b = true;
```
### 2.2 数组
* 静态初始化
    ```java
    int[] numbers = {1, 2, 3, 4, 5};
    String[] names = {"Alice", "Bob", "Charlie"};
    ```
* 动态初始化（参见`new`）
    ```java
    int[] numbers = new int[5]; // 声明一个长度为5的数组
    numbers[0] = 1;
    numbers[1] = 2;
    ```
* 内置属性和方法：`length`用于获取数组的长度；`clone()`方法用于创建一个数组的浅复制（创建一个新的数组，包含与原数组相同的元素）`。
* 遍历数组（参见循环）
    ```java
    // 使用传统for循环
    for (int i = 0; i < numbers.length; i++) {
        //
    }
    
    // 使用range for循环
    for (int number : numbers) {
        //
    }
    ```
### 2.3 类型转换
* **隐式类型转换是指编译器自动完成的类型转换。** 通常发生在将一个较小范围的类型赋值给一个较大范围的类型时，不需要显式地进行转换操作。这种转换是安全的，不会导致数据丢失。
    ```java
    int num = 100;
    long bigNum = num; // int自动转换为long
    float fNum = num;  // int自动转换为float
    ```
* **显式类型转换是需要程序员手动进行的类型转换**，通常在将一个较大范围的类型赋值给一个较小范围的类型时使用。显式类型转换可能会导致数据丢失或精度降低，因此要谨慎使用。
    ```java
    // 语法： targetType variableName = (targetType) value;
    double d = 9.78;
    int i = (int) d; // 强制将double转换为int
    ```

### 2.4 引用类型
* 当你将基本数据类型的量作为参数传递给函数时，传递的是这些值的副本。对这些副本进行的修改不会影响原始变量。
* 当你要将引用数据类型（对象、数组）作为参数传递给函数时，**传递的是引用，而不是副本。** 使用`final`关键字修饰参数可以避免原始对象被修改。
    ```java
    public void exampleMethod(final Object o) {
        // x = 20; // 这会导致编译错误，因为x是final的
        System.out.println(o);
    }
    ```

### 2.5 指针和地址？
在Java中**没有**像C或C++那样的显式指针和内存地址操作。Java是设计为内存安全的语言，旨在防止诸如内存泄漏和指针操作错误等常见问题。虽然你不能直接访问对象的内存地址，但Java提供了`System.identityHashCode()`方法，可以获得对象的HashCode，该HashCode在对象的生命周期内是唯一的，尽管它们并不直接对应实际内存地址。

## 三、运算符
```java
int sum = 10 + 5;
int diff = 10 - 5;
int product = 10 * 5;
int quotient = 10 / 5;
int remainder = 10 % 5;
boolean isEqual = (10 == 5);
boolean isNotEqual = (10 != 5);
boolean isGreater = (10 > 5);
boolean isLess = (10 < 5);
boolean isGreaterOrEqual = (10 >= 5);
boolean isLessOrEqual = (10 <= 5);
boolean and = (true && false);
boolean or = (true || false);
boolean not = !true;
```

## 四、输出
**使用`System.out.println()`输出。**
* `System`是一个Java标准类，位于`java.lang`包中。它包含了一些静态字段和方法，用于访问系统资源。
* `out`是`System`类的一个静态字段，是一个`PrintStream`对象。`PrintStream`是一个可以方便地打印各种数据值表示形式的类（如`int`、`string`、对象等）。`System.out`默认连接到控制台。
* `println`是`PrintStream`类的一个方法，负责在控制台输出文本，并在最后加上一个新行符（换行符）。
* `PrintStream`类中的`println`方法有多种重载形式，可以打印不同类型的数据：
```java
public void println();
public void println(boolean x);
public void println(char x);
public void println(char[] x);
public void println(double x);
public void println(float x);
public void println(int x);
public void println(long x);
public void println(Object x);
public void println(String x);
```
Java 支持使用`+`操作符来连接字符串和其他数据类型（形成“流”）：
```java
System.out.println("Name: " + name + ", Age: " + age);
```
**【为什么】** 在 Java 中，使用`+`操作符进行字符串连接时，不仅可以连接多个字符串，还可以连接其他数据类型。**这是因为Java编译器会自动将非字符串类型转换为字符串**。这种转换过程称为字符串化（string conversion）。

其他输出函数包括：
* `System.out.print`: 打印输出，不换行。
* `System.out.printf`: 格式化打印输出，类似于 C 语言的 printf。
* `System.err.print`: 将输出发送到标准错误流，不换行。
* `System.err.println`: 将输出发送到标准错误流，并换行。
* `System.err.printf`: 将格式化输出发送到标准错误流。

## 五、输入
`Scanner`是Java中最常用的输入类之一，提供了从控制台、文件、字符串等各种输入源**读取数据**的功能。
```java
import java.util.Scanner;

public class ScannerExample {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        System.out.print("Enter your name: ");
        String name = scanner.nextLine(); // 接受输入
        System.out.println("Hello, " + name + "!");
        scanner.close();
    }
}
```
**【为什么`System.in`不能直接处理输入】** `System.in`直接提供的读取方法是`read()`，**它一次读取一个字节**，并返回一个整数值（0到255之间的字节值）。这对于读取文本输入非常不方便，因为它不会自动处理字符和字符串。

## 六、控制结构
```java
if (number > 0) {
    System.out.println("Number is positive.");
} else if (number < 0) {
    System.out.println("Number is negative.");
} else {
    System.out.println("Number is zero.");
}
```
```java
switch (day) {
    case 1:
        System.out.println("Monday");
        break;
    case 2:
        System.out.println("Tuesday");
        break;
    default:
        System.out.println("Invalid day");
}
```
Java的`if`语句内部可以赋值：`if ((a = 5) == 5) { ... }`。

## 七、循环
```java
for (int i = 0; i < 10; i++) {
    System.out.println(i);
}
```
```java
for (String fruit : fruits) {
    System.out.println(fruit);
}
```
```java
int i = 0;
while (i < 10) {
    System.out.println(i);
    i++; // i++与++i在Java中意义与C++一致
}
```
```java
int i = 0;
do {
    System.out.println(i);
    i++;
} while (i < 10);
```

## 八、异常处理
异常处理模型：
```java
try {
    int result = 10 / 0;
} catch (ArithmeticException e) {
    System.out.println("Cannot divide by zero!");
} finally {
    System.out.println("This block is always executed.");
}
```

### 8.1 内置异常列表
Java中的异常大致可以分为两类：Checked Exceptions（受检异常）和Unchecked Exceptions（非受检异常）。Checked Exceptions必须在代码中显式地捕获或声明，而Unchecked Exceptions则由程序运行时抛出，不需要显式地捕获或声明。
* **Checked Exceptions：** 之所以被称为“受检”异常，主要是因为编译器在编译期间会对这些异常进行检查和强制处理。具体来说，Java编译器要求程序必须显式地处理这些异常，要么通过捕获（`catch`）它们，要么通过声明（`throws`）将它们抛出。否则，代码将无法通过编译。
  * `ClassNotFoundException`: 类未找到异常。
  * `InstantiationException`: 实例化异常。
  * `IllegalAccessException`: 非法访问异常。
  * `NoSuchMethodException`: 方法未找到异常（一般出现在反射机制中）。
  * `InvocationTargetException`: 调用目标异常。
  * `InterruptedException`: 线程中断异常。
  * `IOException`: 输入输出异常。
    * `FileNotFoundException`: 文件未找到异常。
    * `EOFException`: 文件结束异常。
    * `SocketException`: 套接字异常。
    * `UnknownHostException`: 未知主机异常。
    * `MalformedURLException`: URL格式错误异常。
    * `ConnectException`: 连接异常。
  * `SQLException`: SQL异常。
* **Unchecked Exceptions：** 与受检异常不同，非受检异常在编译期间不会被强制检查。这意味着程序员在编写代码时不必显式地捕获或声明这些异常。然而，这并不意味着它们不重要。**非受检异常通常表示编程错误或不可恢复的条件**（例如，空指针引用、数组越界等）。
  * `RuntimeException`: 运行时异常。
  * `ArithmeticException`: 算术异常。
  * `ArrayIndexOutOfBoundsException`: 数组索引越界异常。
  * `ArrayStoreException`: 数组存储异常。
  * `ClassCastException`: 类转换异常。
  * `IllegalArgumentException`: 非法参数异常。
  * `IllegalStateException`: 非法状态异常。
  * `IndexOutOfBoundsException`: 索引越界异常。
  * `NegativeArraySizeException`: 数组大小为负异常。
  * `NullPointerException`: 空指针异常。
  * `NumberFormatException`: 数字格式异常。
  * `SecurityException`: 安全异常。
  * `UnsupportedOperationException`: 不支持的操作异常。
* Errors：虽然错误（Errors）不是异常（Exceptions），但它们也是Java异常处理机制的一部分，通常表示更严重的问题，程序一般不应该捕获它们。
  * `VirtualMachineError`: 虚拟机错误。
    * `StackOverflowError`: 栈溢出错误。
    * `OutOfMemoryError`: 内存不足错误。
  * `AssertionError`: 断言错误。
    * `LinkageError`: 链接错误。
    * `ClassCircularityError`: 类循环依赖错误。
    * `ClassFormatError`: 类格式错误。
  * `NoClassDefFoundError`: 类定义未找到错误。
  * `UnknownError`: 未知错误。

### 8.2 用户自定义的异常
* **创建创建受检异常（Checked Exception）。** 受检异常需要继承自`Exception`类，通常要求方法声明抛出这些异常。
  ```java
  // 定义一个自定义的受检异常
  public class MyCheckedException extends Exception {
      // 无参构造方法
      public MyCheckedException() {
          super();
      }
  
      // 带消息的构造方法
      public MyCheckedException(String message) {
          super(message);
      }
  
      // 带消息和原因的构造方法
      public MyCheckedException(String message, Throwable cause) {
          super(message, cause);
      }
  
      // 带原因的构造方法
      public MyCheckedException(Throwable cause) {
          super(cause);
      }
  }
  
  // 示例类，演示如何抛出和捕获自定义受检异常
  public class CustomExceptionExample {
      public static void main(String[] args) {
          try {
              // 调用可能抛出自定义受检异常的方法
              throwCheckedException();
          } catch (MyCheckedException e) {
              // 处理自定义受检异常
              System.out.println("Caught MyCheckedException: " + e.getMessage());
          }
      }
  
      // 方法声明抛出自定义受检异常
      public static void throwCheckedException() throws MyCheckedException {
          throw new MyCheckedException("This is a custom checked exception.");
      }
  }
  ```
* **创建非受检异常（Unchecked Exception） 。** 非受检异常需要继承自`RuntimeException`类，不要求方法声明抛出这些异常。
  ```java
  // 定义一个自定义的非受检异常
  public class MyUncheckedException extends RuntimeException {
      // 无参构造方法
      public MyUncheckedException() {
          super();
      }
  
      // 带消息的构造方法
      public MyUncheckedException(String message) {
          super(message);
      }
  
      // 带消息和原因的构造方法
      public MyUncheckedException(String message, Throwable cause) {
          super(message, cause);
      }
  
      // 带原因的构造方法
      public MyUncheckedException(Throwable cause) {
          super(cause);
      }
  }
  
  // 示例类，演示如何抛出和捕获自定义非受检异常
  public class CustomUncheckedExceptionExample {
      public static void main(String[] args) {
          try {
              // 调用可能抛出自定义非受检异常的方法
              throwUncheckedException();
          } catch (MyUncheckedException e) {
              // 处理自定义非受检异常
              System.out.println("Caught MyUncheckedException: " + e.getMessage());
          }
      }
  
      // 方法可以抛出自定义非受检异常，但不需要声明
      public static void throwUncheckedException() {
          throw new MyUncheckedException("This is a custom unchecked exception.");
      }
  }
  ```

## 九、`new`
在 Java 中，`new`是用于创建对象和数组的关键字，而`delete`关键字并不存在，因为Java有**垃圾回收机制**来自动管理内存。
`new`关键字用于在堆内存中分配空间并初始化对象或数组。它返回对象的引用。
```java
public class ArrayExample {
    public static void main(String[] args) {
        // 创建数组
        int[] numbers = new int[5];
        for (int i = 0; i < numbers.length; i++) {
            numbers[i] = i * 10;
        }

        for (int num : numbers) {
            System.out.println(num);
        }
    }
}
```
```java
public class Person {
    String name;
    int age;

    // 构造函数
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public void display() {
        System.out.println("Name: " + name + ", Age: " + age);
    }

    public static void main(String[] args) {
        Person person = new Person("Alice", 30);
        person.display();
    }
}
```

### `new`与不同的引用类型
接口（Interface，见**10.7**）是一个特殊的类型，你不能直接用`new`运算符来创建接口的实例。接口是一种规定了某些方法必须被实现的契约，但它并不提供这些方法的具体实现。因此，你不能直接创建一个接口的实例，**你需要创建一个实现了该接口的类的实例**：
```java
interface MyInterface {
    void doSomething();
}

class MyClass implements MyInterface {
    public void doSomething() {
        System.out.println("Doing something...");
    }
}

MyInterface mi = new MyClass();
mi.doSomething();  // Prints "Doing something..."
```

`MyInterface mi = new MyClass()`和`MyClass mi = new MyClass()`两者都创建了一个`MyClass`的对象，**但是它们引用该对象的方式不同**。

在`MyInterface mi = new MyClass()`这行代码中，我们创建了`MyClass`的一个实例，并将其引用赋给了一个`MyInterface`类型的变量`mi`。这意味着我们可以通过`mi`变量访问`MyClass`实现的所有`MyInterface`接口的方法。**然而，我们不能直接通过`mi`访问`MyClass`的其他方法（即那些没有在`MyInterface`中定义的方法），除非我们进行类型转换**。

## 十、类（包括函数）
类为创建对象提供了模板，而对象是类的实例。
```java
public class Person {
    private String name;
    private int age;

    // 构造函数
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // 一般方法
    public void display() {
        System.out.println("Name: " + name + ", Age: " + age);
    }

    // Getter和Setter
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }

    // 主方法
    public static void main(String[] args) {
        Person person = new Person("Alice", 30);
        person.display();
    }
}
```
### 10.1 访问控制
* `public`：任何地方都可以访问。
* `private`：只有在同一个类内部可以访问。
* `protected`：在同一个包内或通过继承可以访问。
（默认，即不使用修饰符）：在同一个包内可以访问。

### 10.2 主入口
在Java程序中，主入口是`main`方法。`main`方法是Java应用程序的起点，JVM（Java虚拟机）会在启动应用程序时首先调用这个方法。`main`方法必须是`public`的、`static`的，并且返回类型是`void`，其参数是一个`String`类型的数组。

`String[] args`: 这是一个字符串数组，用于接收命令行参数。当运行Java程序时，可以通过命令行向main方法传递参数，这些参数会被存储在这个数组中。

**每个类都可以有一个`main`方法，在程序运行的时候，需要指定一个类作为程序整体的起点。**

### 10.3 特殊类`String`
在Java中，`String`类有一些特殊之处，允许使用字符串字面量来初始化`String`对象。字符串字面量是Java语言本身的一部分，并且Java编译器对它们进行了特殊处理。

当你使用字符串字面量时，Java编译器会在编译时将这些字面量字符串存储在一个称为“字符串池”（string pool）的共享区域中。字符串池是一种优化机制，旨在减少内存使用和提高性能。

* 字符串池是一个保存字符串字面量的固定区域。当你使用字符串字面量时，Java会检查字符串池中是否已经有一个相同的字符串。如果有，编译器会使用池中的字符串对象，而不是创建一个新的对象：
    ```java
    String str1 = "Hello, World!";
    String str2 = "Hello, World!";
    
    // 因为str1和str2引用的是同一个字符串池中的对象，所以它们是相同的对象
    System.out.println(str1 == str2);  // 输出: true
    ```
* 如果你使用`new`关键字来创建字符串对象，则会创建一个新的`String`对象，即使内容相同，也不会使用字符串池中的对象。
    ```java
    String str1 = "Hello, World!";
    String str2 = new String("Hello, World!");
    
    // str1和str2虽然内容相同，但它们是不同的对象
    System.out.println(str1 == str2);  // 输出: false
    ```
* `intern()`方法可以将一个字符串对象添加到字符串池中，并返回字符串池中的对象引用。
    ```java
    String str1 = new String("Hello, World!");
    String str2 = str1.intern();
    
    // str2现在引用的是字符串池中的对象
    System.out.println(str1 == str2);  // 输出: false
    System.out.println(str2 == "Hello, World!");  // 输出: true
    ```

### 10.4 继承（is-a关系）
通过`extends`关键字来实现。继承使得一个类可以继承另一个类的属性和方法。
```java
public class Employee extends Person {
    private double salary;

    public Employee(String name, int age, double salary) {
        super(name, age); // 调用基类的构造函数
        this.salary = salary;
    }

    public void display() {
        super.display(); // 调用基类的方法
        System.out.println("Salary: " + salary);
    }

    public static void main(String[] args) {
        Employee employee = new Employee("Bob", 28, 50000);
        employee.display();
    }
}
```

### 10.5 多态
* 方法重载
    ```java
    public class MathOperations {
        public int add(int a, int b) {
            return a + b;
        }
    
        public double add(double a, double b) {
            return a + b;
        }
    }
    ```
* 方法重写
    ```java
    public class Animal {
        public void makeSound() {
            System.out.println("Animal makes a sound");
        }
    }
    
    public class Dog extends Animal {
        @Override // 方法重写标识符
        public void makeSound() {
            System.out.println("Dog barks");
        }
    }
    ```

* 运算符重载\
  **Java 不支持运算符重载（operator overloading）**。

### 10.6 抽象类
使用`abstract`关键字定义，可以有抽象方法和具体方法。
```java
public abstract class Animal {
    public abstract void makeSound(); // 抽象方法

    public void sleep() {
        System.out.println("Animal is sleeping");
    }
}

public class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Cat meows");
    }
}
```

### 10.7 接口 (has-a关系)
使用`interface`关键字定义，只包含抽象方法（Java 8及以后的版本可以包含默认方法和静态方法）。这一定程度上类似Swift的**Protocol-oriented Programming**。
```java
public interface Movable {
    void move();
}

public class Car implements Movable { // 这里必须用implements
    @Override
    public void move() {
        System.out.println("Car is moving");
    }
}
```

### 10.8 友元的实现
在 Java 中没有直接的友元（friend）机制，这种机制是C++中的一种特性，允许一个类访问另一个类的私有和保护成员。尽管Java没有这种机制，但可以通过设计模式和访问控制来实现类似的功能。

**在下面的语境中，`FriendClass`是要访问`FriendlyClass`私有成员的类。**

* 使用包级私有（默认）访问控制：将相关类放在同一个包中，并使用包级私有（即**不指定任何访问修饰符**）来控制成员的访问。
    ```java
    // 文件：com/example/FriendClass.java
    package com.example;
    
    public class FriendClass {
        void friendMethod() {
            // 可以访问 FriendlyClass 的包级私有成员
            FriendlyClass friendly = new FriendlyClass();
            friendly.packagePrivateMethod();
        }
    }
    
    // 文件：com/example/FriendlyClass.java
    package com.example;
    
    class FriendlyClass {
        void packagePrivateMethod() {
            System.out.println("Package-private method");
        }
    }
    ```
* 使用内部类：
    ```java
    public class OuterClass {
        private String secret = "Secret";
    
        class FriendClass {
            void revealSecret() {
                System.out.println("The secret is: " + secret);
            }
        }
    
        public static void main(String[] args) {
            OuterClass outer = new OuterClass();
            OuterClass.FriendClass friend = outer.new FriendClass(); // 注意此处的语法
            friend.revealSecret();
        }
    }
    ```
* 使用公共方法或接口：
    ```java
    public class FriendlyClass {
        private String secret = "Secret";
    
        public String getSecret(FriendClass friend) {
            // 检查friend是否有权限访问
            if (friend.hasPermission()) {
                return secret;
            } else {
                throw new SecurityException("No permission to access secret");
            }
        }
    }
    
    public class FriendClass {
        private boolean hasPermission = true;
    
        boolean hasPermission() {
            return hasPermission;
        }
    
        public void revealSecret(FriendlyClass friendly) {
            System.out.println("The secret is: " + friendly.getSecret(this));
        }
    
        public static void main(String[] args) {
            FriendlyClass friendly = new FriendlyClass();
            FriendClass friend = new FriendClass();
            friend.revealSecret(friendly);
        }
    }
    ```
* 使用反射（不推荐）：尽管可以使用反射来访问私有成员，但这种方法通常不推荐，因为**它破坏了封装性和安全性**。
    ```java
    import java.lang.reflect.Field;
    
    public class FriendlyClass {
        private String secret = "Secret";
    }
    
    public class FriendClass {
        public void revealSecret(FriendlyClass friendly) {
            try {
                Field field = FriendlyClass.class.getDeclaredField("secret");
                field.setAccessible(true);
                System.out.println("The secret is: " + field.get(friendly));
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    
        public static void main(String[] args) {
            FriendlyClass friendly = new FriendlyClass();
            FriendClass friend = new FriendClass();
            friend.revealSecret(friendly);
        }
    }
    ```

### 10.9 类的类型转换
* **向上转型是指将派生类对象转换为基类类型。这种类型转换是自动的**，不需要显式的类型转换操作符。向上转型通常用于多态性（polymorphism），因为基类引用可以指向任何子类对象。
    ```java
    class Animal {
        public void makeSound() {
            System.out.println("Animal sound");
        }
    }
    
    class Dog extends Animal {
        @Override
        public void makeSound() {
            System.out.println("Bark");
        }
    }
    
    public class Main {
        public static void main(String[] args) {
            Dog dog = new Dog();
            Animal animal = dog; // Upcasting
            animal.makeSound(); // 输出 "Bark"
        }
    }
    ```
* **向下转型是指将基类对象转换为派生类类型**。向下转型需要显式的类型转换操作符，并且在运行时可能会抛出`ClassCastException`，因此需要特别小心。
    ```java
    class Animal {
        public void makeSound() {
            System.out.println("Animal sound");
        }
    }
    
    class Dog extends Animal {
        @Override
        public void makeSound() {
            System.out.println("Bark");
        }
    
        public void fetch() {
            System.out.println("Fetch!");
        }
    }
    
    public class Main {
        public static void main(String[] args) {
            Animal animal = new Dog(); // Upcasting
            animal.makeSound(); // 输出 "Bark"
    
            if (animal instanceof Dog) {
                Dog dog = (Dog) animal; // Downcasting
                dog.fetch(); // 输出 "Fetch!"
            }
        }
    }
    ```
* 在进行向下转型之前，通常需要使用`instanceof`操作符来检查对象是否是特定子类的实例，以避免`ClassCastException`：
    ```java
    if (animal instanceof Dog) {
        Dog dog = (Dog) animal;
        // 安全的向下转型
    }
    ```
**【为什么向下转型不安全】** 
* **基类可能不包含派生类的所有属性和方法**。如果程序试图访问这些不存在的属性或方法，将会抛出运行时异常。
* 向下转换可能违反多态性的原则。在多态中，派生类对象可以赋值给基类对象，但基类对象不能直接赋值给派生类对象，除非确实是派生类实例。否则，运行时将出现异常。

## 十一、Lambda表达式
Java的Lambda表达式是Java 8引入的一种新特性，旨在简化代码，使其更加简洁和易读。Lambda表达式本质上是一种匿名函数，可以作为参数传递给方法或存储在变量中。它们主要用于简化函数式编程，特别是在处理集合和流操作时。

Lambda表达式的基本语法如下：
```java
(parameters) -> expression;
// 或者
(parameters) -> { statements; }
```
`parameters`：参数列表，可以为空、一个参数或多个参数。\
`expression`：一个简单的表达式，如果有多条语句则使用大括号包裹。

#### 示例 1：无参数的Lambda表达式
```java
Runnable runnable = () -> System.out.println("Hello, Lambda!");
new Thread(runnable).start();
```

#### 示例 2：使用Lambda表达式进行集合排序
```java
List<String> list = Arrays.asList("b", "a", "d", "c");
Collections.sort(list, (s1, s2) -> s1.compareTo(s2));
System.out.println(list); // [a, b, c, d]
```

## 十二、模版类
Java 中的模板类（Template Class）通常指的是使用 **“泛型”（Generics）** 来创建的类。泛型允许类、接口和方法进行参数化，从而使代码更具通用性和复用性。

### 12.1 创建泛型类
泛型类在类名后面使用尖括号`<>`来指定类型参数。
```java
public class Box<T> {
    private T content;

    public void setContent(T content) {
        this.content = content;
    }

    public T getContent() {
        return content;
    }
}
```
在上述代码中，`Box`是一个泛型类，它有一个类型参数`T`。这个`T`可以在**实例化**时被指定为任何类型。

### 12.2 泛型类的属性
在Java中，泛型通过类型擦除来实现，**这意味着在运行时泛型类型信息会被移除**。因此，检查泛型类型的方法和属性通常在编译时进行。**如果你试图调用在泛型类中不存在的方法，编译器会报错。**
```java
public class GenericClass<T> {
    public void print(T item) {
        // 如果T类型没有toString方法，编译时会报错
        System.out.println(item.toString());
    }
}

public class Main {
    public static void main(String[] args) {
        GenericClass<String> generic = new GenericClass<>();
        generic.print("Hello"); // 这段代码是安全的

        // 下面的代码如果在T类型没有toString方法时，编译时会报错
        // GenericClass<SomeClassWithoutToString> generic2 = new GenericClass<>();
        // generic2.print(new SomeClassWithoutToString());
    }
}
```

### 12.3 接口约束
为了防止12.2的情况出现，可以使用接口约束来完成。
```java
interface HasName {
    String getName();
}

class NamePrinter<T extends HasName> {
    private T obj;

    public NamePrinter(T obj) {
        this.obj = obj;
    }

    public void printName() {
        System.out.println(obj.getName());
    }
}
```

## 十三、多文件编程 / 包管理
假设我们有一个简单的项目，其中包含以下三个文件： `Main.java`、`Person.java`和`Utils.java`：
* `Person.java`：
    ```java
    // Person.java
    package com.example.model;
  
    public class Person {
        private String name;
        private int age;
    
        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }
    
        public String getName() {
            return name;
        }
    
        public int getAge() {
            return age;
        }
    
        @Override
        public String toString() {
            return "Person{name='" + name + "', age=" + age + "}";
        }
    }
    ```
* `Utils.java`：
    ```java
    // Utils.java
    package com.example.util;
    import com.example.model.Person;
  
    public class Utils {
        public static void printPerson(Person person) {
            System.out.println(person);
        }
    }
    ```
* `Main.java`：
    ```java
    // Main.java
    package com.example;
    
    import com.example.model.Person;
    import com.example.util.Utils;
  
    public class Main {
        public static void main(String[] args) {
            Person person = new Person("Alice", 30);
            Utils.printPerson(person);
        }
    }
    ```
* 其中的文件结构为：
    ```
    project/
    └── com/
        └── example/
            ├── Main.java
            ├── model/
            │   └── Person.java
            └── util/
                └── Utils.java
    ```
* 编译方法为：
    ```
    javac com/example/**/*.java // 导航到根目录
    java com.example.Main // 运行主类
    ```
* 如果根目录下直接就是三个`.java`文件，那么就不需要`package`和`import`关键字。

## 十四、文件读写
### 14.1 读文件
使用`BufferedReader`和`FileReader`类可以方便地读文件。以下是一个示例代码，展示如何读取一个文本文件的内容：
```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class FileReadExample {
    public static void main(String[] args) {
        String filePath = "example.txt"; // 文件路径

        try (BufferedReader br = new BufferedReader(new FileReader(filePath))) {
            String line;
            while ((line = br.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

**字节流和字符流**：字节流是以字节为单位进行读写操作的IO流。它直接处理二进制数据，不关心字符编码；字符流是以字符为单位进行读写操作的IO流，它处理文本数据，会考虑字符编码。

**【`InputStreamReader`和`Reader`类】** `InputStreamReader`类用于将字节流（原始二进制文件，`InputStream`）转换为字符流（字符文件，`Reader`）。它是字节流和字符流之间的桥梁，通常用于读取二进制数据并将其转换为字符数据。`InputStreamReader`通常与`BufferedReader`结合使用，以提供高效的字符读取功能。`InputStreamReader`类的基类为`Reader`类。
* ```java
    public class InputStreamReader extends Reader { ... }
    public abstract class Reader implements Readable, Closeable { ... }
    ```

**【`FileReader`类】** `FileReader`类用于读取文件内容的一个类。它继承自`InputStreamReader`，主要用于读取字符文件。`FileReader`类是一个简单的字符流类，适合读取文本文件，但不适合读取二进制文件（如图片、视频等）。
* 构造函数`FileReader(String fileName)`： 创建一个新的`FileReader`，给定文件名。
* `int read()`：读取单个字符，返回读取的字符。如果已到达文件末尾，则返回`-1`。
* `int read(char[] cbuf, int offset, int length)`：读取字符到字符数组中，从指定的偏移量开始。
* `void close()`：关闭流并释放与之关联的任何系统资源。

**【`BufferedReader`类】** `BufferedReader`类用于读取字符输入流的一个类，**它提供了缓冲功能，以提高读取效率**。`BufferedReader`通常与其他读取流类（如`FileReader`）结合使用，以便更高效地读取文本文件。
* 构造函数`BufferedReader(Reader in)`：创建一个使用默认大小输入缓冲区的缓冲字符输入流。
* 构造函数`BufferedReader(Reader in, int sz)`：创建一个使用指定大小输入缓冲区的缓冲字符输入流。
* `String readLine()`：读取一行文本。返回一个包含行内容的字符串，不包含任何行终止字符。如果已到达流末尾，则返回`null`。
* `int read()`：读取单个字符。返回读取的字符，如果已到达流末尾，则返回-1。
* `int read(char[] cbuf, int offset, int length)`：读取字符到字符数组中，从指定的偏移量开始。
* `void close()`：关闭流并释放与之关联的任何系统资源。

### 14.2 写文件
使用`BufferedWriter`和`FileWriter`类可以方便地写文件。以下是一个示例代码，展示如何向一个文本文件写入内容：
```java
import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.IOException;

public class FileWriteExample {
    public static void main(String[] args) {
        String filePath = "example.txt"; // 文件路径
        String content = "Hello, World!\nWelcome to file writing in Java.";

        try (BufferedWriter bw = new BufferedWriter(new FileWriter(filePath))) {
            bw.write(content);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
### 14.3 读二进制文件
对于二进制文件，可以使用`FileInputStream`类来读取。以下是一个示例代码：
```java
import java.io.FileInputStream;
import java.io.IOException;

public class BinaryFileReadExample {
    public static void main(String[] args) {
        String filePath = "example.dat"; // 二进制文件路径

        try (FileInputStream fis = new FileInputStream(filePath)) {
            int byteContent;
            while ((byteContent = fis.read()) != -1) {
                System.out.print((char) byteContent);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
### 14.4 写二进制文件
对于二进制文件，可以使用`FileOutputStream`类来写入。以下是一个示例代码：
```java
import java.io.FileOutputStream;
import java.io.IOException;

public class BinaryFileWriteExample {
    public static void main(String[] args) {
        String filePath = "example.dat"; // 二进制文件路径
        byte[] content = { 72, 101, 108, 108, 111 }; // "Hello" in ASCII

        try (FileOutputStream fos = new FileOutputStream(filePath)) {
            fos.write(content);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## 十五、常用类
### 15.1 `String`
参考**第十章**。
```java
// 创建字符串
String str1 = "Hello";
String str2 = new String("World");

// 长度
int length = str1.length();  // 返回5

// 连接字符串
String result = str1.concat(str2);  // "HelloWorld"
// 或者使用 + 运算符
String result2 = str1 + " " + str2;  // "Hello World"

// 字符提取
char ch = str1.charAt(1);  // 返回 'e'
String sub = str1.substring(1, 4);  // 返回 "ell"

// 比较字符串
boolean equals = str1.equals("Hello");  // true
boolean ignoreCase = str1.equalsIgnoreCase("hello");  // true
int compareTo = str1.compareTo("Hellp");  // 返回负数，表示str1在字典序中排在"Hellp"之前

// 查找
int index = str1.indexOf('l');  // 返回2
int lastIndex = str1.lastIndexOf('l');  // 返回3
boolean contains = str1.contains("el");  // true

// 替换
String replaced = str1.replace('l', 'L');  // "HeLLo"
String replacedAll = str1.replaceAll("l", "L");  // "HeLLo"

// 大小写转换
String upper = str1.toUpperCase();  // "HELLO"
String lower = str1.toLowerCase();  // "hello"

// 去除首尾空白
String trimmed = "  Hello  ".trim();  // "Hello"

// 分割字符串
String[] parts = "Hello,World".split(",");  // ["Hello", "World"]

// 检查开始和结束
boolean startsWith = str1.startsWith("He");  // true
boolean endsWith = str1.endsWith("lo");  // true

// 格式化字符串
String formatted = String.format("Name: %s, Age: %d", "Alice", 30);

// 转换为字符数组
char[] chars = str1.toCharArray();

// 判空和空白
boolean isEmpty = "".isEmpty();  // true
boolean isBlank = "  ".isBlank();  // true (Java 11+)
```

### 15.2 `ArrayList`
`ArrayList`是Java集合框架中最常用的类之一，它实现了List接口，底层使用数组实现。`ArrayList`提供了**动态数组**的功能，能够根据需要自动调整大小。以下是`ArrayList`的主要特点和用法:
```java
// 创建ArrayList
ArrayList<String> list = new ArrayList<>();

// 添加元素
list.add("Apple");
list.add("Banana");
list.add(1, "Orange"); // 在指定位置添加元素

// 访问元素
String fruit = list.get(0); // 获取第一个元素

// 修改元素
list.set(1, "Grape"); // 将索引1的元素修改为"Grape"

// 删除元素
list.remove(0); // 删除第一个元素
list.remove("Banana"); // 删除指定元素

// 获取大小
int size = list.size(); 

// 检查是否包含某个元素
boolean contains = list.contains("Apple");

// 遍历列表
for (String item : list) { 
    System.out.println(item); 
}

// 清空列表
list.clear();
```
当在`ArrayList`末尾添加元素时（使用`add(E e)`方法），如果当前容量足够，直接添加即可，时间复杂度为`O(1)`。如果当前容量不足，`ArrayList`会创建一个更大的新数组（通常是当前大小的1.5倍），然后将所有元素复制到新数组中，这个过程的时间复杂度为`O(n)`。当在指定位置添加元素时（使用`add(int index, E element)`方法），需要将该位置及其后面的所有元素向后移动一位，这个过程也涉及到数组复制，时间复杂度为`O(n)`。

### 15.3 `Arrays`
`Arrays`类是`java.util`包中的一个实用工具类，提供了一系列用于**操作数组**的静态方法。
* `Arrays`类中的所有方法都是**静态的**，可以直接通过类名调用。
* 它不能被实例化，构造函数是私有的（不能使用`new`关键字创建`Math`类的对象）。
* 提供了对各种类型数组（包括基本类型和对象类型）的操作。
```java
// 排序
int[] numbers = {5, 2, 8, 1, 9};
Arrays.sort(numbers);  // 对数组进行排序

// 二分查找
int index = Arrays.binarySearch(numbers, 8);  // 在已排序的数组中查找元素

// 填充
int[] fillArray = new int[5];
Arrays.fill(fillArray, 10);  // 用指定值填充数组

// 复制
int[] copy = Arrays.copyOf(numbers, numbers.length);  // 复制数组

// 比较
boolean isEqual = Arrays.equals(numbers, copy);  // 比较两个数组是否相等

// 转换为List
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);  // 将数组转换为List

// 打印数组
System.out.println(Arrays.toString(numbers));  // 将数组转换为字符串

// 并行排序（Java 8+）
Arrays.parallelSort(numbers);  // 使用并行算法排序

// 多维数组操作
int[][] matrix = {{1, 2}, {3, 4}};
System.out.println(Arrays.deepToString(matrix));  // 打印多维数组
```
**【`List`是什么】** `List`是一个**接口**，而不是一个具体的类。它是`java.util`包中`Collection`接口的子接口，定义了一系列**操作有序集合**的方法。
`List`的常用的实现类包括：
* `ArrayList`：基于动态数组实现，随机访问效率高；
* `LinkedList`：基于双向链表实现，插入和删除效率高；
* `Vector`：类似ArrayList，但是线程安全（较少使用）；
* `Stack`：扩展自`Vector`，实现了“后进先出”（LIFO）的栈。
* `List`是一个有序集合（也称为序列）。
* ```java
  List<String> list1 = new ArrayList<>();
  List<Integer> list2 = new LinkedList<>();
  ```
  这里，`List`是接口类型，`ArrayList`和`LinkedList`是具体的实现类。

`List`接口要求：
* 允许重复元素。
* 可以通过索引访问元素。
* 支持动态增加和删除元素。

### 15.4 `Collections`
`Collections`类是Java集合框架中的一个工具类，位于`java.util`包中。它提供了一系列静态方法，用于**操作或返回集合**。
```java
// 排序
List<Integer> list = Arrays.asList(3, 1, 4, 1, 5, 9);
Collections.sort(list);  // 升序排序
Collections.reverse(list);  // 反转列表
Collections.shuffle(list);  // 随机打乱列表元素顺序

// 查找和替换
int max = Collections.max(list);  // 找最大值
int min = Collections.min(list);  // 找最小值
int frequency = Collections.frequency(list, 1);  // 元素出现的次数
Collections.replaceAll(list, 1, 2);  // 将所有的1替换为2

// 二分查找
int index = Collections.binarySearch(list, 5);  // 二分查找，返回索引（列表必须已排序）

// 填充和复制
Collections.fill(list, 0);  // 用0填充整个列表
List<Integer> dest = Arrays.asList(new Integer[list.size()]);
Collections.copy(dest, list);  // 复制列表

// 不可修改的集合
List<Integer> unmodifiableList = Collections.unmodifiableList(list);
Set<Integer> unmodifiableSet = Collections.unmodifiableSet(new HashSet<>(list));
Map<String, Integer> unmodifiableMap = Collections.unmodifiableMap(new HashMap<>());

// 同步集合（线程安全）
List<Integer> syncList = Collections.synchronizedList(new ArrayList<>());
Set<Integer> syncSet = Collections.synchronizedSet(new HashSet<>());
Map<String, Integer> syncMap = Collections.synchronizedMap(new HashMap<>());

// 单一元素或空集合
List<Integer> singletonList = Collections.singletonList(42);
Set<Integer> emptySet = Collections.emptySet();
Map<String, Integer> emptyMap = Collections.emptyMap();

// 对集合进行旋转
Collections.rotate(list, 2);  // 将列表中的元素向右旋转2个位置

// 交换元素
Collections.swap(list, 0, 1);  // 交换索引0和1的元素

// 查找子列表
List<Integer> subList = Arrays.asList(1, 4);
int index = Collections.indexOfSubList(list, subList);  // 查找子列表的起始索引
```

### 15.5 `Math`
`Math`类位于`java.lang`包中,提供了一系列用于数学计算的静态方法。以下是`Math`类的主要特点和常用方法:
* `Math`类中的所有方法都是静态的，可以直接通过类名调用。
* `Math`类不能被实例化，构造函数是私有的（不能使用`new`关键字创建`Math`类的对象）。
* 它包含两个常用的常量: `Math.PI` (π) 和 `Math.E` (自然对数的底)。
```java
Math.abs(-5);       // 绝对值: 5
Math.max(2, 3);     // 最大值: 3
Math.min(2, 3);     // 最小值: 2
Math.pow(2, 3);     // 幂运算: 8.0 (2的3次方)
Math.sqrt(9);       // 平方根: 3.0
Math.ceil(4.3);     // 向上取整: 5.0
Math.floor(4.8);    // 向下取整: 4.0
Math.round(4.5);    // 四舍五入: 5
Math.sin(Math.PI/2);  // 正弦
Math.cos(0);          // 余弦
Math.tan(Math.PI/4);  // 正切
Math.asin(1);         // 反正弦
Math.acos(0);         // 反余弦
Math.atan(1);         // 反正切
Math.log(Math.E);     // 自然对数
Math.log10(100);      // 以10为底的对数
Math.exp(1);          // e的1次方
Math.random();        // 返回0.0到1.0之间的随机数
```

## 十六、注解
在Java中，`@`是一个特殊符号，通常用于注解 (Annotations)。它不是一个运算符，而是一种**语法特性**，用于向代码添加元数据。这些元数据可以在编译时或运行时被访问，以影响程序的行为。

Java提供了一些内置的注解，例如：
* `@Override`: 该注解用于方法，表示该方法覆盖了父类的方法。
* `@Deprecated`: 该注解用于表示某个程序元素（类、方法等）已过时。
* `@SuppressWarnings`: 该注解用于告诉编译器忽略指定的警告。
* `@FunctionalInterface`: 该注解用于表示一个接口是一个函数式接口，也就是说，它只有一个抽象方法。

**【什么是函数式接口】** 函数式接口是一种特殊的接口，它只包含**一个**抽象方法。Java 8引入了这个概念，以支持新的函数式编程特性，如Lambda表达式和方法引用。函数式接口可以包含Java `Object`类中的公共方法，如`equals`，`hashCode`，`toString`等，这些方法不会计入抽象方法的数量。这是因为任何一个Java类都会默认继承`Object`类，所以它们并不会增加抽象方法的数量。
```java
@FunctionalInterface
public interface MyFunctionalInterface {
    void execute();
}

MyFunctionalInterface fi = () -> System.out.println("Executing...");
fi.execute();  // 输出 "Executing..."
```

除了这些内置注解，用户还可以自定义注解。例如：
```java
public @interface MyAnnotation {
  String value() default "";
}
```

然后你可以在代码中如下使用：
```java
@MyAnnotation("Hello, world!")
public class MyClass {
  // ...
}
```

## 十七、多线程编程和异步 (尚未完成)
### 17.1 多线程
多线程是一种允许在一个程序中同时运行多个线程的编程技术。线程是一个轻量级的进程，它可以与其他线程共享进程的资源（如内存）。多线程编程使得程序可以同时执行多个任务，提高程序的效率和响应速度。

在Java中，有两种主要方式创建线程：

* 继承`Thread`类
  ```java
  class MyThread extends Thread {
      public void run() {
          System.out.println("Thread is running.");
      }
      
      public static void main(String[] args) {
          MyThread thread = new MyThread();
          thread.start();  // 启动线程
      }
  }
  ```
* 实现`Runnable`接口
    ```java
    class MyRunnable implements Runnable {
        public void run() {
            System.out.println("Thread is running.");
        }
        
        public static void main(String[] args) {
            Thread thread = new Thread(new MyRunnable());
            thread.start();  // 启动线程
        }
    }
    ```

当多个线程访问共享资源时，可能会引发数据不一致的问题。线程同步是一种用于控制多个线程对共享资源访问的技术，通常使用`synchronized`关键字。

```java
class Counter {
    private int count = 0;
    
    public synchronized void increment() {
        count++;
    }
    
    public int getCount() {
        return count;
    }
}

public class SynchronizedExample {
    public static void main(String[] args) throws InterruptedException {
        Counter counter = new Counter();
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                counter.increment();
            }
        });
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                counter.increment();
            }
        });
        
        t1.start();
        t2.start();
        
        t1.join();
        t2.join();
        
        System.out.println("Count: " + counter.getCount());
    }
}
```


### 17.2 异步编程
异步编程是一种编程范式，**允许程序在等待某个操作完成时，不阻塞当前线程，而是继续执行其他操作**。Java中常用的异步编程方式包括`Future`和`CompletableFuture`。

* 使用`Future`
  ```java
  import java.util.concurrent.Callable;
  import java.util.concurrent.ExecutionException;
  import java.util.concurrent.ExecutorService;
  import java.util.concurrent.Executors;
  import java.util.concurrent.Future;
  
  public class FutureExample {
      public static void main(String[] args) {
          ExecutorService executor = Executors.newSingleThreadExecutor(); 
          // 使用Executors类的newSingleThreadExecutor方法创建一个单线程的执行服务（线程池）。
          
          Callable<Integer> task = () -> {
              Thread.sleep(2000);
              return 123;
          };
          // 创建一个Callable对象，Callable是一个带有返回值的任务。这个任务会睡眠2秒，然后返回整数123。
          
          Future<Integer> future = executor.submit(task);
          // 将Callable任务提交给执行服务，返回一个Future对象。Future用于表示异步计算的结果。
  
          while (!future.isDone()) { // 可以执行其他任务
              System.out.println("Task is not done yet...");
              try {
                  Thread.sleep(500);  // 模拟其他任务执行
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
          }
          
          try {
              Integer result = future.get();  // 阻塞直到任务完成
              System.out.println("Result: " + result);
          } catch (InterruptedException | ExecutionException e) {
              e.printStackTrace();
          } finally {
              executor.shutdown();
          }
      }
  }
  ```

* 使用`CompletableFuture`：`CompletableFuture`提供了更强大的异步编程支持，包括链式操作、组合多个异步任务等。
  ```java
  import java.util.concurrent.CompletableFuture;
  import java.util.concurrent.ExecutionException;
  
  public class CompletableFutureExample {
      public static void main(String[] args) {
          CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
              try {
                  Thread.sleep(2000);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
              return 123;
          });
          
          future.thenAccept(result -> System.out.println("Result: " + result));
          
          try {
              future.get();  // 主线程等待异步任务完成
          } catch (InterruptedException | ExecutionException e) {
              e.printStackTrace();
          }
      }
  }
  ```
