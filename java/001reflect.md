# Java Reflect机制
## 什么是Reflect
   Java反射机制是在运行状态中，对于任意一个类都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

   Java反射（放射）机制：“程序运行时，允许改变程序结构或变量类型，这种语言称为动态语言”。从这个观点看，Perl，Python，Ruby是动态语言，C++，Java，C#不是动态语言。但是Java有着一个非常突出的动态相关机制：Reflection，用在Java身上指的是我们可以于运行时加载、探知、使用编译期间完全未知的classes。换句话说，Java程序可以加载一个运行时才得知名称的class，获悉其完整构造（但不包括methods定义），并生成其对象实体、或对其fields设值、或唤起其methods。
##Java反射的功能
   Java反射机制主要提供了以下功能：在运行时判断任意一个对象所属的类；在运行时构造任意一个类的对象；在运行时判断任意一个类所具有的成员变量和方法；在运行时调用任意一个对象的方法；生成动态代理。

主要作用有三：

* 运行时取得类的方法和字段的相关信息。
* 创建某个类的新实例(.newInstance())
* 取得字段引用直接获取和设置对象字段，无论访问修饰符是什么。
用处如下：

* 观察或操作应用程序的运行时行为。
* 调试或测试程序，因为可以直接访问方法、构造函数和成员字段。
* 通过名字调用不知道的方法并使用该信息来创建对象和调用方法。

##Java反射常用API
   如果你使用Java，那么你应该知道Java中有一个Class类。Class类本身表示Java对象的类型，我们可以通过一个Object（子）对象的getClass方法取得一个对象的类型，此函数返回的就是一个Class类。当然，获得Class对象的方法有许多，但是没有一种方法是通过Class的构造函数来生成Class对象的。也许你从来没有使用过Class类，也许你曾以为这是一个没什么用处的东西。不管你以前怎么认为，Class类是整个Java反射机制的源头。一切关于Java反射的故事，都从Class类开始。因此，要想使用Java反射，我们首先得到Class类的对象。下表列出了几种得到Class类的方法，以供大家参考。

获取Class对象的方法：

* 调用getClass()方法
	
	```String s = new String("test string");
	Class class = s.getClass();```
* 调用Class的getSuperClass()
	
	```String s = new String("test string");
	Class class = s.getClass();
	Class superClass = class.getSuperClass();```
* 调用Class.forName()
	
	```Class class = Class.forName("java.lang.String");```
* 类型的 .class 语法

	```Class class = String.class;```

* 包装类的TYPE语法

	```Class class = Boolean.TYPE;```

获取到Class对象后调取相应API获取，类相关的信息对象。如：：构造函数、成员函数、成员变量。详细参考
代码示例：

```
package com.hainiubl.demo;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.lang.reflect.Modifier;

public class ReflectDemo {

    public static void main(String[] args) throws ClassNotFoundException, InstantiationException,
            IllegalAccessException, NoSuchMethodException, InvocationTargetException {
        Class<String> cl = String.class;

        // 获取基本信息
        /**
         如果是数组类型 className返回 一个或多个 ［  加编码类型，［个数表示数组深度
            Element Type        Encoding
            boolean     Z
            byte        B
            char        C
            class or interface      Lclassname;
            double      D
            float       F
            int     I
            long        J
            short       S

         String.class.getName()
             returns "java.lang.String"
         byte.class.getName()
             returns "byte"
         (new Object[3]).getClass().getName()
             returns "[Ljava.lang.Object;"
         (new int[3][4][5][6][7][8][9]).getClass().getName()
             returns "[[[[[[[I"

         */
        System.out.println("Package Name:" + cl.getPackage().getName() + ",Full Class Name:" + cl.getName()
                + ",ShortName:" + cl.getSimpleName() + ",Super Class:" + cl.getSuperclass().getName());

        // 匿名类会有一下特殊情况
        Object o = new Object() {
            public void doSomething() {
                System.out.println("do something");
            }
        };

        Class cl2 = o.getClass();
        System.out.println("Package Name:" + cl2.getPackage().getName() + ",Full Class Name:" + cl2.getName()
                + ",ShortName:" + cl2.getSimpleName());
        System.out.println("===============================================");  

        // 实例化
        // newInstance 不能加参数，一定要有无参构造函数
        String s1 = cl.newInstance();
        System.out.println("s1:" + s1);

        // 想传参数可以获取构造函数对象实例化,多个构造函数通过类型签名确定
        Constructor<String> constructor = cl.getConstructor(String.class);
        String s2 = constructor.newInstance("test parameter");
        System.out.println("s2:" + s2);

        //或返回构造函数数组判断函数类型
        Constructor<?>[] constructors = cl.getConstructors();
        for (Constructor<?> c : constructors){
            Class [] types = c.getParameterTypes();
            String s = "String(";
            for(Class ct : types){
                s = s + ct.getName() + ",";
            }
            System.out.println((s.charAt(s.length() -1) == ',' ? s.substring(0,s.length() -1):s) + ")");

        }
        System.out.println("===============================================");  

        Field[] fields = cl.getDeclaredFields();  
        for (int i = 0; i < fields.length; i++) {  
            System.out.println("类中的成员: " + fields[i]);  
        }  
        System.out.println("===============================================");  

        //取得类方法  
        Method[] methods = cl.getDeclaredMethods();  
        for (int i = 0; i < methods.length; i++) {  
            System.out.println("函数名：" + methods[i].getName());  
            System.out.println("函数返回类型：" + methods[i].getReturnType());  
            System.out.println("函数访问修饰符：" + Modifier.toString(methods[i].getModifiers()));  
            System.out.println("函数代码写法： " + methods[i]);  
        }  

        System.out.println("===============================================");           
        //调用方法

        Method m = cl.getMethod("length", null);
        m.setAccessible(true);//关闭安全策略检查，提高效率
        System.out.println(m.invoke(s2, null));

        m = cl.getMethod("charAt", Integer.TYPE);
        System.out.println(m.invoke(s2,0));
    }

}

```