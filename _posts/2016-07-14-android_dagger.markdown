---
layout: post
title:  "Android依赖注入之dagger"
date:   2016-07-14 13:49:45 +0800
categories: Java
---

## 关于依赖注入（Dependency Injection，简称DI）
类和类之间要建立联系，比如A类需要B类的实例对象，我们就说A类依赖B类，反过来，就说B类注入到A类中，所以“依赖注入”表示两层含义，依赖和注入。

## View层面的DI
看了两个DI的库，AndroidAnnotation和Butter knife，个人感觉后者更好用一些，不会生成多余的类文件，都是view层面的DI，使用比较简单

## 自定义的DI
当然就是Dagger啦，跟Butter knife一样都是JakeWharton大神的作品。

Dagger使用java的注解，在预编译阶段就初始化了对象。Dagger的使用结构如下：
![能显示了吧](https://raw.githubusercontent.com/boomstack/boomstack.github.io/master/assets/all/headshot.jpg)
在下面的例子中，被注入者就是MainActivity，容器就是以“Module”结尾的类，而注入者就是实际的Teacher、Student类了。
先看被注入者Teacher：
```
public class Teacher {
    @Inject
    public Teacher(){

    }
    public void print(){
        System.out.println("hola: i am the teacher!");
    }
}
```
跟普通的类很像，只是多了一个@Inject注解，它表示这个类将来是要注入到其他类中去的，在构造函数使用。
下一个注入类，Student：
```
public class Student {
    String str;

    @Inject
    public Student() {

    }

    public Student(String name) {
        str = name;
    }

    public void print() {
        System.out.println("hola: string from Student default constructor");
    }

    public void printName() {
        System.out.println("hola: string from Student(String name) constructor: " + str);
    }

}
```
也是一样，不过有多个构造函数，怎么注入使用不同构造函数的对象待会再说。
注意，这里只能在一个构造函数中使用@Inject，否则报错：
```
Too many injectable constructors on com.ethan.daggertestsimple.Student
```
接着看第一个容器类TeacherModule：
```
@Module(library = true)
public class TeacherModule {
    @Provides Teacher provideTeacher(){
        return new Teacher();
    }
}
```
第二个容器类：
```
@Module(injects = MainActivity.class,library = true,includes = {TeacherModule.class})
public class StudentModule {
    @Provides
    public Student provideStudent(){
        return new Student();
    }
    @Provides @Named("Jake")
    public Student provideJakeStudent(){
        return new Student("Jake");
    }
    @Provides @Named("Ethan")
    public Student provideEthanStudent(){
        return new Student("Ethan");
    }
}

```
@Module就声明了这个类是个容器，这个接口中有很多方法，常见的（我用过的。。）有这几个：
```
//这个模块中的类被注入到哪里，这里是类数组，所以可以被注入到多个类中
Class<?>[] injects() default { };
//当前模块可以包含哪几个模块，也是类数组，模块之间可以嵌套，最终只能有一个模块，最终都要被嵌套到最高层的Module中去
Class<?>[] includes() default { };
//这个模块是不是完整的
boolean complete() default true;
//这个模块是不是作为库使用，如果不是，那么所有的方法必须得被调用，不然报错，所以声明为true为好
boolean library() default false;
```
@Provides是提供实例的方法注解，被它注解的方法回实例化被注入者（Student、Teacher）
@Named是javax.inject包中的接口，在这里可以用来区分调用哪个构造方法，如果这里@Provides只用一个方法，完全没有必要声明@Named  

下面是被注入者MainActivity：
```
public class MainActivity extends AppCompatActivity {
    //在使用被注入对象时加入Inject,程序在DI过程中自动初始化
    @Inject
    Student stu;
    @Inject
    @Named("Jake")
    Student stuJake;
    @Inject
    @Named("Ethan")
    Student stuEthan;
    @Inject
    Teacher teacher;

    @Inject
    Provider<Student> stuList;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ObjectGraph.create(StudentModule.class).inject(this);

        stu.print();
        stuJake.printName();
        stuEthan.printName();
        mutiStudent();
        teacher.print();
    }

    public void mutiStudent() {
        System.out.println("hola: =======student list========");
        for (int i = 0; i < 6; i++) {
            stuList.get().print();
        }
        System.out.println("hola: =======student list complete========");
    }
}
```
先不说别的，看着代码是不是很简洁，没有明确的实例化对象，但是在预编译阶段已经实例化了！

这里使用了Provider来提供对象数组，它同样来自javax.inject

下面看最关键的一行代码：
```
ObjectGraph.create(StudentModule.class).inject(this);
```
ObjectGraph是连接注入者与被注入者的类：
```
A graph of objects linked by their dependencies.
```
它的create方法会返回一个graph实例：
```
/**
   * Returns a new dependency graph using the {@literal @}{@link
   * Module}-annotated modules.
   *
   * <p>This <strong>does not</strong> inject any members. Most applications
   * should call {@link #injectStatics} to inject static members and {@link
   * #inject} or get {@link #get(Class)} to inject instance members when this
   * method has returned.
   *
   * <p>This <strong>does not</strong> validate the graph. Rely on build time
   * tools for graph validation, or call {@link #validate} to find problems in
   * the graph at runtime.
   */
  public static ObjectGraph create(Object... modules) {
    return DaggerObjectGraph.makeGraph(null, new FailoverLoader(), modules);
  }
```
然后通过inject方法将依赖注入到当前类中，被注入者和注入这就此连接起来了，后续就可以和平时一样使用已经初始化好的对象了！