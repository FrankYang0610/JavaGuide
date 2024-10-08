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

**【Java程序的最终状态】** Java代码最终在JVM上运行。JVM本身是作为操作系统的一个进程运行的。JVM可以直接解释执行字节码。JIT也会在运行时将热点代码（频繁执行的代码）编译成本地机器码。最准确的说法是，**Java程序不会在编译时直接被编译为汇编或机器码。但在运行时，部分Java代码（尤其是热点代码）会被JIT编译器编译成汇编代码，然后转换为机器码以提高性能。** 

### 1.3 JVM的虚拟内存结构
JVM的虚拟内存结构是Java程序运行的基础。它定义了Java程序在运行时如何使用内存。**Java程序直接与JVM交互，使用JVM提供的内存结构。** 而操作系统仅为JVM分配物理内存、管理JVM进程、提供底层系统调用支持，不涉及具体Java程序。JVM内存结构包括：
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

例如在`public static native Thread currentThread()`方法中，当JVM启动时，它会加载包含本地方法实现的本地库，`currentThread()`的本地实现直接与底层操作系统交互，以获取有关当前线程的信息。以下是本地代码可能的简化伪代码表示：

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
JVM语言是指能够在JVM上运行的编程语言。**这些语言的源代码被编译成Java字节码**，然后可以在任何支持JVM的平台上运行。

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
        // ...
    }
    
    // 使用range for循环
    for (int number : numbers) {
        // ...
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
**【Java类型转换的注意事项】**
* `int`类型不能直接转换为`boolean`类型。`boolean`类型只有两个值：`true`和`false`，而`int`类型可以有任何整数值。如果你想根据`int`值的某些特性（例如，是否为零）转换为`boolean`，你需要自己编写条件表达式。
  * 在Java中，`if (anInteger) { ... }`是不合法的，因为Java的`if`语句需要一个`boolean`类型的表达式。**在Java中，我们不能像在某些其他语言（如C或JavaScript）中那样，将非布尔类型的值用作布尔表达式。**
* Java语言中没有提供类似C++中`const_cast`的操作。在C++中，`const_cast`可以用来改变表达式的常量性，这在Java中是非法的，因此没有这样的操作。
  * 在Java中，将方法参数声明为`final`意味着在方法体中不能改变该参数的引用，即不能重新为它赋值。但是，这并不阻止你修改这个对象的内部状态（即，它的字段或属性）。 例如，假设你有一个名为`Person`的类，该类有一个`name`字段，你可以在方法中改变`Person`对象的`name`，即使该对象的引用被声明为`final`：
    ```java
    class Person {
        String name;
      
        Person(String name) {
            this.name = name;
        }
    }
  
    public class Main {
        public static void main(String[] args) {
            Person person = new Person("Alice");
            System.out.println("Before method call, name = " + person.name);
            changeName(person);
            System.out.println("After method call, name = " + person.name);
        }
  
        static void changeName(final Person p) {
            p.name = "Bob"; // 这是合法的
            // p = new Person("Charlie"); // 这是非法的，因为 p 是 final 的
        }
    }
    ```
    如果想要避免这样的现象，可以使用**深拷贝**。**深拷贝是原始对象的完全复制，修改深拷贝不会影响原始对象。** 如果你将一个对象的深拷贝作为方法参数，那么即使方法修改了这个对象，也不会影响原始对象。在Java中，你可以使用各种方法来创建深拷贝，具体取决于你的需求和你的对象的复杂性。例如，你可以实现`Cloneable`接口并重写`clone`方法，或者使用序列化来创建深拷贝。
* Java中也没有类似C++中的类型重解释（`reinterpret_cast`）或C风格的强制类型转换。Java的类型系统是更强、更严格的，其类型转换必须是类型兼容的，并且在运行时还会进行类型检查。这是Java语言安全性和鲁棒性设计的一部分。

### 2.4 Java对变量的存储
#### 对于基本类型
```java
int number = 5;
```
在这个例子中，JVM执行以下步骤：
* 在栈内存中分配一块能够存储`int`类型数据的空间。
* 在这块内存空间中存储值`5`。
* 这块内存空间被标识符`number`标识，所以当我们在代码中写`number`，我们实际上是在引用这个内存位置的内容。

#### 对于对象类型（参考`new`关键字）
```java
String str = new String("Hello");
```
当使用`new`创建一个对象时，JVM执行以下步骤：
* `new String("Hello")`在堆内存中创建了一个新的`String`对象。
* 这个新对象的内存地址被返回并赋值给`str`。JVM在栈内存中为`str`分配内存空间，并把返回的地址存储在这个空间中。
* 因此，`str`成为这个新`String`对象的**引用**。`str`实际上是是字符串地址的标识。不过在使用`str`的时候，JVM会自动访问地址。

举一个更好的例子，在Java中，数组的实现涉及以下几点：
* 对象头：Java数组在堆上分配，包含对象头信息，其中包括类型标识和数组长度等元数据。
* 数组数据：紧跟对象头之后，是数组的实际数据。
* 引用：在栈上，变量保存的是对数组对象的引用（指针），而不是数组本身。

#### 联系C++
* 在C++中，把变量名理解为「标识」或「标签」是正确的。**一个引用就是一个位置的别名**。这种理解方式符合编程语言设计的预期。
* 当然，在汇编层面，有时，一个引用可能占用一块新的内存空间，这和指针类似：
  ```c++
  int main() {
      int a = 20;
      int& b = a;
      b++;
  }
  ```
  对应的汇编代码是：
  ```arm
  _main:
      sub	sp, sp, #16
      
      % 提前确定了a的地址为sp+12，存在x8寄存器中。
      add	x8, sp, #12
      
      % int a = 20;
      mov	w9, #20
      str	w9, [sp, #12]
      
      % int b = &a;
      str	x8, [sp]    % sp指向为b开辟的空间，其中存储a的地址。
      
      % b++;
      ldr	x9, [sp]
      ldr	w8, [x9]
      add	w8, w8, #1
      str	w8, [x9]    % 这里实际上是a++，符合实际。
      
      mov	w0, #0
      add	sp, sp, #16
      ret
  ```
* Java并不区分引用和指针，一般情况下我们只说引用。在Java的语境下，可能存在一点术语滥用。C++的引用和Java的引用**完全不同**，Java的引用实际上和C++的指针更相似，因为二者都「指向」一个位置，而非一个位置的别名。

### 2.5 函数参数传递
在Java中，基本类型（如`int`, `double`, `char`等）和引用类型的处理方式是**不同**的。

在Java中，所有的非基本类型都被称为引用类型。这包括类（Class）、接口（Interface）、数组（Array）和枚举（Enum）。**引用类型的特点是，它们的值实际上是对存储在堆（Heap）中的对象或数组的引用，而不是实际的对象或数组本身。** 当你创建一个引用类型的对象（比如一个类的实例或一个数组）时，Java会在堆内存中为这个对象分配空间，**并返回一个指向这个内存空间的引用**（类似C的指针）。这个引用，而不是对象本身，就是你在代码中操作的值。

**在Java中，对于基本类型和引用类型，Java都是按值传递的，即传递一个值的拷贝。对于基本类型，这个值就是基本类型的实际值，对于引用类型，这个值就是引用本身（也就是指向对象在内存中的指针）。**

* 当你将一个基本类型量传递给函数时，如果函数试图修改它的值，那么原始对象中的那个值是**不会改变**的。
* 当你将一个**包含引用类型**的对象传递给函数，**函数对这个引用类型的修改可能影响到原始对象**。如果你改变的是引用本身（即指向另一个不同的对象），那么这种改变并不会影响到原始对象。但是，如果你通过这个引用修改了所指向的对象的状态（比如改变一个对象的字段），那么这种改变会影响到原始对象。

**【联系C++】** 在C++中：
* 函数参数中基本数据类型都是按值传递的，这和Java相同。
* 如果函数参数中有**引用类型**，从C++层面来讲（不考虑汇编），这是原始对象的新标识，函数可以直接操控原始对象的内容。不过在汇编层面，程序可能直接操作原始对象，也可能会在栈上开辟一块新空间存储引用内容的地址。
* 如果函数参数中有**指针类型**，程序则在栈上开辟一块新空间存储指向的内容的地址。如果此指针参数也是一个指向对象的指针（类似`func(ptr_to_object)`）而非一个右值（如`func(&var)`），那这和Java中引用类型的参数传递方式最相似（指针的拷贝）。
* Java和C++在内存管理上有很大差异。因此Java很难完全类比C++。

### 2.6 指针和地址？
在Java中**没有**像C或C++那样的显式指针和内存地址操作。Java是设计为内存安全的语言，旨在防止诸如内存泄漏和指针操作错误等常见问题。虽然你不能直接访问对象的内存地址，但Java提供了`System.identityHashCode()`方法，可以获得对象的HashCode，该HashCode在对象的生命周期内是唯一的，尽管它们并不直接对应实际内存地址。

### 2.7 对象整体的拷贝类型
**Java的函数参数（包括C++的函数传递）传递不是以下两种类型的任何一种，其只是指针的拷贝（本质上还是按值传递）。** 以下内容针对的是**赋值运算符和`clone`操作**。

* 浅拷贝创建一个新对象，然后将原始对象的属性值复制到新对象。如果属性值是基本类型，如整数、字符串或布尔值，那么就直接复制这些值。但是，如果属性是引用类型（如数组、对象、函数等），那么复制的是这些引用的地址，而不是引用的实际内容。
  ```java
  class Person implements Cloneable {
      String name;
  
      Person(String name) {
          this.name = name;
      }
  
      @Override
      protected Object clone() throws CloneNotSupportedException {
          return super.clone();
      }
  }
  
  // 使用浅拷贝
  Person person1 = new Person("Alice");
  Person person2 = (Person) person1.clone();
  
  person2.name = "Bob"; // 更改person2的name字段也会影响person1的name字段，因为它们指向的是同一个String对象
  ```
  `super.clone()`调用的是`Object`类中定义的`protected native Object clone() throws CloneNotSupportedException`方法，该方法的作用是创建并返回对象的一个副本。这个副本是通过复制原始对象在内存中的二进制流来创建的，因此得到的是一个全新的对象，其内容与原始对象完全相同。

* 深拷贝创建一个新对象，并递归地复制原始对象的所有字段。**对于引用字段，深拷贝会创建一个新的对象，并复制引用的目标对象的所有字段，而不仅仅是复制引用。** 这意味着新对象和原始对象**不会共享任何**对象或值。
  ```java
  class Person implements Cloneable {
      String name;
  
      Person(String name) {
          this.name = name;
      }
  
      @Override
      protected Object clone() {
          return new Person(new String(name)); // 手动创建一个新的String对象，实现深拷贝
      }
  }
  
  // 使用深拷贝
  Person person1 = new Person("Alice");
  Person person2 = (Person) person1.clone();
  
  person2.name = "Bob"; 
  // 更改person2的name字段不会影响person1的name字段，因为它们指向的是两个不同的String对象
  ```


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

**【如何处理输入的数字】** `Scanner.nextLine()`方法返回的是一个`String`类型的值，不会直接转换数字。然而，你可以使用`Integer.parseInt()`或`Double.parseDouble()`之类的方法将这个字符串转换为一个整数或浮点数。
```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        System.out.println("Enter a number: ");
        String line = scanner.nextLine();

        // 转换为整数
        int num = Integer.parseInt(line);
        System.out.println("The number as an integer is " + num);

        // 转换为浮点数
        double d = Double.parseDouble(line);
        System.out.println("The number as a double is " + d);
    }
}
```

**【什么是`Integer`和`Double`】** `Integer`是一个封装了`int`原始数据类型的类。该类提供了许多用于操作`int`的方法，例如比较、转换为其他类型、进行位操作等等。同样，`Integer`类也允许你将`int`值视为一个对象。这在很多情况下是非常有用的，例如，只有对象才能被添加到Java的集合类中（如`ArrayList`和`HashSet`）。

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
### 8.1 异常处理模型
```java
try {
    int result = 10 / 0;
} catch (ArithmeticException e) {
    System.out.println("Cannot divide by zero!");
} finally {
    System.out.println("This block is always executed.");
}
```

### 8.2 内置异常列表
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

### 8.3 用户自定义的异常
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

### 8.4 `throw`和`throws`
在Java中，`throw`和`throws`是两个关键字，它们都与异常处理有关，但是用法和目的不同。
* `throw`是一个语句，用于显式地抛出一个异常。这个异常可以是Java内置的异常类型，也可以是你自定义的异常类型。**一旦抛出异常，程序的正常执行流程将被中断，并立即跳转到最近的适当的异常处理器（`catch`块）**。
* `throws`是一个关键字，**用于声明一个方法可能会抛出哪些类型的异常**。这些异常类型是列在方法的声明部分的，紧跟在方法签名之后。比如：
  ```java
  public void readFile(String fileName) throws IOException {
      // Method implementation...
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
    static Person DEFAULT_PERSON = new Person("DEFAULT_PERSON_NAME", 0); // 在类运行时即创建，不需要任何实例
    private String name = "DEFAULT_NAME"; // 在对象创建时会被初始化为"DEFAULT_NAME"
    private int age;

    { age = 18; } // 构造器在此之后被调用

    // 构造器
    public Person(String name, int age) { // 在其他初始化完成后，构造器被调用
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
### 10.1 访问控制【参见**13.2**】
* `public`：任何地方都可以访问。
* `private`：只有在同一个类内部可以访问。
* `protected`：在同一个包内或通过继承可以访问。
（默认，即不使用修饰符）：在同一个包内可以访问。

通常，一个公共类（`public class`）应该被保存在一个与类名相同的`.java`文件中。 例如，`class MyClass`应该保存在`MyClass.java`文件中。一个`.java`文件可以包含多个类定义。但只能有一个`public`类，且该`public`类的名称必须与文件名相同。Java文件的位置通常反映了类的包结构。例如，`com.example.MyClass`类应该位于`com/example/MyClass.java`文件中。

在Java中，顶级类（即直接在`.java`文件中定义的类）只能是`public`或者`package-private`（不加任何访问控制符）。不能将顶级类声明为`private`或`protected`。`package-private`类只在同一个包内可见。

```java
// 这是允许的：package-private 顶级类
class PackagePrivateClass {
    // 类的内容
}

// 这是允许的：public 顶级类
public class PublicClass {
    // 私有嵌套类
    private class PrivateNestedClass {
        // 类的内容
    }
}

// 这是不允许的：private 顶级类
// private class PrivateTopLevelClass {  // 编译错误
//     // 类的内容
// }
```

### 10.2 主入口
在Java程序中，主入口是`main`方法。`main`方法是Java应用程序的起点，JVM会在启动应用程序时首先调用这个方法。`main`方法必须是`public`的、`static`的，并且返回类型是`void`，其参数是一个`String`类型的数组。

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
通过`extends`关键字来实现。继承使得一个类可以继承另一个类的属性和方法。派生类无法访问基类的私有成员，只能访问其`public`和`protected`成员。
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

在Java中，你不能直接继才两个基类。**Java不支持多重继承**，这是为了防止各种问题和复杂性，例如二义性，由于两个父类可能有相同的方法或属性。

【`super`关键字】`super`关键字用于：
* 调用父类的构造函数：你可以使用`super()`在子类的构造函数中调用父类的构造函数。**这必须是在构造函数的第一行。**
* 调用父类的方法或访问属性：你可以使用`super.method()`或`super.property`来访问父类的方法或属性。
* 如果你的类实现了多个接口，并且这些接口有默认方法，你可以使用`InterfaceName.super.method()`的形式来调用指定接口的默认方法。这种情况下，super关键字是用来解决默认方法之间的冲突的。
  ```java
  interface InterfaceA {
      default void doSomething() {
          System.out.println("InterfaceA's implementation");
      }
  }
  
  interface InterfaceB {
      default void doSomething() {
          System.out.println("InterfaceB's implementation");
      }
  }
  
  class MyClass implements InterfaceA, InterfaceB {
      public void doSomething() {
          InterfaceA.super.doSomething(); // Calls InterfaceA's implementation
          InterfaceB.super.doSomething(); // Calls InterfaceB's implementation
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

### 10.8 `final`类、`sealed`类和`permits`关键字
`final`类都是不能被继承的类。其目的是确保类的实现不被修改或扩展：
```java
public final class ImmutableClass {
    private final int value;

    public ImmutableClass(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }
}
```

`sealed`类限制哪些类可以继承它。 其目的是提供更好的控制和安全性。
其必须使用`permits`关键字指定允许的子类。子类选项可以为：
* `final`: 不能再被继承。
* `sealed`: 继续限制其子类。
* `non-sealed`: 可以被自由继承。

```java
public sealed class Shape permits Circle, Square {
    // 类的实现
}

public final class Circle extends Shape {
    // Circle的实现
}

public final class Square extends Shape {
    // Square的实现
}
```

### 10.9 友元的实现
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

### 10.10 类的类型转换
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
### 13.1 多文件编程
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

### 13.2 包管理
包（package）是Java中用于组织和管理相关类和接口的机制。它们在Java程序设计中扮演着重要角色。**包是一种命名空间机制，用于将相关的类、接口和子包组织在一起。** 它们类似于文件系统中的文件夹，用于组织和分类Java代码。一个包通常包含多个`.java`源文件。每个源文件可以定义一个或多个类或接口。同一个包中的所有源文件都应该在文件开头有**相同的**包声明。所有属于同一包的文件通常位于同一目录下。

包名通常使用小写字母。常用的命名方式是使用公司的域名倒序，如`com.example.project`。在Java源文件的开头使用`package`关键字声明包。 例如：`package com.example.myproject;`。

使用`import`语句可以导入其他包中的类。例如：`import java.util.ArrayList;`。

包的结构通常对应于文件系统的目录结构。例如，`com.example.myproject`包对应于`com/example/myproject`目录。不过这不是强制的，理论上，包的结构可以不完全对应系统目录，但在实践中，这种做法通常**不被推荐**，而且可能会导致一些问题，尤其是当IDE默认包的结构对应系统目录的时候。

包私有（默认）访问级别：同一包内的类可以相互访问。`public`成员可以被任何类访问。`protected`成员可以被同包类和任何派生类访问。`private`成员只能在声明它们的类内部访问。 Java中的访问修饰符从最严格到最宽松的顺序是: `private -> 默认(包私有) -> protected -> public`。

```java
package com.example;
public class Base {
    private int privateVar;
    int defaultVar;
    protected int protectedVar;
    public int publicVar;
}


package com.example;
class SamePackageClass {
    void method() {
        Base b = new Base();
        // b.privateVar; // 不可访问
        b.defaultVar;    // 可以访问（同包）
        b.protectedVar;  // 可以访问（同包）
        b.publicVar;     // 可以访问
    }
}

package com.another;
class DifferentPackageSubclass extends Base { 
    void method() {
        // this.privateVar; // 不可访问
        // this.defaultVar; // 不可访问（不同包）
        this.protectedVar;  // 可以访问（派生类）
        this.publicVar;     // 可以访问
    }
}
```

包导入**不具有传递性**：如果包C导入包B，它不会自动导入包A，即使包B导入了包A。**导入（Import）** 和 **依赖（Dependency）** 在Java中是两个不同的概念：
* 导入（Import）：用于简化类名的引用，使我们能够在源文件中使用类的简单名称而无需使用全限定名。例如，使用`import java.util.List;`后，代码中可以直接使用`List`而不是`java.util.List`。
* 依赖（Dependency）：指的是一个类或包在编译或运行时所需的其他类或包。即使不使用`import`语句，Java编译器仍然可以根据全限定名找到所需的类。
* JVM实现了必要的模块化，在使用、编译和运行一个包的时候不必要考虑它的依赖。这涉及到了以下技术：
  * 封装
  * 每个模块在`module-info.java`文件中声明其直接依赖项
  * 运行时加载：Java 使用 **类加载器（ClassLoader）** 机制，在运行时动态加载所需的类。这类似于动态链接库（如 `.dll`、`.so` 文件）的动态加载，允许应用程序在需要时加载和卸载类。

包的目的主要有：
* 避免命名冲突：不同包中的类可以有相同的名称。
* 控制访问：包提供了一个访问控制的边界。
* 更好的组织：让代码结构更清晰，便于管理和维护。

编译时需要指定包的路径。运行时需要正确设置类路径（classpath），见**13.3**节。

### 13.3 classpath和类加载器
在Java中，**类加载器（Class Loader）** 负责在运行时动态加载类。默认的类加载器通过搜索在`classpath`中定义的一组路径来查找和加载类。

`classpath`是一组路径，用于指定JVM和Java工具（如javac、java等）在查找类和资源文件时所要搜索的目录或`.jar`文件。比如：
```
c:\jdk\src;d:\mylib\src;
```

* 可以将`classpath`设置为系统的环境变量，这样所有Java应用都可以共享这个设置。
  ```
  c:\> set CLASSPATH=c:\jdk\lib;d:\mylib1;
  ```
* 在执行Java命令时，可以通过`-classpath`或`-cp`参数指定`classpath`。这种方式的优先级高于环境变量中的设置。
  ```
  c:\> java -classpath c:\jdk\lib;d:\mylib2 <其他参数>
  ```
  * 使用命令行参数设置的`classpath`会覆盖环境变量中设置的`classpath`。
  * 如果未显式设置`classpath`，默认的`classpath`仅包含当前工作目录（即`.`）。

`java.lang`等标准Java包确实能被Java程序默认识别和使用，**但它们并不是通过用户设定的`classpath`来加载的**。Java的类加载器采用**父母委托模型（Parent Delegation Model）**，该模型规定了类加载器之间的委托关系，确保在类加载过程中，类加载请求会按照特定的顺序逐级传递，直到被最顶层的类加载器（通常是启动类加载器）处理。类加载器主要分为以下几种类型：
* 启动类加载器（Bootstrap Class Loader）
  * 用于加载Java核心库，如`java.lang.*`、`java.util.*`等。通常来自于JVM内部的核心库（如`rt.jar`，在Java 9及以后版本中则通过模块系统提供）。启动类加载器不使用用户设置的`classpath`，而是使用JVM内部预定义的路径。
* 扩展类加载器（Extension Class Loader）
  * 用于加载Java扩展库，通常位于`jre/lib/ext`目录下的`.jar`文件或由系统属性`java.ext.dirs`指定的目录。它同样不依赖用户直接设置的`classpath`，而是依赖于JVM的扩展库路径。
* 应用程序类加载器（Application/System Class Loader）
  * 用于加载应用程序的类和第三方库。**它依赖于用户通过环境变量`classpath`或命令行参数`-classpath/-cp`设置的路径。**
* 自定义类加载器（Custom Class Loader）
  * 用于允许开发者根据需要自定义类的加载方式，通常用于特定的需求，如动态加载、热部署等。可以通过编程方式指定加载路径或策略。

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
`ArrayList`是Java集合框架中最常用的类之一，它实现了`List`接口，底层使用数组实现。`ArrayList`提供了**动态数组**的功能，能够根据需要自动调整大小。以下是`ArrayList`的主要特点和用法:
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

**在Java中，`ArrayList`只能存储对象，不能直接存储基本数据类型，如`int`、`char`、`boolean`等。** 然而，Java提供了**封装类**（如`Integer`、`Character`、`Boolean`等），可以将基本类型封装为对象，这样就可以将它们存储在`ArrayList`中了。

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

## 十七、多线程和异步编程
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

当多个线程访问共享资源时，可能会引发数据不一致的问题。线程同步是一种用于控制多个线程对共享资源访问的技术，通常使用`synchronized`关键字。`synchronized`可以在多线程环境下保证共享资源的访问是互斥的，即在同一时刻只能有一个线程能访问该资源，防止出现数据不一致的问题。

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

### 17.2 `volatile` && 多线程的进一步理解
`volatile`关键字**表明一个变量可能会被多个线程同时访问并修改**。`volatile`能够保证变量的可见性和有序性。

**【什么是可见性】** 可见性指的是当一个线程修改了`volatile`变量的值，其他线程能立即看到这个变化。通常，线程会在自己的工作内存中缓存变量的副本，`volatile`可以保证每次读写都直接从主内存获取或更新值，避免读取到过时的数据。

**【什么是有序性】** 为了优化性能，Java编译器可能会重新排列指令顺序。`volatile`确保对其的读写操作不会被重排序，保持代码执行的顺序一致，从而避免因指令重排带来的问题。

```java
public class SharedData {
    public volatile int counter = 0;
}
```

在这个例子中，`counter`是一个`volatile`变量，意味着一个线程对`counter`的修改对所有其他线程都是可见的。然而，`volatile`并不保证操作的原子性。例如，`counter++`实际上包含三个步骤：读取、增加和写回。如果多个线程同时执行，可能会发生竞态条件，导致计数不准确。此时需要使用锁或其他同步机制来保证原子性。

【联系**17.1**的例子】
```java
// 17.1
class Counter {
    private int count = 0;
    
    public synchronized void increment() {
        count++;
    }
    
    public int getCount() {
        return count;
    }
}

```

在17.1的例子中，`count`变量由`synchronized`的`increment()`方法保护，因此不需要声明为`volatile`。`synchronized`确保了以下几点：
* 互斥访问：同一时间只有一个线程可以执行被`synchronized`修饰的代码块。
* 可见性：线程在进入`synchronized`块时，会从主内存读取最新的变量值；在退出时，会将修改写回主内存。这符合Java内存模型中的Happens-Before原则。

* 然而，如果`getCount()`方法可能被多个线程并发访问，并且你需要它总是返回`count`的最新值，那么你应该将`getCount()`方法也声明为`synchronized`，或者将`count`声明为`volatile`。否则，`getCount()`方法可能返回`count`的过时值。
  * 即使线程A在`increment()`方法中修改了`count`的值并释放了锁（这将把`count`的最新值刷新到主内存），**另一个线程B在执行`getCount()`方法时可能仍然从其工作内存中读取`count`的过时值，而不是从主内存中读取最新的值**。这是因为`getCount()`方法并没有被声明为`synchronized`，所以它不会在执行时**强制**从主内存中读取`count`的值。

**【哪些操作是原子的】**
以下操作是原子的：
* 基本数据类型的读取和赋值：除`long`和`double`外，其他基本类型的读写是原子的。
* 引用类型的赋值：引用的读写也是原子的。
* 原子类：如`AtomicInteger`、`AtomicLong`和`AtomicBoolean`提供了原子的操作方法，例如`AtomicInteger`的`incrementAndGet`方法。

**【联系`synchronized`关键字】** `volatile`和`synchronized`都是Java中用于处理多线程环境下的可见性和同步问题的关键字，但它们的工作方式和使用场景有所不同。
* 原子性
  * `synchronized`保证代码块内操作的原子性，确保同一时间只有一个线程执行。
  * `volatile`只保证单个读写操作的原子性，无法处理复合操作（如 counter++）。
* 互斥锁
  * `synchronized`使用互斥锁，带来一定的锁开销。
  * `volatile`不使用锁，因此不会有锁相关的性能开销。
* 可见性
  * `synchronized`通过获取和释放锁来保证可见性。
  * `volatile`通过每次读写直接操作主内存，保证最新值的可见性。
* 如果需要执行的操作是一个复合操作，那么你应该使用`synchronized`来保证这个操作的原子性。如果只是读取和写入单个变量，而且不关心操作的原子性，那么可以使用`volatile`。

### 17.3 异步编程
异步编程是一种编程范式，**允许程序在等待某个操作完成时，不阻塞当前线程，而是继续执行其他操作**。异步操作在未来的某个时间点完成，而不是立即完成。当异步操作完成时，通过某种机制（如回调函数、`Future`对象、事件等）通知调用方或处理结果。

**【异步和多线程的异同】** 
* 异步是一种编程模型，允许程序在**等待**某个操作完成的同时继续执行其他操作。多线程是一种并发执行的方法，允许多个线程（执行路径）同时运行。
* 异步可以实现并发（同一**时间间隔**内执行多个任务），但不一定是并行（同一**时刻**同时执行多个任务）的。多线程可以实现真正的并行执行（在多核系统上）。
  * 异步通常使用 **事件循环（event loop）** 机制。在单线程环境中，事件循环允许程序在等待某个操作（如I/O）时切换到其他任务，而不是一直等待。在单线程异步模型中（如Node.js的默认模型），虽然可以同时处理多个任务，**但在任何给定的时刻，只有一个任务在实际执行。这就是并发但不并行。**
* 异步主要目的是提高程序的响应性和效率，特别是在处理I/O操作时。多线程主要目的是实现真正的并行处理，提高程序的整体吞吐量。
* 异步特别适合I/O密集型任务，如网络请求、文件操作等。多线程特别适合CPU密集型任务，可以充分利用多核处理器。

Java中常用的异步编程方式包括`Future`和`CompletableFuture`。
* 使用`CompletableFuture`：`CompletableFuture`提供了更直观和更强大的异步编程支持，包括链式操作、组合多个异步任务等。
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
          // 这一行是异步的。它"注册"了一个回调函数，当future完成时会被调用。这个操作本身不会阻塞主线程。
          // 下面的代码会继续执行，在异步结束后这个“注册”的函数会被自动执行。
  
          // 其他操作...
          
          try {
              future.get();  // 等待异步任务完成，确保异步任务完成再继续
          } catch (InterruptedException | ExecutionException e) {
              e.printStackTrace();
          }
      }
  }
  ```

* 使用`Future`，`Future`是`CompletableFuture`的基类。
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
          // Callable对象是一个带有返回值的任务。
          
          Future<Integer> future = executor.submit(task);
          // 将Callable任务提交给执行服务，返回一个Future对象。Future用于表示异步计算的结果。
          // 这就是“注册”的函数。
  
          // 可以执行其他任务
          
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

## 十八、其他Java关键字
### 18.1 `assert`
`assert`用于进行断言。断言是一种编程手段，使得程序员可以在代码中插入一些检查点，以测试假设的条件是否为真。如果断言的条件为真，程序会正常运行。但如果断言的条件为假，程序会抛出一个`AssertionError`。Java 中的 assert 语句有两种形式：
* `assert Expression1;`
* `assert Expression1 : Expression2;`
```java
public void deposit(double value) {
    assert value > 0 : "Deposit value must be greater than 0";
    balance += value;
}
```

在这个例子中，我们假设存款值必须大于0。如果这个假设不成立，那么会抛出一个`AssertionError`，并带有错误消息 `Deposit value must be greater than 0`。

### 18.2 `strictfp`
`strictfp`是Java中的一个关键字，用于精确控制浮点计算的精度。当你在一个类或者方法上使用`strictfp`关键字，那么**在该类或方法中的所有浮点计算**都将严格遵循IEEE 754规范。

IEEE 754规范定义了浮点数的表示方法、舍入方式、特殊值（如`NaN`和无穷大）、异常（如除以零和溢出）等内容。在严格模式下，所有的浮点运算都会精确地遵循这个规范，以提供可预见的、跨平台的结果。

### 18.3 `transient`
`transient`是一个关键字，用于表明某个对象的属性不应该被序列化。

序列化是将对象的状态（即其属性）转换为字节流的过程，以便将其保存到磁盘上，或者通过网络将其发送到另一个运行Java的机器上。如果一个对象的某个属性被标记为`transient`，那么这个属性就不会参与到序列化过程中，也就不会被保存到磁盘或通过网络传输。

```java
public class User implements Serializable {
    private String name;
    private transient String password;
    
    // rest of the class...
}
```

在这个例子中，`User`类有两个属性：`name`和`password`。当我们尝试序列化一个`User`对象时，`name`属性会被序列化，但是`password`属性由于被标记为`transient`，所以不会被序列化。

被`transient`关键字修饰的字段在对象序列化时**不会被保存下来**。这意味着如果你将一个对象序列化（比如写入到文件，或者通过网络发送），然后再反序列化（从文件读取，或者从网络接收），那么这个对象的`transient`字段将不会被恢复，而是采用其类型的默认值。对于对象类型，这个默认值是`null`；对于基本类型，例如`int`，`double`，这个默认值是`0`，`boolean`类型的默认值是`false`。

`transient`关键字只影响序列化过程。在对象的生命周期内，即使属性被标记为`transient`，它仍然可以被正常地读取和写入。`transient`关键字只是告诉JVM在序列化或反序列化过程中忽略这个字段。
