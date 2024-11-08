在 Java 中，`classpath` 是一个用于指定 Java 程序运行时所需的类文件和资源路径的参数。它告诉 Java 虚拟机（JVM）在哪里查找需要加载的类文件（`.class`）或 JAR 文件。在开发和运行 Java 应用程序时，`classpath` 的正确配置非常重要，尤其是在项目依赖多个库或资源的情况下。

### 一、classpath 的作用

---

当 JVM 运行一个 Java 程序时，它需要知道从哪里加载类文件或资源文件。**默认情况下，JVM 只会在当前目录下查找这些文件**。通过 `classpath`，你可以告诉 JVM 额外的目录、JAR 文件或库位置，这样它就能够找到并加载程序所依赖的类。



### 二、如何设置 classpath

---

#### 2.1. 在命令行中设置

当使用命令行运行 Java 程序时，可以通过 `-classpath`（或 `-cp`）选项来指定 `classpath`。

```bash
java -classpath /path/to/classes:/path/to/lib/some-library.jar com.example.MyProgram
```

上面的命令告诉 JVM 在 `/path/to/classes` 目录和 `some-library.jar` 文件中查找 `MyProgram` 所需的类。

- 路径之间用 **冒号 (`:`)** 分隔（在 UNIX/Linux/MacOS 系统中）。

- 在 Windows 系统中，使用 

  分号 (`;`)

   作为分隔符：

  ```
  bash
  
  
  复制代码
  java -classpath C:\path\to\classes;C:\path\to\lib\some-library.jar com.example.MyProgram
  ```

#### 2.2 通过环境变量设置

可以通过设置系统的 `CLASSPATH` 环境变量来指定 `classpath`。例如，在 UNIX/Linux 系统的 shell 中：

```bash
export CLASSPATH=/path/to/classes:/path/to/lib/some-library.jar
```

然后你可以直接运行 Java 程序，而无需每次都指定 `-classpath` 选项。

```bash
java com.example.MyProgram
```

然而，使用环境变量设置 `CLASSPATH` 并不常见，因为它对整个系统生效，容易导致不同程序之间的冲突。

#### 2.3 在 IDE 中设置

在大多数集成开发环境（IDE）中（例如 IntelliJ IDEA、Eclipse、NetBeans），`classpath` 通常通过项目设置来管理。在这些 IDE 中，添加外部库、JAR 文件或模块依赖项会自动更新 `classpath`。

### 三、classpath 的内容

---

`classpath` 可以包含以下内容：

- **目录**：指定某个包含 `.class` 文件的目录。例如：

  ```bash
  java -classpath /path/to/classes com.example.MyProgram
  ```

- **JAR 文件**：指定单个或多个 JAR 文件。JAR 文件是 Java 类文件和资源的打包格式。例如：

  ```bash
  java -classpath /path/to/lib/some-library.jar com.example.MyProgram
  ```

- **通配符（Wildcard）**：可以使用 `*` 通配符来指定某个目录下的所有 JAR 文件（但不会递归子目录）。例如：

  ```bash
  java -classpath "/path/to/lib/*" com.example.MyProgram
  ```



### 四、默认 classpath

---

如果不指定 `classpath`，JVM 会使用当前工作目录作为默认的 `classpath`。也就是说，它会在当前目录下查找类文件。

```bash
java com.example.MyProgram
```

在这种情况下，JVM 会在当前目录中查找 `com/example/MyProgram.class`。



### 五、classpath 中的优先级

---

如果 `classpath` 中的不同路径包含相同的类名，JVM 会按照 `classpath` 中的路径顺序加载第一个找到的类。因此，顺序很重要。



### 六、总结

---

- `classpath` 是告诉 JVM 从哪里加载类和资源的参数。
- 可以通过命令行选项 `-classpath` 或系统环境变量 `CLASSPATH` 来设置。
- `classpath` 中可以包含目录、JAR 文件，或者使用通配符指定一组 JAR 文件。
- 在 IDE 中，通常通过项目设置来管理 `classpath`，而无需手动配置。