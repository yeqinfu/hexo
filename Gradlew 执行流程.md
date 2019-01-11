### Gradlew 执行流程

当我们执行

```
gradlew assembleRelease
```

到底发生了什么？打算从这个角度去分析，gradle在构建的时候发生的事情。

gradlew 在Terminal执行的时候，实际上执行的是一个sh脚本，win对应的是一个bat脚本，这里从gradlew shell脚本出发。执行gradlew --help可以看到gradlew 脚本所有提供的命令功能。

![image](https://ws2.sinaimg.cn/large/c1b251b3gy1fz05druv2wj20ni0htmym.jpg)



可以看到，gradlew 后面跟着是一个task的名字，当然中间可以插入一下option配置。所以可以确定assembleRelease这是一个gradle系统自带的task。

然后，看了一下gradlew 脚本做了什么，发现其实也就是在区别一些不同环境下的JAVA_HOME CLASSPATH的取值等等，还有其他。

由于我的环境是win环境，这里又改成看bat脚本，里面的逻辑都差不多，最精华的是这句

![image](https://ws1.sinaimg.cn/large/c1b251b3gy1fz066l1a5yj212b04odg2.jpg)

我在前面加了echo看看执行的整体是什么命令。而后面的变量配置都是上面脚本逻辑加入进来的

```
"C:\Program Files\Java\jdk1.8.0_131/bin/java.exe"    "-Dorg.gradle.appname=gradlew" -classpath "E:\WorkSpace\repo\branch\BZWv142\\gradle\wrapper\gradle-wrapper.jar" org.gradle.wrapper
.GradleWrapperMain assembleRelease
```

输出了以上的内容，也就是我在执行assembleRelease的时候，bat脚本处理后变成了这些。它自动找到了JAVA_HOME的环境变量，还有JVM配置参数。

所以这里看第一个入参

#### DEFAULT_JVM_OPTS

```
set DEFAULT_JVM_OPTS="-Xmx512m"
```

似乎应该是用于设置jvm堆内存的，工程比较大的时候，有时候需要配置大一点才能编译通过，这个在gradle.properties也可以配置

#### JAVA_OPTS

也是空的，

#### GRADLE_OPTS

```
set GRADLE_OPTS="$GRADLE_OPTS -Xmx512m"
```

似乎也是一样的功能，可能还有别的用户，不深究。



#### GradleWrapperMain

最后就是调用这个java方法把我们输入的task名字一个不差的搬进去。



![image](https://wx1.sinaimg.cn/large/c1b251b3gy1fz0bemvym0j213v0em0uf.jpg)





接下来简单了解一下gradlewrappermain这个类的内容，看看逻辑就知道gradle的配置文件是如何被整合在一起的了。

```java
 File wrapperJar = wrapperJar();
 File propertiesFile = wrapperProperties(wrapperJar);
 File rootDir = rootDir(wrapperJar);
```

前两句分别找到了wrapperjar的jar文件对象和同级目录的配置文件，也就是一下内容

```
#Mon Dec 28 10:00:20 PST 2015
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-2.14.1-all.zip
```



```
 private static File rootDir(File wrapperJar) {
    return wrapperJar.getParentFile().getParentFile().getParentFile();
  }
```

第三句找到功能根目录，也是很暴力嘛~

```java
  private static void addSystemProperties(File gradleHome, File rootDir)
  {
    System.getProperties().putAll(SystemPropertiesHandler.getSystemProperties(new File(gradleHome, "gradle.properties")));
    System.getProperties().putAll(SystemPropertiesHandler.getSystemProperties(new File(rootDir, "gradle.properties")));
  }
```

从这句可以看出，gradle.properties这个配置文件，在工程目录下有一个会被载入，还有一个在系统用户下的.gradle文件夹下的配置也会被加载进来作为全局配置。

> 所以以后需要用到全局配置就可以放在当前用户目录下了

```java

public class BootstrapMainStarter
{
  public void start(String[] args, File gradleHome)
    throws Exception
  {
    File gradleJar = findLauncherJar(gradleHome);
    URLClassLoader contextClassLoader = new URLClassLoader(new URL[] { gradleJar.toURI().toURL() }, ClassLoader.getSystemClassLoader().getParent());
    Thread.currentThread().setContextClassLoader(contextClassLoader);
    Class mainClass = contextClassLoader.loadClass("org.gradle.launcher.GradleMain");
    Method mainMethod = mainClass.getMethod("main", new Class[] { [Ljava.lang.String.class });
    mainMethod.invoke(null, new Object[] { args });
  }

  private File findLauncherJar(File gradleHome) {
    File[] arr$ = new File(gradleHome, "lib").listFiles(); int len$ = arr$.length; for (int i$ = 0; i$ < len$; ++i$) { File file = arr$[i$];
      if (file.getName().matches("gradle-launcher-.*\\.jar"))
        return file;
    }

    throw new RuntimeException(String.format("Could not locate the Gradle launcher JAR in Gradle distribution '%s'.", new Object[] { gradleHome }));
  }
}
```



逻辑一直往下走，发现执行了这个GradleMain并且进行了反射调用。那这个类在哪里呢？在类似这个目录下

```
C:\Users\Administrator\.gradle\wrapper\dists\gradle-2.14.1-all\8bnwg5hd3w55iofp58khbp6yv\gradle-2.14.1\lib
```

发现这里真的是好多jar包，这啥时候看到头呢？这样子确实不知道啥时候才能跟项目工程里面的gradle脚本联系起来。

```java
public class GradleMain
{
  public static void main(String[] args)
    throws Exception
  {
    new ProcessBootstrap().run("org.gradle.launcher.Main", args);
  }
}
```



最后还是发现别人写的好

[参考](https://blog.csdn.net/verymrq/article/details/80564048)







































































