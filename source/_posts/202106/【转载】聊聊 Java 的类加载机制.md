title: 【转载】聊聊 Java 的类加载机制
date: 2021-06-18 22:58:50
toc: true
category: 
 - 转载
 - Java
tags: 
 - 转载
 - Java
 - 类加载机制
---

本文转载自：[聊聊 Java 的类加载机制二 | Nicksxs's Blog](https://nicksxs.me/2021/06/13/%E8%81%8A%E8%81%8A-Java-%E7%9A%84%E7%B1%BB%E5%8A%A0%E8%BD%BD%E6%9C%BA%E5%88%B6%E4%BA%8C/)

---

## 类加载器

类加载机制中说来说去其实也逃不开类加载器这个话题，我们就来说下类加载器这个话题，Java 在 jdk1.2 以后开始有了
Java 虚拟机设计团队有意把加载阶段中的“通过一个类的全限定名来获取描述该类的二进制字节流”这个动作放到 Java 虚拟机外部去实现，以便让应用程序自己去决定如何去获取所需的类。实现这个动作的代码被称为“类加载器”(Class Loader).


<!-- more -->


其实在 Java 中类加载器有一个很常用的作用，比如一个类的唯一性，其实是由加载它的类加载器和这个类一起来确定这个类在虚拟机的唯一性，这里也参考下周志明书里的例子

```java
public class ClassLoaderTest {

    public static void main(String[] args) throws Exception {
        ClassLoader myLoader = new ClassLoader() {
            @Override
            public Class<?> loadClass(String name) throws ClassNotFoundException {
                try {
                    String fileName = name.substring(name.lastIndexOf(".") + 1) + ".class";
                    InputStream is = getClass().getResourceAsStream(fileName);
                    if (is == null) {
                        return super.loadClass(name);
                    }
                    byte[] b = new byte[is.available()];
                    is.read(b);
                    return defineClass(name, b, 0, b.length);
                } catch (IOException e) {
                    throw new ClassNotFoundException(name);
                }
            }
        };
        Object object = myLoader.loadClass("com.nicksxs.demo.ClassLoaderTest").newInstance();
        System.out.println(object.getClass());
        System.out.println(object instanceof ClassLoaderTest);
    }
}
```

可以看下结果
[![](https://b3logfile.com/file/2021/06/solo-fetchupload-2468044438732444980-9efc0334.png)](https://b3logfile.com/file/2021/06/solo-fetchupload-2468044438732444980-9efc0334.png)
这里说明了当一个是由虚拟机的应用程序类加载器所加载的和另一个由自己写的自定义类加载器加载的，虽然是同一个类，但是 instanceof 的结果就是 false 的

## 双亲委派

自 JDK1.2 以来，Java 一直有些三层类加载器、双亲委派的类加载架构

### 启动类加载器

首先是启动类加载器，`Bootstrap Class Loader`，这个类加载器负责加载放在\lib目录，或者被-Xbootclasspath参数所指定的路径中存放的，而且是Java 虚拟机能够识别的（按照文件名识别，如 rt.jar、tools.jar，名字不符合的类库即使放在 lib 目录中，也不会被加载）类库加载到虚拟机的内存中，启动类加载器无法被 Java 程序直接引用，用户在编写自定义类加载器时，如果需要把家在请求为派给引导类加载器去处理，那直接使用 null 代替即可，可以看下 java.lang.ClassLoader.getClassLoader()方法的代码片段

```java
/**
     * Returns the class loader for the class.  Some implementations may use
     * null to represent the bootstrap class loader. This method will return
     * null in such implementations if this class was loaded by the bootstrap
     * class loader.
     *
     * <p> If a security manager is present, and the caller's class loader is
     * not null and the caller's class loader is not the same as or an ancestor of
     * the class loader for the class whose class loader is requested, then
     * this method calls the security manager's {@code checkPermission}
     * method with a {@code RuntimePermission("getClassLoader")}
     * permission to ensure it's ok to access the class loader for the class.
     *
     * <p>If this object
     * represents a primitive type or void, null is returned.
     *
     * @return  the class loader that loaded the class or interface
     *          represented by this object.
     * @throws SecurityException
     *    if a security manager exists and its
     *    {@code checkPermission} method denies
     *    access to the class loader for the class.
     * @see java.lang.ClassLoader
     * @see SecurityManager#checkPermission
     * @see java.lang.RuntimePermission
     */
    @CallerSensitive
    public ClassLoader getClassLoader() {
        ClassLoader cl = getClassLoader0();
        if (cl == null)
            return null;
        SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            ClassLoader.checkClassLoaderPermission(cl, Reflection.getCallerClass());
        }
        return cl;
    }
```

### 扩展类加载器

这个类加载器是在类sun.misc.Launcher.ExtClassLoader中以 Java 代码的形式实现的，它负责在家\lib\ext 目录中，或者被 java.ext.dirs系统变量中所指定的路径中的所有类库，它其实目的是为了实现 Java 系统类库的扩展机制

### 应用程序类加载器

这个类加载器是由sun.misc.Launcher.AppClassLoader实现，通过 java 代码，并且是 ClassLoader 类中的 getSystemClassLoader()方法的返回值，可以看一下代码

```java
/**
     * Returns the system class loader for delegation.  This is the default
     * delegation parent for new <tt>ClassLoader</tt> instances, and is
     * typically the class loader used to start the application.
     *
     * <p> This method is first invoked early in the runtime's startup
     * sequence, at which point it creates the system class loader and sets it
     * as the context class loader of the invoking <tt>Thread</tt>.
     *
     * <p> The default system class loader is an implementation-dependent
     * instance of this class.
     *
     * <p> If the system property "<tt>java.system.class.loader</tt>" is defined
     * when this method is first invoked then the value of that property is
     * taken to be the name of a class that will be returned as the system
     * class loader.  The class is loaded using the default system class loader
     * and must define a public constructor that takes a single parameter of
     * type <tt>ClassLoader</tt> which is used as the delegation parent.  An
     * instance is then created using this constructor with the default system
     * class loader as the parameter.  The resulting class loader is defined
     * to be the system class loader.
     *
     * <p> If a security manager is present, and the invoker's class loader is
     * not <tt>null</tt> and the invoker's class loader is not the same as or
     * an ancestor of the system class loader, then this method invokes the
     * security manager's {@link
     * SecurityManager#checkPermission(java.security.Permission)
     * <tt>checkPermission</tt>} method with a {@link
     * RuntimePermission#RuntimePermission(String)
     * <tt>RuntimePermission("getClassLoader")</tt>} permission to verify
     * access to the system class loader.  If not, a
     * <tt>SecurityException</tt> will be thrown.  </p>
     *
     * @return  The system <tt>ClassLoader</tt> for delegation, or
     *          <tt>null</tt> if none
     *
     * @throws  SecurityException
     *          If a security manager exists and its <tt>checkPermission</tt>
     *          method doesn't allow access to the system class loader.
     *
     * @throws  IllegalStateException
     *          If invoked recursively during the construction of the class
     *          loader specified by the "<tt>java.system.class.loader</tt>"
     *          property.
     *
     * @throws  Error
     *          If the system property "<tt>java.system.class.loader</tt>"
     *          is defined but the named class could not be loaded, the
     *          provider class does not define the required constructor, or an
     *          exception is thrown by that constructor when it is invoked. The
     *          underlying cause of the error can be retrieved via the
     *          {@link Throwable#getCause()} method.
     *
     * @revised  1.4
     */
    @CallerSensitive
    public static ClassLoader getSystemClassLoader() {
        initSystemClassLoader();
        if (scl == null) {
            return null;
        }
        SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            checkClassLoaderPermission(scl, Reflection.getCallerClass());
        }
        return scl;
    }
    private static synchronized void initSystemClassLoader() {
        if (!sclSet) {
            if (scl != null)
                throw new IllegalStateException("recursive invocation");
            // 主要的第一步是这
            sun.misc.Launcher l = sun.misc.Launcher.getLauncher();
            if (l != null) {
                Throwable oops = null;
                // 然后是这
                scl = l.getClassLoader();
                try {
                    scl = AccessController.doPrivileged(
                        new SystemClassLoaderAction(scl));
                } catch (PrivilegedActionException pae) {
                    oops = pae.getCause();
                    if (oops instanceof InvocationTargetException) {
                        oops = oops.getCause();
                    }
                }
                if (oops != null) {
                    if (oops instanceof Error) {
                        throw (Error) oops;
                    } else {
                        // wrap the exception
                        throw new Error(oops);
                    }
                }
            }
            sclSet = true;
        }
    }
// 接着跟到sun.misc.Launcher#getClassLoader
public ClassLoader getClassLoader() {
        return this.loader;
    }
// 然后看到这 sun.misc.Launcher#Launcher
public Launcher() {
        Launcher.ExtClassLoader var1;
        try {
            var1 = Launcher.ExtClassLoader.getExtClassLoader();
        } catch (IOException var10) {
            throw new InternalError("Could not create extension class loader", var10);
        }

        try {
            // 可以看到 就是 AppClassLoader
            this.loader = Launcher.AppClassLoader.getAppClassLoader(var1);
        } catch (IOException var9) {
            throw new InternalError("Could not create application class loader", var9);
        }

        Thread.currentThread().setContextClassLoader(this.loader);
        String var2 = System.getProperty("java.security.manager");
        if (var2 != null) {
            SecurityManager var3 = null;
            if (!"".equals(var2) && !"default".equals(var2)) {
                try {
                    var3 = (SecurityManager)this.loader.loadClass(var2).newInstance();
                } catch (IllegalAccessException var5) {
                } catch (InstantiationException var6) {
                } catch (ClassNotFoundException var7) {
                } catch (ClassCastException var8) {
                }
            } else {
                var3 = new SecurityManager();
            }

            if (var3 == null) {
                throw new InternalError("Could not create SecurityManager: " + var2);
            }

            System.setSecurityManager(var3);
        }

    }
```

它负责加载用户类路径(ClassPath)上所有的类库，我们可以直接在代码中使用这个类加载器，如果我们的代码中没有自定义的类在加载器，一般情况下这个就是程序中默认的类加载器

### 双亲委派模型

[![](https://b3logfile.com/file/2021/06/solo-fetchupload-6864044685722962310-8bcc4fec.png)](https://b3logfile.com/file/2021/06/solo-fetchupload-6864044685722962310-8bcc4fec.png)
双亲委派模型的工作过程是：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试家在这个类，而是把这个请求为派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的家在请求最终都应该传送到最顶层的启动类加载器中，只有当父类加载器反馈自己无法完成加载请求（它的搜索范围中没有找到所需要的类）时，子加载器才会尝试自己去完成加载。
使用双亲委派模型来组织类加载器之间的关系，一个显而易见的好处就是 Java 中的类随着它的类加载器一起举杯了一种带有优先级的层次关系。例如类 java.lang.Object，它存放在 rt.jar 之中，无论哪一个类加载器要家在这个类，最终都是委派给处于模型最顶层的启动类加载器进行加载，因此 Object 类在程序的各种类加载器环境中都能够保证是同一个类。反之，如果没有使用双薪委派模型，都由各个类加载器自行去加载的话，如果用户自己也编写了一个名为 java.lang.Object 的类，并放在程序的 ClassPath 中，那系统中就会出现多个不同的 Object 类，Java 类型体系中最基础的行为也就无从保证，应用程序将会变得一片混乱。
可以来看下双亲委派模型的代码实现

```java
/**
     * Loads the class with the specified <a href="#name">binary name</a>.  The
     * default implementation of this method searches for classes in the
     * following order:
     *
     * <ol>
     *
     *   <li><p> Invoke {@link #findLoadedClass(String)} to check if the class
     *   has already been loaded.  </p></li>
     *
     *   <li><p> Invoke the {@link #loadClass(String) <tt>loadClass</tt>} method
     *   on the parent class loader.  If the parent is <tt>null</tt> the class
     *   loader built-in to the virtual machine is used, instead.  </p></li>
     *
     *   <li><p> Invoke the {@link #findClass(String)} method to find the
     *   class.  </p></li>
     *
     * </ol>
     *
     * <p> If the class was found using the above steps, and the
     * <tt>resolve</tt> flag is true, this method will then invoke the {@link
     * #resolveClass(Class)} method on the resulting <tt>Class</tt> object.
     *
     * <p> Subclasses of <tt>ClassLoader</tt> are encouraged to override {@link
     * #findClass(String)}, rather than this method.  </p>
     *
     * <p> Unless overridden, this method synchronizes on the result of
     * {@link #getClassLoadingLock <tt>getClassLoadingLock</tt>} method
     * during the entire class loading process.
     *
     * @param  name
     *         The <a href="#name">binary name</a> of the class
     *
     * @param  resolve
     *         If <tt>true</tt> then resolve the class
     *
     * @return  The resulting <tt>Class</tt> object
     *
     * @throws  ClassNotFoundException
     *          If the class could not be found
     */
    protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        // 委托父类加载
                        c = parent.loadClass(name, false);
                    } else {
                        // 使用启动类加载器
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    // 调用自己的 findClass() 方法尝试进行加载
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

### 破坏双亲委派

关于破坏双亲委派模型，第一次是在 JDK1.2 之后引入了双亲委派模型之前，那么在那之前已经有了类加载器，所以java.lang.ClassLoader 中添加了一个 protected 方法 findClass()，并引导用户编写的类加载逻辑时尽可能去重写这个方法，而不是在 loadClass()中编写代码。这个跟上面的逻辑其实类似，当父类加载失败，会调用 findClass()来完成加载；第二次是因为这个模型本身还有一些不足之处，比如 SPI 这种，所以有设计了线程下上下文类加载器(Thread Context ClassLoader)。这个类加载器可以通过 java.lang.Thread 类的 java.lang.Thread#setContextClassLoader() 进行设置，然后第三种是为了追求程序动态性，这里有涉及到了 osgi 等概念，就不展开了

