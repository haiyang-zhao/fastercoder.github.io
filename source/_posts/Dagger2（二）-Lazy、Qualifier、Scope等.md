title: Dagger2（二） Lazy、Qualifier、Scope等
date: 2019-02-10 15:40:15
categories: Dagger2
tags: Dagger2
---



#### Lazy 延迟加载

使用 @Inject Lazy<xxx>来提供延迟加载,Lazy和Provider功类似

```java
public interface Lazy<T> {
  /**
   * Return the underlying value, computing the value if necessary. All calls to
   * the same {@code Lazy} instance will return the same result.
   */
  T get();
}
```




```
 public class MainActivity extends AppCompatActivity {
    @Inject
    Lazy<PrintService> printService;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        DaggerPrintComponent.builder()
                .build().inject(this);
    }

    public void print(View view) {
        printService.get().startWork();
    }

```

dagger生成的代码如下：
```java
public final class MainActivity_MembersInjector implements MembersInjector<MainActivity> {
  private final Provider<PrintService> printServiceProvider;

  public MainActivity_MembersInjector(Provider<PrintService> printServiceProvider) {
    this.printServiceProvider = printServiceProvider;
  }

  public static MembersInjector<MainActivity> create(Provider<PrintService> printServiceProvider) {
    return new MainActivity_MembersInjector(printServiceProvider);
  }

  @Override
  public void injectMembers(MainActivity instance) {
    injectPrintService(instance, DoubleCheck.lazy(printServiceProvider));
  }

  public static void injectPrintService(MainActivity instance, Lazy<PrintService> printService) {
    instance.printService = printService;
  }
}
```
关键代码
```java
@Override
  public void injectMembers(MainActivity instance) {
    injectPrintService(instance, DoubleCheck.lazy(printServiceProvider));
  }

```

DoubleCheck.lazy()是将Provider转化为DoubleCheck
```java
public static <P extends Provider<T>, T> Lazy<T> lazy(P provider) {
    if (provider instanceof Lazy) {
      @SuppressWarnings("unchecked")
      final Lazy<T> lazy = (Lazy<T>) provider;
      // Avoids memoizing a value that is already memoized.
      // NOTE: There is a pathological case where Provider<P> may implement Lazy<L>, but P and L
      // are different types using covariant return on get(). Right now this is used with
      // DoubleCheck<T> exclusively, which is implemented such that P and L are always
      // the same, so it will be fine for that case.
      return lazy;
    }
    return new DoubleCheck<T>(checkNotNull(provider));
  }
```
在调用Lazy.get()时，就是调用DoubleCheck.get(),代码如下，返回的是同一个对象

```java
@SuppressWarnings("unchecked") // cast only happens when result comes from the provider
  @Override
  public T get() {
    Object result = instance;
    if (result == UNINITIALIZED) {
      synchronized (this) {
        result = instance;
        if (result == UNINITIALIZED) {
          result = provider.get();
          instance = reentrantCheck(instance, result);
          /* Null out the reference to the provider. We are never going to need it again, so we
           * can make it eligible for GC. */
          provider = null;
        }
      }
    }
    return (T) result;
  }
```
* 使用 Lazy延迟加载时，get()方法返回同一个对象

#### Provide 延迟加载

和Lazy用法一致，但每次调用get()方法，返回不同的对象。


#### Qualifier 限定符

在Module中要提供接口的多个实现类，下面的写法编译器会报错
 
 ```java
 @Module(includes = {AppModule.class})
public class PrinterModule {

    @Provides
    public Printer providePrinter() {
        return new PdfPrinter();
    }


    @Provides
    public Printer provideSamplePrinter() {
        return new SamplePrinter();
    }

}

error: [Dagger/DuplicateBindings] com.zhy.dagger2.printer.Printer is bound multiple times:

 ```
 使用Qualifier可以解决此类问题。创建注解Sample,Pdf他们都被Qualifier标注
 ```java
 @Qualifier
@Documented
@Retention(RetentionPolicy.RUNTIME)
public @interface Pdf {
}

@Qualifier
@Documented
@Retention(RetentionPolicy.RUNTIME)
public @interface Sample {
}

 ```
 添加到对应的Provider中
 ```
 @Module(includes = {AppModule.class})
public class PrinterModule {

    @Provides
    @Pdf
    public Printer providePrinter() {
        return new PdfPrinter();
    }


    @Provides
    @Sample
    public Printer provideSamplePrinter() {
        return new SamplePrinter();
    }

}
```

依赖注入时带上对应的Annotation，

```
@Inject
public PrintService(@Sample Printer printer) {
    this.printer = printer;
    System.out.println("Loading PrintService....");
}
```

#### @Named

JSR330中内置的Qualifier，使用起来很方便

```java
@Module(includes = {AppModule.class})
public class PrinterModule {

    @Provides
//  @Pdf
    @Named("pdf")
    public Printer providePrinter() {
        return new PdfPrinter();
    }


    @Provides
//  @Sample
    @Named("sample")
    public Printer provideSamplePrinter() {
        return new SamplePrinter();
    }

}

PrintService//

@Inject
public PrintService(@Named("pdf") Printer printer) {
     this.printer = printer;
     System.out.println("Loading PrintService....");
}
```


#### @Scope  作用域
> Scope 是用来确定注入的实例的生命周期的，如果没有使用 Scope 注解，Component 每次调用 Module 中的 provide 方法或 Inject 构造函数生成的工厂时都会创建一个新的实例，而使用 Scope 后可以复用之前的依赖实例
 
dagger2中提供了@Singleton表示单例，它也是被@Scope所标注

```java
@Scope
@Documented
@Retention(RUNTIME)
public @interface Singleton {}
```

现在，我们有两个Activity,MainActivity和SecondActivity,两者都依赖PrintService

```java
public class MainActivity extends AppCompatActivity {
    @Inject
    PrintService printService;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        DaggerMainComponent.builder()
                .build().inject(this);
    }

    public void print(View view) {
        printService.startWork();
    }

    public void testScope(View view) {
        startActivity(new Intent(this, SecondActivity.class));
    }
}

执行print()时，PrintService对象如下
com.zhy.dagger2 I/System.out: com.zhy.dagger2.printer.PrintService@38bb568

```

SecondActivity
```java
public class SecondActivity extends AppCompatActivity {

    @Inject
    PrintService printService;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_second);
        DaggerSecondComponent.builder()
                .build()
                .inject(this);
    }

    public void testScope(View view) {
        System.out.println(printService.toString());
    }
}
执行print()时，PrintService对象如下
I/System.out: com.zhy.dagger2.printer.PrintService@1a9a612
```

如何让PrintService是单例呢？这时候就用到@Singleton注解了

```java

@Singleton
public class PrintService {
//.....
}

然后在MainComponent和SecondComponent上标记@Singleton（必须也是@Singleton）

@Singleton
@Component(modules = PrinterModule.class)
public interface MainComponent {
    void inject(MainActivity activity);

}
Singleton
@Component(modules = PrinterModule.class)
public interface SecondComponent {

    void inject(SecondActivity secondActivity);
}

```
然而，PrintService依然还是不同的对象

MainActivity:</br>
I/System.out: com.zhy.dagger2.printer.PrintService@38bb568

SecondActivity:</br>
I/System.out: com.zhy.dagger2.printer.PrintService@a3fd874

***为什么呢*？**

> 这是因为Component 间接持有依赖实例的引用，把实例的作用域Component 绑定。也即是说同一个Component标注的@Singleton对象才是单例。

有两种方法：
###### 1.使用相同的Component
MainActivity和SecondActivity使用同一个Component(AppComponent)

AppComponent
```java
@Singleton
@Component(modules = {AppModule.class, PrinterModule.class})
public interface AppComponent {

    void inject(App app);

    void inject(MainActivity mainActivity);

    void inject(SecondActivity secondActivity);


}

```
在Application中实例化AppComponent

```java
public class App extends Application {


    private static App app;
    private AppComponent appComponent;

    @Override
    public void onCreate() {
        super.onCreate();
        app = this;
        appComponent = DaggerAppComponent
                .builder().build();
        appComponent.inject(this);
    }

    public AppComponent getAppComponent() {
        return appComponent;
    }


    public static AppComponent component() {
        return app.getAppComponent();
    }

}

```
使用AppComponent
```java
public class MainActivity extends AppCompatActivity {
    @Inject
    PrintService printService;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        App.component()
                .inject(this);

    }
    //...
}

public class SecondActivity extends AppCompatActivity {
    @Inject
    PrintService printService;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_second);
        App.component()
                .inject(this);
    }
    //...
}
```
Logcat中输出的PrintService是同一个对象
    
    2019-02-05 23:00:30.640 26381-26381/com.zhy.dagger2 I/System.out: com.zhy.dagger2.printer.PrintService@f90971b
    2019-02-05 23:00:30.640 26381-26381/com.zhy.dagger2 I/System.out: Pdf Printer work.....
    2019-02-05 23:00:33.859 26381-26381/com.zhy.dagger2 I/System.out: com.zhy.dagger2.printer.PrintService@f90971b

分析源码
```
public final class DaggerAppComponent implements AppComponent {
    //...
     private DaggerAppComponent(Builder builder) {
     initialize(builder);
     }
    
    private void initialize(final Builder builder) {
    this.providePrinterProvider = PrinterModule_ProvidePrinterFactory.create(builder.printerModule);
    this.printServiceProvider =
        DoubleCheck.provider(PrintService_Factory.create(providePrinterProvider));
  }
  //...
}
```

 this.printServiceProvider是DoubleCheck,而DoubleCheck每次get()的是同一个对象。
 
 ###### 2.依赖相同的Component
 
创建自定义Scope
```java
@Scope
@Documented
@Retention(RUNTIME)
public @interface ActivityScope {
}
```
AppComponet如下:
```java
@Singleton
@Component(modules = {AppModule.class,
        PrinterModule.class})
public interface AppComponent {

    void inject(App app);
    
    PrintService printService();
}

```
可以看到,在AppComponent中printService()返回PrintService。在一个Component中，方法的参数和返回值Dagger都会给你实例化。

Dagger生成的代码如下：
```java
public final class DaggerAppComponent implements AppComponent {
  private PrinterModule_ProvidePrinterFactory providePrinterProvider;

  private Provider<PrintService> printServiceProvider;

  private DaggerAppComponent(Builder builder) {

    initialize(builder);
  }

  public static Builder builder() {
    return new Builder();
  }

  public static AppComponent create() {
    return new Builder().build();
  }

  @SuppressWarnings("unchecked")
  private void initialize(final Builder builder) {
    this.providePrinterProvider = PrinterModule_ProvidePrinterFactory.create(builder.printerModule);
    this.printServiceProvider =
        DoubleCheck.provider(PrintService_Factory.create(providePrinterProvider));
  }

  @Override
  public void inject(App app) {}

  @Override
  public PrintService printService() {
    return printServiceProvider.get();
  }

  public static final class Builder {
    private PrinterModule printerModule;

    private Builder() {}

    public AppComponent build() {
      if (printerModule == null) {
        this.printerModule = new PrinterModule();
      }
      return new DaggerAppComponent(this);
    }

    /**
     * @deprecated This module is declared, but an instance is not used in the component. This
     *     method is a no-op. For more, see https://google.github.io/dagger/unused-modules.
     */
    @Deprecated
    public Builder appModule(AppModule appModule) {
      Preconditions.checkNotNull(appModule);
      return this;
    }

    public Builder printerModule(PrinterModule printerModule) {
      this.printerModule = Preconditions.checkNotNull(printerModule);
      return this;
    }
  }
}

```

MainComponent和SecondComponent分别依赖AppComponent并且被ActivityScope标注
```
@ActivityScope
@Component(dependencies = AppComponent.class)
public interface MainComponent {
    void inject(MainActivity activity);

}
@ActivityScope
@Component(dependencies = AppComponent.class)
public interface SecondComponent {
    void inject(SecondActivity secondActivity);
}

```
在MainAcitivy和SecondActivity注入依赖
```java
MainActivity的onCreate()中

DaggerMainComponent.builder()
                .appComponent(App.component())
                .build()
                .inject(this);
```

Dagger代码为:
```
public final class DaggerMainComponent implements MainComponent {
  private AppComponent appComponent;

  private DaggerMainComponent(Builder builder) {
    this.appComponent = builder.appComponent;
  }

  public static Builder builder() {
    return new Builder();
  }

  @Override
  public void inject(MainActivity activity) {
    injectMainActivity(activity);
  }

  private MainActivity injectMainActivity(MainActivity instance) {
    MainActivity_MembersInjector.injectPrintService(
        instance,
        Preconditions.checkNotNull(
            appComponent.printService(),
            "Cannot return null from a non-@Nullable component method"));
    return instance;
  }

  public static final class Builder {
    private AppComponent appComponent;

    private Builder() {}

    public MainComponent build() {
      Preconditions.checkBuilderRequirement(appComponent, AppComponent.class);
      return new DaggerMainComponent(this);
    }

    public Builder appComponent(AppComponent appComponent) {
      this.appComponent = Preconditions.checkNotNull(appComponent);
      return this;
    }
  }
}

```

我们看到injectMainActivity()方法中
MainActivity_MembersInjector.injectPrintService()调用了appComponent.printService()，由此得到单例的PrintService对象。

对比这两种方法，第二种是推荐的。

##### 使用@Scope的一些经验

* @Component关联的@Module中的任何一个@Provides有@scope，则该整个@Component要加上这个scope。
* @Component的dependencies与@Component自身的scope不能相同，即组件之间的scope不能相同。
* @Singleton的组件不能依赖其他scope的组件，但是其他scope的组件可以依赖@Singleton组件。
* 没有scope组件不能依赖有scope的组件。
* 一个component不能同时有多个scope(Subcomponent除外)


#### Binding Instances

通过前面作用域的讲解，可以清楚 Component 可以间接持有 Module 或 Inject 目标类构造函数提供的依赖实例，除了这两种方式，Component 还可以在创建 Component 的时候绑定依赖实例，用以注入。这就是@BindsInstance注解的作用，只能在 Component.Builder 中使用。

```
@Component
public interface ThirdComponent {

    Login login();

    @Component.Builder
    interface Builder {
        @BindsInstance
        Builder login(Login login);

        ThirdComponent build();
    }

    void inject(ThirdActivity thirdActivity);
}

//Component就可以拥有Login实例了
public class ThirdActivity extends AppCompatActivity {

    private ThirdComponent component;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_third);
        component = DaggerThirdComponent.builder()
                .login(new Login("zhangsan", "lisi"))
                .build();
        component
                .inject(this);


    }

    public void login(View view) {
        TextView tvMsg = findViewById(R.id.tv_msg);
        Login login = component.login();
        tvMsg.setText(String.format("%s %s login success.", login.getUsername(), login.getPassword()));
    }
}
```

* 所有@BindsInstance方法必须在build()之前调用。
* 如果@BindsInstance方法的参数可能为 null，需要再用@Nullable标记，同时标注 Inject 的地方也需要用@Nullable标记。这时 Builder 也可以不调用@BindsInstance方法，这样 Component 会默认设置 instance 为 null。


[Demo Github地址](https://github.com/zhy060307/dagger2) 

