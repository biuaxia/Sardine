title: 【转载】原来 java 注解只是个标记，没什么本领，一文精通，值得收藏！
date: 2021-08-16 09:18:01
comment: false
toc: true
category:
 - Java
tags:
 - 转载
 - Java
 - 注解
 - 标记
 - 本领
 - 收藏
 - 精通
---

本文转载自：[原来 java 注解只是个标记，没什么本领，一文精通，值得收藏！](https://juejin.cn/post/6996586907010334733?utm_source=gold_browser_extension)

---

![镇楼](https://b3logfile.com/file/2021/08/solo-fetchupload-3905946659787129734-d2817141.webp)

---

# 前言

java里有个神奇的存在，注解，就是那个天天@别人的家伙，它到底是何方神圣啊？
---

# 什么是注解

> 从JDK5开始，Java增加对元数据的支持，也就是注解，注解与注释是有一定区别的，可以把注解理解为代码里的特殊标记，这些标记可以在编译，类加载，运行时被读取，并执行相应的处理。通过注解开发人员可以在不改变原有代码和逻辑的情况下在源代码中嵌入补充信息。

---

# 内置的注解

二当家的发现在 IDE 中如果创建一个类并实现一个接口之后，那些实现接口的方法上面会自动帮我添加 @Override 的标记。

而这个标记就是注解，像@Override这样JDK内置的注解还有好几个呢，他们都在java.lang包下面，我们分别看看。

## @Override

当子类重写父类方法时，子类可以加上这个注解，那这有什么什么用？这可以确保子类确实重写了父类的方法，避免出现低级错误。

![](https://b3logfile.com/file/2021/08/solo-fetchupload-5250466463452894545-9a4dccb3.webp)

开始不知道有什么用，直到有一天，我把接口里的这个方法删除了，结果就编译不通过了。

![](https://b3logfile.com/file/2021/08/solo-fetchupload-5446808499488518185-93973e34.webp)

下面是直接在命令行编译的结果。这回我知道这个标记的作用了，它可以告诉编译器和阅读源码的人这个方法是覆盖或实现超类型的方法。

![](https://b3logfile.com/file/2021/08/solo-fetchupload-1342783522270693941-c85a2833.webp)

## @Deprecated

这个注解用于表示某个程序元素类，方法等已过时，当其他程序使用已过时的类，方法时编译器会给出警告（删除线，这个见了不少了吧）。

![](https://b3logfile.com/file/2021/08/solo-fetchupload-129714643609347048-c9541090.webp)

## @SuppressWarnings

抑制警告。

```java
package com.secondgod.annotation;

import java.util.ArrayList;
import java.util.List;

public class Test {

    public static void main(String[] args) {
        List<Integer> intList = new ArrayList<>();
        List          objList = intList;
        objList.add("二当家的");
    }
}
```

在命令行启用建议警告的编译(命令形如 javac -d . -Xlint Test.java ,下文再提到启用建议警告的编译，不再赘述解释)。

![](https://b3logfile.com/file/2021/08/solo-fetchupload-2973121356256192257-79bad038.webp)

代码稍作修改，则可以消除编译警告。

```java
package com.secondgod.annotation;

import java.util.ArrayList;
import java.util.List;

public class Test {

    @SuppressWarnings({"unchecked", "rawtypes"})
    public static void main(String[] args) {
        List<Integer> intList = new ArrayList<>();
        List          objList = intList;
        objList.add("二当家的");
    }
}
```

## @SafeVarargs

在声明具有模糊类型（比如：泛型）的可变参数的构造函数或方法时，Java编译器会报unchecked警告。鉴于这些情况，如果程序员断定声明的构造函数和方法的主体不会对其varargs参数执行潜在的不安全的操作，可使用@SafeVarargs进行标记，这样的话，Java编译器就不会报unchecked警告。

```java
package com.secondgod.annotation;

import java.util.List;

public class Test {
    public final void test(List<String>... stringLists) {

    }
}
```

在命令行启用建议警告的编译。

![](https://b3logfile.com/file/2021/08/solo-fetchupload-5180740535706986392-8dd8783f.webp)

代码稍作修改，则可以消除编译警告。

```java
package com.secondgod.annotation;

import java.util.List;

public class Test {
    @SafeVarargs
    public final void test(List<String>... stringLists) {

    }
}
```

## @FunctionalInterface

函数式接口注解，这个是 Java 1.8 版本引入的新特性。函数式编程很火，所以 Java 8 也及时添加了这个特性。
这个注解有什么用？这个注解保证这个接口只有一个抽象方法，注意这个只能修饰接口。

1. 没有抽象方法的接口

```java
package com.secondgod.annotation;

@FunctionalInterface
public interface Test {
    
}
```

![](https://b3logfile.com/file/2021/08/solo-fetchupload-3535568705720552085-844ad37d.webp)

2. 有超过一个抽象方法的接口

```java
package com.secondgod.annotation;

@FunctionalInterface
public interface Test {
    String test1();

    String test2();
}
```

![](https://b3logfile.com/file/2021/08/solo-fetchupload-999147028488548571-080bf259.webp)

3. 只有一个抽象方法的接口

编译通过。

```java
package com.secondgod.annotation;

@FunctionalInterface
public interface Test {
    String test();
}
```

4. 只有一个抽象方法的抽象类

```java
package com.secondgod.annotation;

@FunctionalInterface
public abstract class Test {
    public abstract String test();
}
```

![](https://b3logfile.com/file/2021/08/solo-fetchupload-8362710045724233580-0c2cb950.webp)

---

# 元注解

我们打开@Override的源码看下，会发现这个注解的定义上本身还有注解，即注解的注解，人们通常叫他们元注解。元注解是负责对其它注解进行说明的注解，自定义注解时可以使用元注解。

![](https://b3logfile.com/file/2021/08/solo-fetchupload-4996815152125325547-22f0a235.webp)

JDK内置的元注解都在java.lang.annotation包下面。

![](https://b3logfile.com/file/2021/08/solo-fetchupload-2701543448979420360-787668c3.webp)

## @Target

Target 是目标的意思，@Target 指定了注解运用的地方。

| 取值            | 注解使用范围                               |
| ----------------- | -------------------------------------------- |
| TYPE            | 可用于类或者接口上                         |
| FIELD           | 可用于域/字段/属性上                       |
| METHOD          | 可用于方法上                               |
| PARAMETER       | 可用于参数上                               |
| CONSTRUCTOR     | 可用于构造方法上                           |
| LOCAL_VARIABLE  | 可用于局部变量上                           |
| ANNOTATION_TYPE | 可用于注解类型上（被@interface修饰的类型） |
| PACKAGE         | 用于记录java文件的package信息              |
| TYPE_PARAMETER  | 可用于类型变量的声明语句中                 |
| TYPE_USE        | 可用于使用类型的任何语句中                 |

## @Documented

用 @Documented 注解修饰的注解类会被 JavaDoc 工具提取成文档。默认情况下，JavaDoc 是不包括注解的，但如果声明注解时指定了 @Documented，就会被 JavaDoc 之类的工具处理，所以注解类型信息就会被包括在生成的帮助文档中。

## @Inherited

Inherited 是继承的意思，但是它并不是说注解本身可以继承，而是说如果一个超类被 @Inherited 注解过的注解进行注解的话，那么如果它的子类没有被任何注解应用的话，那么这个子类就继承了超类的注解。

## @Repeatable

Repeatable 自然是可重复的意思。Java 8 新增加的，它允许在相同的程序元素中重复注解，在需要对同一种注解多次使用时，往往需要借助 @Repeatable 注解。Java 8 版本以前，同一个程序元素前最多只能有一个相同类型的注解，如果需要在同一个元素前使用多个相同类型的注解，则必须使用注解“容器”。

## @Retention

@Retention 用于描述注解的生命周期，也就是该注解被保留的时间长短。@Retention 注解中的成员变量（value）用来设置保留策略，value 是 java.lang.annotation.RetentionPolicy 枚举类型，RetentionPolicy 有 3 个枚举常量，如下所示。

| 取值    | 生命周期                                             |
| --------- | ------------------------------------------------------ |
| SOURCE  | 在源文件中有效（编译器可以使用）                     |
| CLASS   | 在 class 文件中有效（编译器和虚拟机可以使用）        |
| RUNTIME | 在运行时有效（编译器，虚拟机，程序运行时都可以使用） |

## @Native

使用 @Native 注解修饰成员变量，则表示这个变量可以被本地代码引用，常常被代码生成工具使用。他也在java.lang.annotation包下面，但是我认为他不算是元注解，因为他的使用目标不是注解。

![](https://b3logfile.com/file/2021/08/solo-fetchupload-3774953892529960831-34861ba9.webp)

---

# 自定义注解

在Spring，Hibernate，Mybatis等框架下都有注解，二当家的也尝试自定义一个注解，先看下@Override的源码吧。

```java
package java.lang;

import java.lang.annotation.*;

/**
 * Indicates that a method declaration is intended to override a
 * method declaration in a supertype. If a method is annotated with
 * this annotation type compilers are required to generate an error
 * message unless at least one of the following conditions hold:
 *
 * <ul><li>
 * The method does override or implement a method declared in a
 * supertype.
 * </li><li>
 * The method has a signature that is override-equivalent to that of
 * any public method declared in {@linkplain Object}.
 * </li></ul>
 *
 * @author  Peter von der Ah&eacute;
 * @author  Joshua Bloch
 * @jls 9.6.1.4 @Override
 * @since 1.5
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```

我们照葫芦画瓢也自定义一个。

```java
package com.secondgod.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 测试注解
 *
 * @author 二当家的白帽子 https://le-yi.blog.csdn.net/
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
}
```

我们自定义的注解通常都是定义生命周期为 RUNTIME 的。生命周期为 SOURCE 的，需要编译器的配合。生命周期为 CLASS 的，需要虚拟机的配合。只有程序运行时的行为是我们可以自定义的。

好了，去使用一下这个自定义的注解。

```java
package com.secondgod.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 测试注解
 *
 * @author 二当家的白帽子 https://le-yi.blog.csdn.net/
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@interface MyAnnotation {
}

/**
 * 对自定义注解的测试
 *
 * @author 二当家的白帽子 https://le-yi.blog.csdn.net/
 */
public class Test {
    @MyAnnotation
    private String name;
}
```

有的注解还带参数，二当家的也加上。

```java
package com.secondgod.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 测试注解
 *
 * @author 二当家的白帽子 https://le-yi.blog.csdn.net/
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@interface MyAnnotation {
    String name();
}

/**
 * 对自定义注解的测试
 *
 * @author 二当家的白帽子 https://le-yi.blog.csdn.net/
 */
public class Test {
    @MyAnnotation(name = "二当家的")
    private String name;
}
```

使用时候赋值必须写属性名，像@Target等一些注解赋值的时候就不用写属性名，那是因为属性名是“value”，就可以不用写。

```java
package com.secondgod.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 测试注解
 *
 * @author 二当家的白帽子 https://le-yi.blog.csdn.net/
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@interface MyAnnotation {
    String value();
}

/**
 * 对自定义注解的测试
 *
 * @author 二当家的白帽子 https://le-yi.blog.csdn.net/
 */
public class Test {
    @MyAnnotation("二当家的")
    private String name;
}
```

在注解里定义了属性后，使用的地方必须赋值。 不过可以在注解里定义默认值，使用时就可以不赋值了。

```java
package com.secondgod.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 测试注解
 *
 * @author 二当家的白帽子 https://le-yi.blog.csdn.net/
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@interface MyAnnotation {
    String value() default "二当家的";
}

/**
 * 对自定义注解的测试
 *
 * @author 二当家的白帽子 https://le-yi.blog.csdn.net/
 */
public class Test {
    @MyAnnotation
    private String name;
}
```

注解里的内容非常的少，显然他的功能并不在里面实现。所以注解就是个标记，本身并没有任何功能。那他的功能如何实现呢？

---

# 注解实战

我们用 [反射](https://juejin.cn/post/6995366359278288932 "https://juejin.cn/post/6995366359278288932") 和 [内省](https://juejin.cn/post/6995738368235552781 "https://juejin.cn/post/6995738368235552781") 配合注解来实现一个小工具，可以自动用环境变量初始化属性。

```java
package com.secondgod.annotation;

import java.beans.IntrospectionException;
import java.beans.PropertyDescriptor;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;

/**
 * 工具类
 *
 * @author 二当家的白帽子 https://le-yi.blog.csdn.net/
 */
public class MyUtils {
    /**
     * 私有构造，不可实例化
     */
    private MyUtils() {}

    /**
     * 用于对属性按照环境变量默认初始化
     */
    @Target(ElementType.FIELD)
    @Retention(RetentionPolicy.RUNTIME)
    @interface EnvironmentVariables {
        /**
         * 环境变量的key
         * @return
         */
        String value();
    }

    /**
     * 取得类实例，并用环境变量初始化
     *
     * @param clazz
     * @param <T>
     * @return
     * @throws InstantiationException
     * @throws IllegalAccessException
     * @throws java.beans.IntrospectionException
     * @throws InvocationTargetException
     */
    public static <T> T getInstance(Class<T> clazz) throws InstantiationException, IllegalAccessException, IntrospectionException, InvocationTargetException {
        T obj = clazz.newInstance();
        setEnvironmentVariables(obj);
        return obj;
    }

    /**
     * 为对象属性按照环境变量赋值
     *
     * @param o
     * @throws IllegalAccessException
     */
    public static void setEnvironmentVariables(Object o) throws IllegalAccessException, IntrospectionException, InvocationTargetException {
        for (Field f : o.getClass().getDeclaredFields()) {
            // 利用反射读取注解
            EnvironmentVariables environmentVariables = f.getDeclaredAnnotation(EnvironmentVariables.class);
            if (environmentVariables != null) {
                // 如果注解存在就读取环境变量
                String value = System.getProperty(environmentVariables.value());
                // 利用内省将值写入
                PropertyDescriptor pd = new PropertyDescriptor(f.getName(), o.getClass());
                pd.getWriteMethod().invoke(o, value);
            }
        }
    }
}
```

我们定义一个类用来表示虚拟机的信息，并且用注解标注属性字段。

```java
package com.secondgod.annotation;

import java.text.MessageFormat;

/**
 * 虚拟机信息
 *
 * @author 二当家的白帽子 https://le-yi.blog.csdn.net/
 */
public class VmInfo {
    @MyUtils.EnvironmentVariables(value = "java.vm.version")
    private String version;
    @MyUtils.EnvironmentVariables(value = "java.vm.vendor")
    private String vendor;
    @MyUtils.EnvironmentVariables(value = "java.vm.name")
    private String name;
    @MyUtils.EnvironmentVariables(value = "java.vm.specification.name")
    private String specName;
    @MyUtils.EnvironmentVariables(value = "java.vm.specification.vendor")
    private String specVendor;
    @MyUtils.EnvironmentVariables(value = "java.vm.specification.version")
    private String specVersion;
    @MyUtils.EnvironmentVariables(value = "java.vm.info")
    private String info;

    public String getVersion() {
        return version;
    }

    public void setVersion(String version) {
        this.version = version;
    }

    public String getVendor() {
        return vendor;
    }

    public void setVendor(String vendor) {
        this.vendor = vendor;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getSpecName() {
        return specName;
    }

    public void setSpecName(String specName) {
        this.specName = specName;
    }

    public String getSpecVendor() {
        return specVendor;
    }

    public void setSpecVendor(String specVendor) {
        this.specVendor = specVendor;
    }

    public String getSpecVersion() {
        return specVersion;
    }

    public void setSpecVersion(String specVersion) {
        this.specVersion = specVersion;
    }

    public String getInfo() {
        return info;
    }

    public void setInfo(String info) {
        this.info = info;
    }

    public String toString() {
        return MessageFormat.format("version={0}，vendor={1}，name={2}，specName={3}，specVendor={4}，specVersion={5}，info={6}", version, vendor, name, specName, specVendor, specVendor, info);
    }
}
```

好了，来测试一下我们的工具吧。

```java
package com.secondgod.annotation;

import java.beans.IntrospectionException;
import java.lang.reflect.InvocationTargetException;

/**
 * 对自定义注解的测试
 *
 * @author 二当家的白帽子 https://le-yi.blog.csdn.net/
 */
public class Test {

    public static void main(String[] args) throws IntrospectionException, InvocationTargetException, IllegalAccessException, InstantiationException {
        VmInfo info1 = new VmInfo();
        System.out.println("普通方式实例化后：" + info1);
        VmInfo info2 = MyUtils.getInstance(VmInfo.class);
        System.out.println("使用我们的小工具实例化后：" + info2);
    }
}
```

![](https://b3logfile.com/file/2021/08/solo-fetchupload-1028680520417006290-4ecc04b6.webp)

内容比较多，图没有截全，但是很明显可以看出我们的小工具符合我们的预期想法，很Nice。

---

# 尾声

在Spring，Hibernate，Mybatis等框架中都有自定义注解，常常用来作为除配置文件之外的另一种配置选择。他们各有利弊，应该根据实际情况选择。配置文件修改重启程序即可，而注解修改显然只能重新编译。但是二当家的觉得配置直接放在使用的地方非常方便，可读性也更好，所以除非是有发布后修改的需求，否则二当家都喜欢用注解。不过用注解就会比较分散，不如配置文件集中。哎，大家仁者见仁，智者见智吧。
