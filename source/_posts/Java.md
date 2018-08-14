---
title: 注解（Annotation） by Thinking in Java
date: 2018-04-27 16:25:17
tags: 注解（Annotation）
---
###注解（元数据）为我们在代码中添加信息提供了一种形式化的方法，使我们可以在某个时刻非常方便的使用这些数据（受到了C#的启发 C#覆盖一个方法必须使用@Override，但是java不是必选的）

 java SE5重要的语言的变化
 * 注解使得我们的能够以将有编译器来测试和验证的格式，存储有关程序的额外的信息
 * 注解可以用来生成描述符文件，甚至或是新的类定义，并有助于减轻编写样板代码的的负担
 * 使用注解，我们可以将这些元数据保存在Java代码中，并利用 annotion API 为自己的注解构造处理工具
 * 更加干净易读的代码以及编译器类型的检查
 * 虽然Java SE5提供一些元数据，但是一般来说，主要还是需要我们自己定义新的注解，并且按照自己的方法使用它们
<!--  more  --> 
###定义注解
```
/**
 * author： Created by shiming on 2018/4/27 11:41
 * mailbox：lamshiming@sina.com
 * 　　@Target(ElementType.TYPE)   //接口、类、枚举、注解
 * 　　@Target(ElementType.FIELD) //字段、枚举的常量
 * 　　@Target(ElementType.METHOD) //方法
 * 　　@Target(ElementType.PARAMETER) //方法参数。参数声明
 * 　　@Target(ElementType.CONSTRUCTOR)  //构造函数，构造器的声明
 * 　　@Target(ElementType.LOCAL_VARIABLE)//局部变量
 * 　　@Target(ElementType.ANNOTATION_TYPE)//注解
 * 　　@Target(ElementType.PACKAGE) ///包
 */
@Target(ElementType.METHOD)//定义你的注解将应用在什么地方（一个方法，一个域）
@Retention(RetentionPolicy.RUNTIME)//定义注解在哪个级别可以使用，在源码中（SOURCE）、类文件中（CALSS），或者运行时（RUNTIME）
//没有元素的注解，称为标记注解（marker annotation），如果某个方法或者实现某个用例的需求，加上次方法，项目经理可以很好地观察到进度
//如果需要修改系统的业务逻辑，则维护改项目的开发人员也可以很容易的找到相对于的实例
public @interface Test {
}
```
关于RetentionPolicy 类
```
/**
 * author： Created by shiming on 2018/4/27 15:09
 * mailbox：lamshiming@sina.com
 */
public enum  RetentionPolicy {
        /**
         * Annotations are to be discarded by the compiler.
         * 注解将被编译器丢弃
         */
        SOURCE,

        /**
         * Annotations are to be recorded in the class file by the compiler
         * but need not be retained by the VM at run time.  This is the default
         * behavior.
         * 注解在class文件中可用，但会被VM丢弃
         */
        CLASS,

        /**
         * Annotations are to be recorded in the class file by the compiler and
         * retained by the VM at run time, so they may be read reflectively.
         * VM将在运行期也保留注解，因此可以通过反射机制读取注解的信息
         * @see java.lang.reflect.AnnotatedElement
         */
        RUNTIME

}
```
###目前可以理解为注释
```

public class Testale {
    public void execute(){
        System.out.println("shiming execute");
    }
    @Test//就好比一个注释
    void testExecute(){
        execute();
    }
}
```
###Demo----->
```
/**
 * author： Created by shiming on 2018/4/27 14:25
 * mailbox：lamshiming@sina.com
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface UseCase {
    //由于编译器会对id进行类型检查，因此将用例文档的追踪数据库与源代码想关联是可靠的
    public int id();//默认值的限定，首先，元素不能有不确定的值，也就是说，元素必须要么具有默认值，要么具有使用元素提供的元素的值
    //如果注解某个方法没有给出值的话，则该注解的处理器就会使用此元素的默认值
    public String description() default "shiming no descriprition 我是默认的des";
}
```

```
/**
 * author： Created by shiming on 2018/4/27 14:35
 * mailbox：lamshiming@sina.com
 * 注解的元素在使用时候为名-值对的形式，并需要置于@USECase申明之后的括号内，在方法注解中可以给出description的值，也可以不给
 * 因此，在UseCase的注解处理器分析处理这个类时会使用该元素的默认值
 */
public class PasswordUtils {
    @UseCase(id = 1,description = "start need d  我是id==1")
    public boolean validatePassword(String s){
        return s.startsWith("d");
    }

    @UseCase(id = 2)
    public String validatePasswordReverse(String s){
        return new StringBuilder(s).reverse().toString();
    }

    @UseCase(id = 3,description = "start need 我是id==3")
    public boolean validatePasswordStirng(String s){
        return s.startsWith("shiming");
    }

}
```
 ####如果没有用来读取注解的工具，那么注解也不会比注释更加有用
```
/**
 * author： Created by shiming on 2018/4/27 15:16
 * mailbox：lamshiming@sina.com
 *
 * 如果没有用来读取注解的工具，那么注解也不会比注释更加有用
 */
public class UseCaseTracker {
    @SuppressLint("UseValueOf")
    public static void trackUseCases(List<Integer> useCases, Class<?> cl){
        /**
         * getDeclaredMethods返回 Method 对象的一个数组，
         * 这些对象反映此 Class 对象表示的类或接口声明的所有方法，
         * 包括公共、保护、默认（包）访问和私有方法，但不包括继承的方法
         */
        for (Method m:cl.getDeclaredMethods()){
            UseCase annotation = m.getAnnotation(UseCase.class);
            System.out.println("执行了第几次 "+annotation+"----->m.getName"+m.getName());
            if (annotation!=null){
                System.out.println("shiming  "+annotation.id()+"---->"+annotation.description());
                //注意这是元素还有角标的问题
                useCases.remove(new Integer(annotation.id()));
            }
            for (int i:useCases){
                System.out.println("shiming warning Missing usecase"+i);
            }
        }
    }
}
```
客户端调用
```
      ArrayList<Integer> integers = new ArrayList<>();
        integers.add(1);
        integers.add(2);
        integers.add(3);
        integers.add(4);
        UseCaseTracker.trackUseCases(integers,PasswordUtils.class);
```
执行结果如下
```
04-27 15:40:19.830 4425-4425/annotation.shimihg.com I/System.out: 执行了第几次 null----->m.getNameaccess$super
04-27 15:40:19.830 4425-4425/annotation.shimihg.com I/System.out: shiming warning Missing usecase1
04-27 15:40:19.830 4425-4425/annotation.shimihg.com I/System.out: shiming warning Missing usecase2
04-27 15:40:19.830 4425-4425/annotation.shimihg.com I/System.out: shiming warning Missing usecase3
04-27 15:40:19.830 4425-4425/annotation.shimihg.com I/System.out: shiming warning Missing usecase4
04-27 15:40:19.830 4425-4425/annotation.shimihg.com I/System.out: 执行了第几次 @annotation.shimihg.demo.UseCase(description=start need d  我是id==1, id=1)----->m.getNamevalidatePassword
04-27 15:40:19.830 4425-4425/annotation.shimihg.com I/System.out: shiming  1---->start need d  我是id==1
04-27 15:40:19.830 4425-4425/annotation.shimihg.com I/System.out: shiming warning Missing usecase2
04-27 15:40:19.830 4425-4425/annotation.shimihg.com I/System.out: shiming warning Missing usecase3
04-27 15:40:19.830 4425-4425/annotation.shimihg.com I/System.out: shiming warning Missing usecase4
04-27 15:40:19.830 4425-4425/annotation.shimihg.com I/System.out: 执行了第几次 @annotation.shimihg.demo.UseCase(description=shiming no descriprition 我是默认的des, id=2)----->m.getNamevalidatePasswordReverse
04-27 15:40:19.830 4425-4425/annotation.shimihg.com I/System.out: shiming  2---->shiming no descriprition 我是默认的des
04-27 15:40:19.830 4425-4425/annotation.shimihg.com I/System.out: shiming warning Missing usecase3
04-27 15:40:19.830 4425-4425/annotation.shimihg.com I/System.out: shiming warning Missing usecase4
04-27 15:40:19.830 4425-4425/annotation.shimihg.com I/System.out: 执行了第几次 @annotation.shimihg.demo.UseCase(description=start need 我是id==3, id=3)----->m.getNamevalidatePasswordStirng
04-27 15:40:19.830 4425-4425/annotation.shimihg.com I/System.out: shiming  3---->start need 我是id==3
04-27 15:40:19.830 4425-4425/annotation.shimihg.com I/System.out: shiming warning Missing usecase4
```

####未完待续，☺☺☺！git：https://github.com/Shimingli/AnnotationDemo
