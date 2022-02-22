#### 1. 制作只含有字节码文件的jar包

---

制作只含有字节码文件的jar包。

##### 1.1 直接输出hello

> 创建java文件
>
> `$ vim Hello.java`

```java
class Hello{
  public static void main(String[] agrs){
    System.out.println("hello");
  }
}
```

> 编译
>
> `$ javac Hello.java`

>将编译后的class文件打成jar包。
>
>```
>Manifest-Version: 1.0
>Created-By: 1.8.0_221 (Oracle Corporation)
>Main-Class: Hello
>```
>
>`$ jar -cvfm hello.jar MANIFEST.MF Hello.class`
>
>c表示要创建一个新的jar包，v表示创建的过程中在控制台输出创建过程的一些信息，f表示给生成的jar包命名，m指定需要包含的 MANIFEST 清单文件 。

>运行jar包
>
>`java -jar hello.jar`

##### 1.2 含有两个类的jar包——通过调用输出hello

>创建java文件

```java
class Hello{
  public static void main(String[] agrs){
    Tom.speak();
  }
}

class Tom{
  public static void speak(){
    System.out.println("hello");
  }
}
```

>编译（此时Hello.java和Tom.java同时被编译，因为Hello中调用了Tom，在编译的过程中发现还需要编译Tom）
>
>`$ javac hello.java`

>打jar包
>
>`$ jar -cvfm hello.jar MANIFEST.MF Hello.class Tom.class`

>运行
>
>`$ java -jar hello.jar`

##### 1.3 有目录结构的jar包——通过引包并调用输出hello

将上一个稍稍变化一下，将Tom这个类放在com包下，源文件目录结构变成

com
	Tom.java
	Hello.java

同时Tom.java需要在第一行声明自己的包名`package com;`

Hello.java需要引入Tom这个类，同样要在第一行进行import`import com.Tom;`

>编译
>
>`$ javac hello.java`

>打jar包
>
>`$ jar -cvfm hello.jar MANIFEST.MF Hello.class com`

>运行
>
>`$ java -jar hello.jar`

>优化过程
>
>com包下是有Tom.java源文件的，也被打进了jar包里，这样不太好，优化一下javac命令，使所有的编译后文件编译到另一个隔离的地方。
>
>在编译Hello.java时，先新建一个target文件夹。然后我们用如下命令
>
>`javac Hello.java -d target`
>
>该命令表示，将所有编译后的文件，都放到target文件夹下。
>
>将META-INF文件夹也复制到target目录下，进入这个目录，输入如下命令
>
>`jar -cvfm hello.jar META-INF\MENIFEST.MF *` 
>
>注意最后一个位置变成了，表示把当前目录下所有文件都打在jar包里。

>javac 要编译的文件 -d 目标位置
>
>jar -cvfm 命名 MENIFEST文件 要打包的文件1 要打包的文件2

#### 2. 制作含有jar文件的jar包

---

我们将场景稍稍变得复杂一点，看看jar包中需要引入其他jar包的场景。

##### 2.1 两个jar包间相互调用——调用jar外的jar输出hello

最终生成的jar包结构

> hello.jar
> tom.jar

方法步骤

准备：将上述一中写好的那个不带包的tom.jar复制过来（目的是调用里面的speak方法）

（1）编写一个Hello.java并将其编译成Hello.class，注意，由于Hello里面引用了Tom类的speak方法，因此在打jar包时应使用-cp参数，将tom.jar包引入

　　 javac -cp tom.jar Hello.class 

　　这里的 -cp 表示 -classpath，指的是把tom.jar加入classpath路径下

（2）将hello.class达成jar包，步骤略

（3）此时运行 java -jar 发现报错 ClassNotFoundException：Tom 

　　原因很简单，引入jar包需要在MENIFEST.MF文件中配置一个新属性：Class-Path，路径指向你需要的所有jar包

　　现在MENIFEST.MF这个文件应该变成

```
Manifest-Version: 1.0
Created-By: 1.8.0_121 (Oracle Corporation)
Main-Class: Hello
Class-Path: Tom.jar
```

（4）好了，修改这个文件，再次运行，发现成功在控制台输出 hello 

tips：引入多个jar包，中间用空格隔开

> 至此，我们可以总结出，命令变化如下
>
> javac -cp xxx.jar 要编译的文件 -d 目标位置
>
> jar -cvfm 命名 MENIFEST文件 要打包的文件1 要打包的文件2

##### 2.2 jar包中含有jar包——调用jar内的jar输出hello

最终生成的jar包结构

> META-INF
> Hello.class
> tom.jar

当项目中我们把所需要的第三方jar包也打进了我们自己的jar包中时，如果仍然按照上述操作方式，会报找不到Class异常。原因就是jar引用不到放在自己内部的jar包。

#### 3. 制作含有资源文件的jar包

---

##### 3.1 资源文件在jar包内部——读取jar内的文件

最终生成的jar包结构

> META-INF
> Hello.class
> text.txt

```java
import java.io.InputStream;
import java.io.BufferedReader;
import java.io.InputStreamReader;

class Hello{
  public static void main(String[] args) throws Exception{
    Hello hello = new Hello();
    InputStream is = hello.getClass().getResourceAsStream("text.txt");
    print(is);
  }

  /**
   * 读取文件，输出里面的内容，通用方法
   */
  public static void print(InputStream inputStream) throws Exception {
    InputStreamReader reader = new InputStreamReader(inputStream, "utf-8");
    BufferedReader br = new BufferedReader(reader);
    String s = "";
    while ((s = br.readLine()) != null)
      System.out.println(s);
    inputStream.close();
  }
}
```

##### 3.2 资源文件在另一个jar包内部——读取另一个jar内的文件

最终生成的jar包结构

> hello.jar
> resource.jar
> 　text.txt

方法步骤

同1一样，只不过需要在MENIFEST文件中将resource.jar加入classpath

```java
import java.io.InputStream;
import java.io.BufferedReader;
import java.io.InputStreamReader;

class Hello{
  public static void main(String[] args) throws Exception{
    Hello hello = new Hello();
    InputStream is = hello.getClass().getResourceAsStream("text.txt");
    print(is);
  }

  /**
   * 读取文件，输出里面的内容，通用方法
   */
  public static void print(InputStream inputStream) throws Exception {
    InputStreamReader reader = new InputStreamReader(inputStream, "utf-8");
    BufferedReader br = new BufferedReader(reader);
    String s = "";
    while ((s = br.readLine()) != null)
      System.out.println(s);
    inputStream.close();
  }
}
```

##### 3.3 资源文件在jar包外部——读取jar外的文件

最终生成的jar包结构

> hello.jar
> text.txt

 方法步骤

```java
import java.io.InputStream;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.FileInputStream;

class Hello{
  public static void main(String[] args) throws Exception{
    Hello hello = new Hello();
    InputStream is = new FileInputStream("text.txt");
    print(is);
  }

  /**
   * 读取文件，输出里面的内容，通用方法
   */
  public static void print(InputStream inputStream) throws Exception {
    InputStreamReader reader = new InputStreamReader(inputStream, "utf-8");
    BufferedReader br = new BufferedReader(reader);
    String s = "";
    while ((s = br.readLine()) != null)
      System.out.println(s);
    inputStream.close();
  }
}
```