---
layout: di
title: Guice
date: 2019-02-10 15:34:45
tags:
---


### SOLID 
* Single responsibility   单一职责
> a class should have only a single responsibility
* Open/closed  开闭原则
> software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification
* Liskov substitution  里氏替换原则
> objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program.
* Interface segregation  接口隔离
> many client-specific interfaces are better than one general-purpose interface.
* Dependency inversion   依赖倒置
> one should depend upon abstractions, [not] concretions.
### 依赖倒置 (Dependency inversion principle)
* High-level modules should not depend on low-level modules. Both should depend on abstractions.
* Abstractions should not depend on details.Details should depend on abstractions.


### 依赖注入   DI (dependency injection)

> Dependency injection is one form of the broader technique of inversion of control（IoC）.

* Reduced Dependencies
* More Reusable Code
* More Testable Code
* More Readable Code

### [JSR330](https://jcp.org/en/jsr/detail?id=330) 


JSR330在javax.inject中规定了依赖注入的标准注解(Annotations)。包括：

* @Inject : 标记为“可注入”。可用于构造器(constructors), 方法(methods)或字段(fields)
* @Qualifier : 限定器
* @Scope : 标记作用域
* @Named : 基于 String 的限定器
* @Singleton : 标记为单例




### Guice 
> Guice (pronounced 'juice') is a lightweight dependency injection framework for Java 6 and above, brought to you by Google.


#### Bindings

* Linked Binding
    ```java
    @Override
    protected void configure() {
        bind(Printer.class).to(PdfPrinter.class);
        bind(PdfPrinter.class).to(ColorPdfPrinter.class);
    
    }
    ```

* BindingAnnotations

    ```java
    @BindingAnnotation
    @Target({FIELD, PARAMETER, METHOD})
    @Retention(RUNTIME)
    public @interface Alipay {
    }
    ```

    ```java
    bind(Pay.class)
    .annotatedWith(Alipay.class)
    .to(AlipayImpl.class);
    ```
    
    ```java
    @Inject
    public BillService(@Alipay Pay pay) {
        this.pay = pay;
    }
    ```

* Named

    ```java
    bind(Pay.class)
    .annotatedWith(Names.named("wechat"))
    .to(WechatPayImpl.class);
    
    @Inject
    public void setPay(@Named("wechat") Pay pay) {
        this.pay = pay;
    }
    ```

* Constant Bindings
    ```java
        
    @Inject
    public void connectDatabase(@Named("JBDC") String dbUrl) {
            //...
    }
    
    bind(String.class).annotatedWith(Names.named("JBDC")).toInstance("jdbc:mysql://localhost:5326/emp");
    
    ```

* ProvidesMethods
    ```java
    private static class CheckVersionModule extends AbstractModule {
            @Override
            protected void configure() {
                bind(String.class)
                        .annotatedWith(Names.named("update url"))
                        .toInstance("http://console.qa.roomis.com.cn/api/client-apps/latest");
    
            }
    
            @Provides
            public Request buildRequest(@Named("update url") String url) {
                return new Request.Builder()
                        .url(url)
                        .addHeader("X-Consumer-Custom-ID", "roomis-k12-own")
                        .addHeader("deviceSn", "000ffd4a014f")
                        .addHeader("apikey", "0001e01807cd4a9ebab4bbb5576f1815")
                        .build();
    
            }
    
            @Provides
            public OkHttpClient provideClient() {
                OkHttpClient.Builder builder = new OkHttpClient.Builder();
                builder.connectTimeout(60, TimeUnit.SECONDS)
                        .readTimeout(30, TimeUnit.SECONDS)
                        .writeTimeout(30, TimeUnit.SECONDS);
    
                return builder.build();
            }
        }
    ```

* ProviderBindings
    ```java
    class PrinterModule extends AbstractModule {
        @Override
        protected void configure() {
            bind(Printer.class)
                    .toProvider(PrinterProvider.class);
        }
    }
    
    class PrinterProvider implements Provider<Printer> {
    
        @Override
        public Printer get() {
            return new PdfPrinter();
        }
    }
    ```
* Constructor Bindings
    ```java
    public class _06_ConstructorBindings {
    
        public static void main(String[] args) {
            Injector injector = Guice.createInjector(new TextEditorModule());
            TextEditor editor = injector.getInstance(TextEditor.class);
            editor.makeSpellCheck();
        }
    }
    
    class TextEditor {
        private SpellChecker spellChecker;
    
        @Inject
        public TextEditor(SpellChecker spellChecker) {
            this.spellChecker = spellChecker;
        }
    
        public void makeSpellCheck() {
            spellChecker.checkSpelling();
        }
    }
    
    //Binding Module
    class TextEditorModule extends AbstractModule {
        @Override
    
        protected void configure() {
            try {
                bind(SpellChecker.class)
                        .toConstructor(SpellCheckerImpl.class.getConstructor(String.class));
            } catch (NoSuchMethodException | SecurityException e) {
                System.out.println("Required constructor missing");
            }
            bind(String.class)
                    .annotatedWith(Names.named("JDBC"))
                    .toInstance("jdbc:mysql://localhost:5326/emp");
        }
    }
    
    //spell checker interface
    interface SpellChecker {
        void checkSpelling();
    }
    
    //spell checker implementation
    class SpellCheckerImpl implements SpellChecker {
        private String dbUrl;
    
        public SpellCheckerImpl() {
        }
    
        public SpellCheckerImpl(@Named("JDBC") String dbUrl) {
            this.dbUrl = dbUrl;
        }
    
        @Override
        public void checkSpelling() {
            System.out.println("Start checkSpelling.");
            System.out.println(dbUrl);
        }
    }
    ```
#### Scopes

默认创建新实例
* @Singleton
```java
@Singleton
public class PrintService {
//...
}
```
或者
```java
@Singleton
@Provides
public OkHttpClient provideClient() {
    
}
```
或者
```java
bind(Printer.class).to(PdfPrinter.class).in(Singleton.class);
```
Eager Singletons
立即加载
```java
bind(TransactionLog.class).to(InMemoryTransactionLog.class).asEagerSingleton();

```
* @SessionScoped
* @RequestScoped


#### Injections

* Constructor Injection
* Method Injection
* Field Injection
```java
@Singleton
public class PrintService {


    private final Printer printer;

    @Inject
    public PrintService(Printer printer) {
        this.printer = printer;
    }


//    @Inject
//    private Printer printer;

//    private Printer printer;
//
//    @Inject
//    public void setPrinter(Printer printer) {
//        this.printer = printer;
//    }

    public void startWork() {
        this.printer.print();
    }
}

```


---

## Dagger2
> Dagger is a fully static, compile-time dependency injection framework for both Java and Android. It is an adaptation of an earlier version created by Square and now maintained by Google.

why reinvent the wheel? 

> Dagger 2 is the first to implement the full stack with generated code. 


#### Start
build.gradle
    
     implementation 'com.google.dagger:dagger:2.20'
    annotationProcessor 'com.google.dagger:dagger-compiler:2.20'

#### 


* Inject
* Module
* Provides
* Component
* Scope
* Qualifier



    















 


#
https://en.wikipedia.org/wiki/SOLID </br>
https://en.wikipedia.org/wiki/Dependency_inversion_principle </br>
https://blog.csdn.net/briblue/article/details/75093382 </br>
http://tutorials.jenkov.com/dependency-injection/dependency-injection-benefits.html </br>
https://blog.csdn.net/xtayfjpk/article/details/40657781


https://google.github.io/dagger/ </br>
https://blog.csdn.net/qq_17766199/article/details/73030696
