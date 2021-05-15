> Java 的基本原理就是“形式错误的代码不会运行”。
>
> 与 C++类似，<u>*捕获错误最理想的是在编译期间*</u>。然而，并非所有错误都能在编译期间侦测到。有些问题必须在运行期间解决，让错误的缔结者通过一些手续向接收者传递一些适当的信息，使其知道该如何正确地处理遇到的问题。
>
> ==解决的方法是在错误控制中排除所有偶然性，强制格式的正确==。==“违例”(Exception)==这个词表达的是一种“例外”情况，亦即正常情况之外的一种“异常”。在问题发生的时候，我们可能不知具体该如何解决，但肯定知道已不能不顾一切地继续下去。此时，必须坚决地停下来，并由某人、某地指出发生了什么事情，以及该采取何种对策。但为了真正解决问题，当地可能并没有足够多的信息。因此，我们需要将其移交给更高级的负责人，令其作出正确的决定(类似一个命令链)。违例机制的另一项好处就是能够简化错误控制代码。我们再也不用检查一个特定的错误，然后在程序的多处地方对其进行控制。此外，也不需要在方法调用的时候检查错误(因为保证有人能捕获这里的错误)。我们只需要在一个地方处理问题:“违例控制模块”或者“违例控制器”。这样可有效减少代码量，并将那些用于描述具体操作的代码与专门纠正错误的代码分隔开。一般情况下，用于读取、写入以及调试的代码会变得更富有条理。
>
> 由于==违例控制是由 Java 编译器强行实施的==，所以毋需深入学习违例控制，便可正确使用本书编写的大量例子。本章向大家介绍了用于正确控制违例所需的代码，以及在某个方法遇到麻烦的时候，该如何生成自己的违例。

### 1. 基本违例

> ==违例条件==：表示在出现什么问题的时候应中止方法或作用域的继续。
>
> 为了将违例条件与普通问题区分开，违例条件是非常重要的一个因素。在普通问题的情况下，我们在当地已拥有足够的信息，可在某种程度上解决碰到的问题。而<u>*在违例条件的情况下，却无法继续下去，因为当地没有提供解决问题所需的足够多的信息。此时，我们能做的唯一事情就是跳出当地环境，将那个问题委托给一个更高级的负责人*</u>。这便是出现违例时出现的情况。 一个简单的例子是“除法”。如可能被零除，就有必要进行检查，确保程序不会冒进，并在那种情况下执行 除法。但具体通过什么知道分母是零呢?在那个特定的方法里，在我们试图解决的那个问题的环境中，我们或许知道该如何对待一个零分母。但假如它是一个没有预料到的值，就不能对其进行处理，所以必须产生一 个违例，而非不顾一切地继续执行下去。

> 产生一个违例时，会发生几件事情。
>
> - 首先，按照与创建Java 对象一样的方法创建违例对象：在内存“堆” 里，使用 new 来创建。
> - 随后，<u>*停止当前执行路径*</u>(记住不可沿这条路径继续下去)，然后从当前的环境中*<u>释放出违例对象的句柄</u>*。
> - 此时，违例控制机制会接管一切，并开始查找一个恰当的地方，用于继续程序的执行。这个恰当的地方便是“==违例控制器==”，它的职责是<u>*从问题中恢复，使程序要么尝试另一条执行路径，要么简单地继续*</u>。

> 作为产生违例的一个简单示例。有些时候，程序可能传递一个尚未初始化的句柄。所以在用那个对象句柄调用一个方法之前，最好进行一番检查。可将与错误有关的信息发送到一 个更大的场景中，方法是创建一个特殊的对象，用它代表我们的信息，并将其“掷”(Throw)出我们当前的场景之外。这就叫作“*<u>产生一个违例</u>*”或者“*<u>掷出一个违例</u>*”。下面是它的大概形式:
>
> ```java
> if(t == null) {
> 	throw new NullPointerException(); 
> }
> ```
>
> 这样便“掷”出了一个违例。在当前场景中，它使我们能放弃进一步解决该问题的企图。该问题会被转移到其他更恰当的地方解决。准确地说，那个地方不久就会显露出来。

#### 1.1 违例自变量

> 和 Java 的其他任何对象一样，需要用 new 在内存堆里创建违例，并需调用一个构建器。在所有标准违例中，存在着两个构建器：第一个是==默认构建器==，第二个则==需使用一个字串自变量，使我们能在违例里置入相关信息==：
>
> ```java
> if(t == null) {
> 	throw new NullPointerException("t = null");
> }
> ```
>
> 在这儿，关键字 ==throw== 会像变戏法一样做出一系列不可思议的事情。
>
> - 它首先执行 new 表达式，*<u>创建一个不在程序常规执行范围之内的对象</u>*。
> - 随后，对象实际会从方法中返回——尽管对象的类型通常并不是方法设计为返回的类型。
> - 通过“掷”出一个违例，亦可==从原来的作用域中退出==。但是<u>*会先返回一个值，再退出方法或作用域*</u>。但是，与普通方法返回的相似性到此便全部结束了，因为我们返回的地方与从普通方法调用中返回的地方是迥然有异的(我们结束于一个恰当的违例控制器，它距离违例“掷”出的地方可能相当遥远——在调用堆栈中要低上许多级)。 此外，我们可根据需要掷出任何类型的“可掷”对象。典型情况下，我们要为每种不同类型的错误“掷”出一类不同的违例。我们的思路是在违例对象以及挑选的违例对象类型中保存信息，所以在更大场景中的某个人可知道如何对待我们的违例(通常，唯一的信息是违例对象的类型，而违例对象中保存的没什么意义)。

### 2. 违例的捕获

> 若某个方法产生一个违例，必须保证该违例能被捕获，并获得正确对待。对于Java 的违例控制机制，它的一个好处就是允许我们在一个地方将精力集中在要解决的问题上，然后在另一个地方对待来自那个代码内部的错误。为理解违例是如何捕获的，首先必须掌握“==警戒区==”的概念。<u>*它代表一个特殊的代码区域，有可能产生违例，并在后面跟随用于控制那些违例的代码*</u>。

#### 2.1 try块

若位于一个方法内部，并“掷”出一个违例(或在这个方法内部调用的另一个方法产生了违例)，那个方法就会在违例产生过程中退出。若不想一个 throw 离开方法，可在那个方法内部设置一个特殊的代码块，用它捕获违例。==try 块属于一种普通的作用域==， 用一个try关键字开头:

```java
try {
	// 可能产生违例的代码 
}
```

#### 2.2 违例控制器

当然，==生成的违例必须在某个地方中止。这个“地方”便是违例控制器或者违例控制模块==。而且*<u>针对想捕获的每种违例类型，都必须有一个相应的违例控制器</u>*。违例控制器紧接在 try 块后面，且用 catch关键字标记。如下所示:

```java
try {
	// Code that might generate exceptions
} catch(Type1 id1) {
	// Handle exceptions of Type1
} catch(Type2 id2) {
	// Handle exceptions of Type2
} catch(Type3 id3) {
	// Handle exceptions of Type3
}
```

每个 catch 从句——即违例控制器——都类似一个小型方法，它需要采用一个(而且只有一个)特定类型的自变量。可在控制器内部使用标识符(id1，id2 等等)，就象一个普通的方法自变量那样。我们有时也根本不使用标识符，因为违例类型已提供了足够的信息，可有效处理违例。但即使不用，标识符也必须就位。控制器必须“紧接”在 try 块后面。若“掷”出一个违例，违例控制机制就会搜寻自变量与违例类型相符的第一个控制器。随后，它会进入那个catch 从句，并认为违例已得到控制(==一旦catch 从句结束，对控制器的搜索也会停止==)。只有相符的catch 从句才会得到执行。*<u>在try 块内部，请注意大量不同的方法调用可能生成相同的违例，但只需要一个控制器</u>*。

> 中断与恢复：
>
> 在违例控制理论中，共存在两种基本方法。在“中断”方法中(Java 和 C++提供了对这种方法的支持)，我们假定错误非常关键，没有办法返回违例发生的地方。无论谁只要“掷”出一个违例，就表明没有办法补救错误，而且也不希望再回来。 另一种方法叫作“恢复”。它意味着违例控制器有责任来纠正当前的状况，然后取得出错的方法，假定下一次会成功执行。若使用恢复，意味着在违例得到控制以后仍然想继续执行。在这种情况下，我们的违例更象 一个方法调用——我们用它在 Java 中设置各种各样特殊的环境，产生类似于“恢复”的行为(换言之，此时不是“掷”出一个违例，而是调用一个用于解决问题的方法)。另外，也可以将自己的 try 块置入一个 while循环里，用它不断进入try块，直到结果满意时为止。 从历史的角度看，若程序员使用的操作系统支持可恢复的违例控制，最终都会用到类似于中断的代码，并跳过恢复进程。所以尽管“恢复”表面上十分不错，但在实际应用中却显得困难重重。其中决定性的原因可能是：我们的控制模块必须随时留意是否产生了违例，以及是否包含了由产生位置专用的代码。这便使代码很难编写和维护——大型系统尤其如此，因为违例可能在多个位置产生。

#### 2.3 违例规范

在Java 中，对那些要调用方法的客户程序员，我们要通知他们可能从自己的方法里“掷”出违例。这是一种有礼貌的做法，只有它才能使客户程序员准确地知道要编写什么代码来捕获所有潜在的违例。当然，若你同时提供了源码，客户程序员甚至能全盘检查代码，找出相应的throw语句。但尽管如此，通常并不随同源码提供库。为解决这个问题，Java 提供了一种特殊的语法格式(并强迫我们采用)，以便礼貌地告诉客户程序员该方法会“掷”出什么违例，令对方方便地加以控制。这便是我们在这里要讲述的“违例规范”，它属于方法声明的一部分，位于自变量(参数)列表的后面。违例规范采用了一个额外的关键字：throws;后面跟随全部潜在的违例类型。因此，我们的方法定义看起来应象下面这个样子:

```java
void f() throws tooBig, tooSmall, divZero {  // ... 
```

但不能完全依赖违例规范——假若方法造成了一个违例，但没有对其进行控制，编译器会侦测到这个情况，并告诉我们*<u>必须控制违例，或者指出应该从方法里“掷”出一个违例规范</u>*。通过坚持从顶部到底部排列违例规范，Java 可在编译期保证违例的正确性。

#### 2.4 捕获所有违例

我们可创建一个控制器，令其捕获所有类型的违例。具体的做法是捕获基础类违例类型 Exception(也存在其他类型的基础违例，但Exception是适用于几乎所有编程活动的基础)。如下所示:

```java
catch(Exception e) {
    System.out.println("caught an exception");
} 
```

> 这段代码能捕获任何违例，所以*<u>在实际使用时最好将其置于控制器列表的末尾，防止跟随在后面的任何特殊违例控制器失效</u>*。

对于程序员常用的所有违例类来说，由于Exception类是它们的基础，所以我们不会获得关于违例太多的信息，但可调用来自它的基础类 Throwable 的方法:

> String getMessage() 获得详细的消息。
>
> String toString()
> 返回对 Throwable 的一段简要说明，其中包括详细的消息(如果有的话)。
>
> void printStackTrace()
> void printStackTrace(PrintStream)
> 打印出 Throwable 和 Throwable 的调用堆栈路径。调用堆栈显示出将我们带到违例发生地点的方法调用的顺 序。
>
> 第一个版本会打印出标准错误，第二个则打印出我们的选择流程。若在 Windows 下工作，就不能重定向标准错误。因此，我们一般愿意使用第二个版本，并将结果送给 System.out;这样一来，输出就可重定向到我们 希望的任何路径。
>
> 除此以外，我们还==可从 Throwable 的基础类Object(所有对象的基础类型)获得另外一些方法==。对于违例控制来说，其中一个可能有用的是getClass()，它的作用是返回一个对象，用它代表这个对象的类。我们可依次用getName()或toString()查询这个Class类的名字。亦可对Class对象进行一些复杂的操作，尽管那些操作在违例控制中是不必要的。下面是一个特殊的例子，它展示了Exception方法的使用：

```java
//: ExceptionMethods.java
// Demonstrating the Exception Methods package c09;

public class ExceptionMethods {
    public static void main(String[] args) {
        try {
            throw new Exception("Here's my Exception"); 
        } catch(Exception e) {
            System.out.println("Caught Exception"); 
            System.out.println("e.getMessage(): " + e.getMessage());
            System.out.println("e.toString(): " + e.toString()); 
            System.out.println("e.printStackTrace():"); 
            e.printStackTrace();
        } 
    }
} ///:~
```

#### 2.5 重新“掷”出违例

> 在某些情况下，我们想重新掷出刚才产生过的违例，特别是在用Exception 捕获所有可能的违例时。由于我 们已拥有当前违例的句柄，所以只需简单地重新掷出那个句柄即可。下面是一个例子:

```java
catch(Exception e) {
    System.out.println("一个违例已经产生");
    throw e;
}
```


重新“掷”出一个违例导致违例进入更高一级环境的违例控制器中。用于同一个 try 块的任何更进一步的 catch 从句仍然会被忽略。此外，与违例对象有关的所有东西都会得到保留，所以用于捕获特定违例类型的更高一级的控制器可以从那个对象里提取出所有信息。若只是简单地重新掷出当前违例，我们打印出来的、与 printStackTrace()内的那个违例有关的信息会与违例的起源地对应，而不是与重新掷出它的地点对应。若想安装新的堆栈跟踪信息，可调用fillInStackTrace()，它会返回一个特殊的违例对象。这个违例的创建过程如下：将当前堆栈的信息填充到原来的违例对象里。下面列出它的形式:

```java
public class Rethrowing {
    public static void f() throws Exception {
        System.out.println("originating the exception in f()");
        throw new Exception("thrown from f()"); 
    }
    
    public static void g() throws Throwable { 
        try {
            f();
        } catch(Exception e) {
            System.out.println("Inside g(), e.printStackTrace()");
            e.printStackTrace();
            throw e; // 17
            // throw e.fillInStackTrace(); // 18
        } 
    }
    public static void main(String[] args) throws Throwable {
        try { 
            g();
        } catch(Exception e) { 
            System.out.println("Caught in main, e.printStackTrace()");
            e.printStackTrace();
        } 
    }
}
```

违例堆栈路径无论如何都会记住它的真正起点，无论自己被重复“掷”了好几次。若将第17行标注(变成注释行)，而撤消对第18行的标注，就会换用fillInStackTrace()。第18行成为违例的新起点。 针对g()和main()，Throwable类必须在违例规格中出现，因为fillInStackTrace()会生成一个Throwable 对象的句柄。由于 Throwable 是 Exception 的一个基础类，所以有可能获得一个能够“掷”出的对象(具有 Throwable属性)，但却并非一个Exception(违例)。因此，在main()中用于Exception的句柄可能丢失自己的目标。为保证所有东西均井然有序，编译器强制Throwable使用一个违例规范。举个例子来说，下述程序的违例便不会在 main()中被捕获到:

```java
public class ThrowOut {
    public static void main(String[] args) throws Throwable {
        try {
            throw new Throwable();
        } catch(Exception e) { 
            System.out.println("Caught in main()");
        }
    }
}
```

也可能从一个已经捕获的违例重新“掷”出一个不同的违例。但假如这样做，会得到与使用fillInStackTrace()类似的效果：与违例起源地有关的信息会全部丢失，我们留下的是与新的 throw 有关的信息。如下所示：

```java
public class RethrowNew {
    public static void f() throws Exception {
        System.out.println("originating the exception in f()");
        throw new Exception("thrown from f()"); }
    public static void main(String[] args) { 
        try {
            f();
        } catch(Exception e) {
            System.out.println("Caught in main, e.printStackTrace()");
            e.printStackTrace();
            throw new NullPointerException("from main"); 
        }
    }
}
```

最后一个违例只知道自己来自main()，而非来自f()。注意Throwable e在任何违例规范中都不是必须的。永远不必关心如何清除前一个违例，或者与之有关的其他任何违例。它们都属于用 new 创建的、以内存堆为 基础的对象，所以垃圾收集器会自动将其清除。

### 3. 标准Java违例

> Java包含了一个==Throwable==类，*<u>它对可以作为违例“掷”出的所有东西进行了描述</u>*。Throwable对象有两种常规类型(亦即“从Throwable继承”)。其中，==Error 代表编译期和系统错误==，我们一般不必特意捕获它们(除在特殊情况以外)。==Exception 是可以从任何标准 Java 库的类方法中“掷”出的基本类型==。此外，它们亦可从我们自己的方法以及运行期偶发事件中“掷”出。除名字外，一个违例和下一个违例之间并不存在任何特殊的地方。
>
> ==java.lang.Exception这是程序能捕获的基本违例==。其他违例都是从它衍生出去的。这里要注意的是违例的名字代表发生的问题，而且违例名通常都是精心挑选的，可以很清楚地说明到底发生了什么事情。

#### 3.1 RuntimeException的特殊情况

```java
if(t == null) {
    throw new NullPointerException();
}
```


看起来似乎在传递进入一个方法的每个句柄中都必须检查null(因为不知道调用者是否已传递了一个有效的 句柄)，我们根本不必这样做——它属于*<u>Java进行的标准运行期检查的一部分</u>*。*<u>若对一个空句柄发出了调用，Java会自动产生一个NullPointerException违例</u>*。所以上述代码在任何情况下都是多余的。这个类别里含有一系列违例类型。它们全部由 Java 自动生成，毋需我们亲自动手把它们包含到自己的违例规范里。最方便的是，通过将它们置入单独一个名为RuntimeException的基础类下面，它们全部组合到一起。 这是一个很好的继承例子：它建立了一系列具有某种共通性的类型，都具有某些共通的特征与行为。此外， 我们没必要专门写一个违例规范，指出一个方法可能会“掷”出一个 RuntimeException，因为已经假定可能出现那种情况。由于它们用于指出编程中的错误，所以几乎永远不必专门捕获一个“运行期违例”——RuntimeException——它在默认情况下会自动得到处理。若必须检查RuntimeException，我们的代码就会变得相当繁复。在我们自己的包里，可选择“掷”出一部分RuntimeException。 如果不捕获这些违例，又会出现什么情况呢？==由于编译器并不强制违例规范捕获它们，所以假如不捕获的话，一个RuntimeException可能过滤掉我们到达 main()方法的所有途径==。为体会此时发生的事情，请试试 下面这个例子:

```java
public class NeverCaught { 
    static void f() {
        throw new RuntimeException("From f()"); 
    }
    static void g() { 
        f();
    }
    public static void main(String[] args) {
        g();
    }
}
```

一个 RuntimeException(或者从它继承的任何东西)属于一种特殊情况，因为编译器不要求为这些类型指定违例规范。所以答案就是：假若一个RuntimeException获得到达main()的所有途径，同时不被捕获，那么当程序退出时，会为那个违例调用 printStackTrace()。注意也许能在自己的代码中仅忽略 RuntimeException，因为编译器已正确实行了其他所有控制。因为 RuntimeException 在此时代表一个编程错误:

(1) 一个我们不能捕获的错误(例如，由客户程序员接收传递给自己方法的一个空句柄)。
(2) 作为一名程序员，一个应在自己的代码中检查的错误(如 ArrayIndexOutOfBoundException，此时应注 意数组的大小)。
可以看出，最好的做法是在这种情况下违例，因为它们有助于程序的调试。 另外一个有趣的地方是，我们不可将Java违例划分为单一用途的工具。的确，它们设计用于控制那些讨厌的运行期错误——由代码控制范围之外的其他力量产生。但是，它也特别有助于调试某些特殊类型的编程错误，那些是编译器侦测不到的。

### 4. 创建自己的违例

并不一定非要使用Java违例。这一点必须掌握，因为经常都需要创建自己的违例，以便指出自己的库可能生成的一个特殊错误——但创建Java分级结构的时候，这个错误是无法预知的。 为创建自己的违例类，必须从一个现有的违例类型继承——最好在含义上与新违例近似。继承一个违例相当简单：

```java
class MyException extends Exception { 
    public MyException() {
    }
    public MyException(String msg) {
        super(msg); 
    }
}
```

由于违例不过是另一种形式的对象，所以可以继续这个进程，进一步增强违例类的能力。但要注意，对使用自己这个包的客户程序员来说，他们可能错过所有这些增强。因为他们可能只是简单地寻找准备生成的违例，除此以外不做任何事情——这是大多数 Java 库违例的标准用法。若出现这种情况，有可能创建一个新违例类型，其中几乎不包含任何代码:

```java
class SimpleException extends Exception {
}
```

它要依赖编译器来创建默认构建器(会自动调用基础类的默认构建器)。当然，在这种情况下，我们不会得 到一个SimpleException(String)构建器，但它实际上也不会经常用到。

### 5. 违例的限制



### 6. 用finally清除

无论一个违例是否在try块中发生，我们经常都想*<u>执行一些特定的代码</u>*。对一些特定的操作，经常都会遇到这种情况，但在恢复内存时一般都不需要（因为垃圾收集器会自动照料一切）。为达到这个目的，可在所有违例控制器的末尾使用一个finally从句。

```java
try {
    // 要保卫的区域：
    // 可能“掷”出A,B,或C的危险情况
} catch (A a1) {
	// 控制器 A
} catch (B b1) {
	// 控制器 B
} catch (C c1) {
	// 控制器 C
} finally {
	// 每次都会发生的情况
}
```

> ==无论是否“掷”出一个违例，finally从句都会执行。==

#### 6.1 用finally做什么

在没有“垃圾收集”以及“自动调用破坏器”机制的一种语言中，finally显得特别重要，因为程序员可用它<u>*担保内存的正确释放*</u>——无论在try块内部发生了什么状况。但Java提供了垃圾收集机制，所以内存的释放几乎绝对不会成为问题。另外，它也没有构建器可供调用。既然如此，Java里何时才会用到finally呢？

> 除将内存设回原始状态以外，若要设置另一些东西，finally就是必需的。例如，我们有时需要打开一个文件或者建立一个网络连接，或者在屏幕上画一些东西，甚至设置外部世界的一个开关，等等。如下例所示:

```java
class Switch {
    boolean state = false;
    boolean read() { return state; }
    void on() { state = true; }
    void off() { state = false; }
}

public class OnOffSwitch {
    static Switch sw = new Switch();
    public static void main(String[] args) {
        try {
            sw.on();
            // Code that can throw exceptions...
            sw.off();
        } catch(NullPointerException e) {
            System.out.println("NullPointerException");
            sw.off();
        } catch(IllegalArgumentException e) {
            System.out.println("IOException");
            sw.off();
        }
    }
}
```

这里的目标是保证main()完成时开关处于关闭状态，所以将sw.off()置于每一个违例控制器的末尾。但产生的一个违例有肯能不是在这里捕获的，这便会错过sw.off()。利用finally，我们可以将来自try块的关闭代码只置于一个地方。

```java
class Switch {
    boolean state = false;
    boolean read() { return state; }
    void on() { state = true; }
    void off() { state = false; }
}

public class OnOffSwitch {
    static Switch sw = new Switch();
    public static void main(String[] args) {
        try {
            sw.on();
            // Code that can throw exceptions...
        } catch(NullPointerException e) {
            System.out.println("NullPointerException");
        } catch(IllegalArgumentException e) {
            System.out.println("IOException");
        } finally {
            sw.off()
        }
    }
}
```

> 注意：若调用了break和continue语句，finally语句也会得以执行。请注意，与作上标签的break和continue一道，finally排除了Java对goto跳转语句的需求。

#### 6.2 缺点：丢失的违例

尽管违例指出程序里存在一个危机，而且绝不应忽略，但一个违例仍有可能简单地“丢失”。在采用finally从句的一种特殊配置下，便有可能发生这种情况：

```java
class VeryImportantException extends Exception {
    public String toString() {
        return "A very important exception!";
    }
}

class HoHumException extends Exception {
    public String toString() {
        return "A trivial exception";
    }
}

public class LostMessage {
    void f() throws VeryImportantException {
        throw new VeryImportantException();
    }
    void dispose() throws HoHumException {
        throw new HoHumException();
    }
    public static void main(String[] args) throws Exception {
        LostMessage lm = new LostMessage();
        try {
            lm.f();
        } finally {
            lm.dispose();
        }
    }
}
```

```
Exception in thread "main" A trivial exception
	at com.chance.basis.exception.LostMessage.dispose(LostMessage.java:15)
	at com.chance.basis.exception.LostMessage.main(LostMessage.java:23)
```

可以看到，这里不存在VeryImportantException（非常重要的违例）的迹象，它只是简单地被finally从句中的HoHumException代替了。

### 7.构建器

构建器将对象置于一个安全的起始状态，但它可能执行一些操作——如打开一个文件。除非用户完成对象的使用，并调用一个特殊的清除方法，否则那些操作不会得到正确的清除。若从一个构建器内部“掷”出一个违例，这些清除行为也可能不会正确地发生。所有这些都意味着在编写构建器时，我们必须特别加以留意。

由于前面刚学了finally，所以大家可能认为它是一种合适的方案。但事情并没有这么简单，因为finally每次都会执行清除代码——即使我们在清除方法运行之前不想执行清除代码。因此，假如真的用finally进行清除，必须在构建器正常结束时设置某种形式的标志。而且只要设置了标志，就不要执行finally块内的任何东西。由于这种做法并不完美（需要将一个地方的代码同另一个地方的结合起来），所以除非特别需要，否则一般不要尝试在finally中进行这种形式的清除。

在下面这个例子里，我们创建了一个名为InputFile的类。它的作用是打开一个文件，然后每次读取它的一行内容（转换为一个字串）。它利用了由Java标准IO库提供的FileReader以及BufferedReader类（将于第10章讨论）。这两个类都非常简单，大家现在可以毫无困难地掌握它们的基本用法：

```java
class InputFile {
    private BufferedReader in;
    InputFile(String fname) throws Exception {
        try {
            in = new BufferedReader(new FileReader(fname));
            // Other code that might throw exceptions
        } catch(FileNotFoundException e) {
            System.out.println("Could not open " + fname);
            // Wasn't open, so don't close it
            throw e;
        } catch(Exception e) {
            // All other exceptions must close it
            try {
                in.close();
            } catch(IOException e2) {
                System.out.println("in.close() unsuccessful");
            }
            throw e;
        } finally {
            // Don't close it here!!!
        }
    }
    String getLine() {
        String s;
        try {
            s = in.readLine();
        } catch(IOException e) {
            System.out.println("readLine() unsuccessful");
            s = "failed";
        }
        return s;
    }
    void cleanup() {
        try {
            in.close();
        } catch(IOException e2) {
            System.out.println("in.close() unsuccessful");
        }
    }
}

public class Cleanup {
    public static void main(String[] args) {
        try {
            InputFile in = new InputFile("Cleanup.java");
            String s;
            int i = 1;
            while((s = in.getLine()) != null)
                System.out.println(""+ i++ + ": " + s);
            in.cleanup();
        } catch(Exception e) {
            System.out.println("Caught in main, e.printStackTrace()");
            e.printStackTrace();
        }
    }
}
```

该例使用了Java 1.1 IO类。

用于InputFile的构建器采用了一个String（字串）参数，它代表我们想打开的那个文件的名字。在一个try块内部，它用该文件名创建了一个FileReader。对FileReader来说，除非转移并用它创建一个能够实际与之“交谈”的BufferedReader，否则便没什么用处。注意InputFile的一个好处就是它同时合并了这两种行动。

若FileReader构建器不成功，就会产生一个FileNotFoundException（文件未找到违例）。必须单独捕获这个违例——这属于我们不想关闭文件的一种特殊情况，因为文件尚未成功打开。其他任何捕获从句（catch）都必须关闭文件，因为文件已在进入那些捕获从句时打开（当然，如果多个方法都能产生一个FileNotFoundException违例，就需要稍微用一些技巧。此时，我们可将不同的情况分隔到数个try块内）。close()方法会掷出一个尝试过的违例。即使它在另一个catch从句的代码块内，该违例也会得以捕获——对Java编译器来说，那个catch从句不过是另一对花括号而已。执行完本地操作后，违例会被重新“掷”出。这样做是必要的，因为这个构建器的执行已经失败，我们不希望调用方法来假设对象已正确创建以及有效。

在这个例子中，没有采用前述的标志技术，finally从句显然不是关闭文件的正确地方，因为这可能在每次构建器结束的时候关闭它。由于我们希望文件在InputFile对象处于活动状态时一直保持打开状态，所以这样做并不恰当。

getLine()方法会返回一个字串，其中包含了文件中下一行的内容。它调用了readLine()，后者可能产生一个违例，但那个违例会被捕获，使getLine()不会再产生任何违例。对违例来说，一项特别的设计问题是决定在这一级完全控制一个违例，还是进行部分控制，并传递相同（或不同）的违例，或者只是简单地传递它。在适当的时候，简单地传递可极大简化我们的编码工作。

getLine()方法会变成：

```java
String getLine() throws IOException {
    return in.readLine();
}
```

但是当然，调用者现在需要对可能产生的任何IOException进行控制。

用户使用完毕InputFile对象后，必须调用cleanup()方法，以便释放由BufferedReader以及／或者FileReader占用的系统资源（如文件句柄）。除非InputFile对象使用完毕，而且到了需要弃之不用的时候，否则不应进行清除。大家可能想把这样的机制置入一个finalize()方法内，但正如第4章指出的那样，并非总能保证finalize()获得正确的调用（即便确定它会调用，也不知道何时开始）。这属于Java的一项缺陷——除内存清除之外的所有清除都不会自动进行，所以必须知会客户程序员，告诉他们有责任用finalize()保证清除工作的正确进行。

在Cleanup.java中，我们创建了一个InputFile，用它打开用于创建程序的相同的源文件。同时一次读取该文件的一行内容，而且添加相应的行号。所有违例都会在main()中被捕获——尽管我们可选择更大的可靠性。

### 8. 违例匹配

“掷”出一个违例后，违例控制系统会按当初编写的顺序搜索“最接近”的控制器。一旦找到相符的控制器，就认为违例已得到控制，不再进行更多的搜索工作。

> 违例准则：
>
> (1) 解决问题并再次调用造成违例的方法。
>
> (2) 平息事态的发展，并在不重新尝试方法的前提下继续。
>
> (3) 计算另一些结果，而不是希望方法产生的结果。
>
> (4) 在当前环境中尽可能解决问题，以及将相同的违例重新“掷”出一个更高级的环境。
>
> (5) 在当前环境中尽可能解决问题，以及将不同的违例重新“掷”出一个更高级的环境。
>
> (6) 中止程序执行。
>
> (7) 简化编码。若违例方案使事情变得更加复杂，那就会令人非常烦恼，不如不用。
>
> (8) 使自己的库和程序变得更加安全。这既是一种“短期投资”（便于调试），也是一种“长期投资”（改善应用程序的健壮性）

### 9. 总结

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gj0m5va2n9j30ru0q40w3.jpg" style="zoom:60%">



