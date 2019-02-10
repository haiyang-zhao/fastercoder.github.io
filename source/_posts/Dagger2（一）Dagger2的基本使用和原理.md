title: Dagger2（一）Dagger2的基本使用和原理
date: 2019-02-10 15:37:01
categories: Dagger2
tags: Dagger2
---


# Dagger2的基本使用

#### 引入dagger2

在build.gradle文件中添加依赖

```
implementation 'com.google.dagger:dagger:2.20'
annotationProcessor 'com.google.dagger:dagger-compiler:2.20'
```

Android gradle plugin 版本小于2.2时要引入 [apt插件](https://bitbucket.org/hvisser/android-apt)

#### 使用 @Inject 注入依赖
```java
public class MainActivity extends AppCompatActivity {
    @Inject
    PrintService printService;
    //.......
}
```
编译后 在build/generated/source/apt/ 下可以看到生成的代码

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
    injectPrintService(instance, printServiceProvider.get());
  }
  
  public static void injectPrintService(MainActivity instance, PrintService printService) {
    instance.printService = printService;
  }
}
```
* injectPrintService()方法注入依赖，从 instance.printService = printService;可以看出 @Inject 注入的成员变量不能为private

#### 创建依赖对象的实例
```java
public class PrintService {
    private final Printer printer;

    @Inject
    public PrintService(Printer printer) {
        this.printer = printer;
    }

    public void startWork() {
        this.printer.print();
    }
}
```

Dagger会生成一个PrintService_Factory类
```java
public final class PrintService_Factory implements Factory<PrintService> {
  private final Provider<Printer> printerProvider;

  public PrintService_Factory(Provider<Printer> printerProvider) {
    this.printerProvider = printerProvider;
  }

  @Override
  public PrintService get() {
    return provideInstance(printerProvider);
  }

  public static PrintService provideInstance(Provider<Printer> printerProvider) {
    return new PrintService(printerProvider.get());
  }

  public static PrintService_Factory create(Provider<Printer> printerProvider) {
    return new PrintService_Factory(printerProvider);
  }

  public static PrintService newPrintService(Printer printer) {
    return new PrintService(printer);
  }
}
```

#### @Module @Provides 提供依赖

使用@Inject标注构造方法提供依赖时有限制，比如：

* 依赖对象为接口，而接口是没有构造方法
* @Inject不能标注到第三方库
* 构造方法中的参数是动态配置的

这时就需要@Provides标注的方法提供依赖，而@Provides使用时，必须方法@Module标注的类中。

```java
@Module
public class PrinterModule {

    @Provides
    public Printer providePrinter() {
        return new PdfPrinter();
    }

    @Provides
    public Handler provideHandler() {
        return new Handler();
    }
}

```
* Module的优先级比@Inject标注构造函数的高

dagger会为每一个@Provides标注的方法生成一个Factory
```java
PrinterModule_ProvideHandlerFactory
PrinterModule_ProvidePrinterFactory
```

```java
public final class PrinterModule_ProvidePrinterFactory implements Factory<Printer> {
  private final PrinterModule module;

  public PrinterModule_ProvidePrinterFactory(PrinterModule module) {
    this.module = module;
  }

  @Override
  public Printer get() {
    return provideInstance(module);
  }

  public static Printer provideInstance(PrinterModule module) {
    return proxyProvidePrinter(module);
  }

  public static PrinterModule_ProvidePrinterFactory create(PrinterModule module) {
    return new PrinterModule_ProvidePrinterFactory(module);
  }

  public static Printer proxyProvidePrinter(PrinterModule instance) {
    return Preconditions.checkNotNull(
        instance.providePrinter(), "Cannot return null from a non-@Nullable @Provides method");
  }
}

```

#### @Component 作为桥梁，关联依赖和被依赖的对象

```java
@Component(modules = PrinterModule.class)
public interface PrintComponent {
    void inject(MainActivity activity);

}
```

* Component类必须是接口后者抽象类

在MainActivity调用PrintComponent的inject方法完成注入
```java
 DaggerPrintComponent.builder()
                .build().inject(this);
```

dagger为PrintComponent生成的实现类如下：
```java
public final class DaggerPrintComponent implements PrintComponent {
  private PrinterModule printerModule;

  private DaggerPrintComponent(Builder builder) {
    this.printerModule = builder.printerModule;
  }

  public static Builder builder() {
    return new Builder();
  }

  public static PrintComponent create() {
    return new Builder().build();
  }

  private PrintService getPrintService() {
    return new PrintService(PrinterModule_ProvidePrinterFactory.proxyProvidePrinter(printerModule));
  }

  @Override
  public void inject(MainActivity activity) {
    injectMainActivity(activity);
  }

  private MainActivity injectMainActivity(MainActivity instance) {
    MainActivity_MembersInjector.injectPrintService(instance, getPrintService());
    return instance;
  }

  public static final class Builder {
    private PrinterModule printerModule;

    private Builder() {}

    public PrintComponent build() {
      if (printerModule == null) {
        this.printerModule = new PrinterModule();
      }
      return new DaggerPrintComponent(this);
    }

    public Builder printerModule(PrinterModule printerModule) {
      this.printerModule = Preconditions.checkNotNull(printerModule);
      return this;
    }
  }
}

```

[Demo Github地址](https://github.com/zhy060307/dagger2)


