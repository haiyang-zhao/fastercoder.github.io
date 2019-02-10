title: Dagger2（三）Subcomponents
date: 2019-02-10 15:41:32
categories: Dagger2
tags: Dagger2
---

Component之间除了dependencies依赖关系外，还可以有继承关系。
* 依赖关系 
> 一个 Component 依赖其他 Compoent 公开的依赖实例，用 Component 中的dependencies声明
* 继承关系 
> 一个 Component 继承（也可以叫扩展）某 Component 提供更多的依赖，SubComponent 就是继承关系的体现。

#### 依赖关系 dependencies
MainComponent依赖AppComponent。
*  MainComponent要使用AppComponent的PrintService，MainComponent必须显示提供方法将其暴露出来。
*  MainComponent和AppComponent的Scope不能相同，因为他们的生命周期不同。
```
@Singleton
@Component(modules = {AppModule.class,
        PrinterModule.class})
public interface AppComponent {

    void inject(App app);

    PrintService printService();

}
//使用dependencies
@ActivityScope
@Component(dependencies = AppComponent.class)
public interface MainComponent {
    void inject(MainActivity activity);

}

```

#### 继承关系 Subcomponents

##### 声明subcomponent

* 和Component类似，必须是abstract class 或者interface.使用@Subcomponent标注。
* Subcomponent和parent component的Scope不能相同，所有用自定义Scope @ActivityScope
* SubComponent 必须显式地声明 Subcomponent.Builder，parent Component 需要用 Builder 来创建 SubComponent

```
@ActivityScope
@Subcomponent
public interface SubMainComponent {

    @Subcomponent.Builder
    interface Builder {

        SubMainComponent build();
    }

    void inject(MainActivity mainActivity);
}


@Singleton
@Component(modules = {AppModule.class, PrinterModule.class})
public interface AppComponent {

    void inject(App app);// 继承关系中不用显式地提供暴露依赖实例的接口

    SubMainComponent.Builder mainBuilder(); // 用来创建 Subcomponent

    SubSecondComponent.Builder secondBuilder();
}

```
SubComponent 编译时不会生成 DaggerXXComponent，需要通过 parent Component 的获取 SubComponent.Builder 方法获取 SubComponent 实例

```
MainActivity中注入依赖:

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    App.component().mainBuilder()
            .build()
            .inject(this);
}
```
Dagger生成的代码如下：
```
public final class DaggerAppComponent implements AppComponent {
  //....

  @Override
  public SubMainComponent.Builder mainBuilder() {
    return new SubMainComponentBuilder();
  }

  @Override
  public SubSecondComponent.Builder secondBuilder() {
    return new SubSecondComponentBuilder();
  }

  private final class SubMainComponentBuilder implements SubMainComponent.Builder {
    @Override
    public SubMainComponent build() {
      return new SubMainComponentImpl(this);
    }
  }

  private final class SubMainComponentImpl implements SubMainComponent {
    private SubMainComponentImpl(SubMainComponentBuilder builder) {}

    @Override
    public void inject(MainActivity mainActivity) {
      injectMainActivity(mainActivity);
    }

    private MainActivity injectMainActivity(MainActivity instance) {
      MainActivity_MembersInjector.injectPrintService(
          instance, DaggerAppComponent.this.printServiceProvider.get());
      return instance;
    }
  }

  private final class SubSecondComponentBuilder implements SubSecondComponent.Builder {
    @Override
    public SubSecondComponent build() {
      return new SubSecondComponentImpl(this);
    }
  }

  private final class SubSecondComponentImpl implements SubSecondComponent {
    private SubSecondComponentImpl(SubSecondComponentBuilder builder) {}

    @Override
    public void inject(SecondActivity secondActivity) {
      injectSecondActivity(secondActivity);
    }

    private SecondActivity injectSecondActivity(SecondActivity instance) {
      SecondActivity_MembersInjector.injectPrintService(
          instance, DaggerAppComponent.this.printServiceProvider.get());
      return instance;
    }
  }
}
```
从代码中可以看出，SubMainComponentImpl中直接使用了DaggerAppComponent.this.printServiceProvider.get()，也就是说Subcomponent中可以使用ParentComponent提供 的依赖


##### 依赖关系 vs 继承关系
相同点：

* 两者都能复用其他 Component 的依赖

* 有依赖关系和继承关系的 Component 不能有相同的 Scope

区别：

* 依赖关系中被依赖的 Component 必须显式地提供公开依赖实例的接口，而 SubComponent 默认继承 parent Component 的依赖。

* 依赖关系会生成两个独立的 DaggerXXComponent 类，而 SubComponent 不会生成 独立的 DaggerXXComponent 类。

#### Repeated modules 

当相同的 Module 注入到 parent Component 和它的 SubComponent 中时，则每个 Component 都将自动使用这个 Module 的同一实例。也就是如果在 SubComponent.Builder 中调用相同的 Module 或者在返回 SubComponent 的抽象工厂方法中以重复 Module 作为参数时，会出现错误。（前者在编译时不能检测出，是运行时错误）

```
@Component(modules = {RepeatedModule.class, ...})
interface ComponentOne {
  ComponentTwo componentTwo(RepeatedModule repeatedModule); // COMPILE ERROR!
  ComponentThree.Builder componentThreeBuilder();
}

@Subcomponent(modules = {RepeatedModule.class, ...})
interface ComponentTwo { ... }

@Subcomponent(modules = {RepeatedModule.class, ...})
interface ComponentThree {
  @Subcomponent.Builder
  interface Builder {
    Builder repeatedModule(RepeatedModule repeatedModule);
    ComponentThree build();
  }
}

DaggerComponentOne.create().componentThreeBuilder()
    .repeatedModule(new RepeatedModule()) // UnsupportedOperationException!
    .build();
```

[Demo Github地址](https://github.com/zhy060307/dagger2)
