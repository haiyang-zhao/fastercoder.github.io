title: Dagger2（五）Android中的扩展使用
date: 2019-02-10 15:45:03
categories: Dagger2
tags: Dagger2
---

> 在Android平台上使用Dagger的一个主要的不同是，很多类的实例化依赖于操作系统本身，像是Activity和Fragment，但是Dagger最理想的工作方式是它能够构造所有需要注入的类的实例。所以，你必须在它们（Activity、Fragment等）的生命周期中进行成员的注入。很多类看上去跟下面类似：

```java
public class FrombulationActivity extends Activity {
  @Inject Frombulator frombulator;

  @Override
  public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    //先写如下代码， 否则frombulator可能为null!
    ((SomeApplicationBaseType) getContext().getApplicationContext())
        .getApplicationComponent()
        .newActivityComponentBuilder()
        .activity(this)
        .build()
        .inject(this);
    // ...其它代码
  }
}
```
这么做有以下问题：

1. 复制粘贴同样的代码使得以后想要重构变得困难。越来越多的这样复制粘贴的代码，开发者反而对这段代码的作用了解的更少。
2. 更加重要的是，它需要被注入类（FrombulationActivity）知道它的注入类。即使这是通过接口实现的，而不是具体的类。但是，这仍然破坏了依赖注入的的核心原则：一个类不应该对它是如何被注入的有任何的了解.
(a class shouldn’t know anything about how it is injected.)

dagger.android 可以用来简化以上步骤。

#### build.gradle添加依赖:
```
implementation 'com.google.dagger:dagger-android:2.20'
implementation 'com.google.dagger:dagger-android-support:2.20'
```
####  Activity的注入
1. 在你的Application Component中加入AndroidInjectionModule模块，以提供所有基本类型的绑定。

```
@Component(modules = {AndroidInjectionModule.class,
        ...})
public interface ApplicationComponent {
    ...
}
```

AndroidInjectionModule是一个普通的Module,其中声明了两种Multibinds
```java
@Beta
@Module
public abstract class AndroidInjectionModule {
  @Multibinds
  abstract Map<Class<?>, AndroidInjector.Factory<?>> classKeyedInjectorFactories();

  @Multibinds
  abstract Map<String, AndroidInjector.Factory<?>> stringKeyedInjectorFactories();

  private AndroidInjectionModule() {}
}

```

2. 声明subcomponent并且实现接口AndroidInjector<YourActivity>，该subcomponent需要有一个被@Subcomponent.Builder注解的并扩展自AndroidInjector.Builder<YourActivity>的构造器：

```java
@Subcomponent
public interface MainActivitySubComponent extends AndroidInjector<MainActivity> {
    @Subcomponent.Builder
    abstract class Builder extends AndroidInjector.Builder<MainActivity> {
    }
}

```

3. 声明过subcomponent之后，把它通过如下方式加入到主component体系中：定义一个提供该subcomponent builder的module，并且把该module加入到你的AppComponent中。

```java
@Module(subcomponents = MainActivitySubComponent.class)
public abstract class MainModule {

    @Binds
    @IntoMap
    @ClassKey(MainActivity.class)
    abstract AndroidInjector.Factory<?>
    bindYourMainActivityInjectorFactory(MainActivitySubComponent.Builder builder);
}
```
* 注意：如果你的subcomponent和它的builder除了第2步中提及的方法或者超类没有其它的内容，你可以用 @ContributesAndroidInjector生成2、3步中的一切。现在不需要步骤2和3，你只需声明一个abstract module，返回你所需的activity（用 @ContributesAndroidInjector注解），可以声明subcomponent需要的其它的module。如果这个subcomponent需要scope注解，也可以声明：


```java

@Module
public abstract class ActivityBuildersModule {
    @ActivityScope
    @ContributesAndroidInjector(modules = {/*subcomponent需要的module*/})
    abstract MainActivity contributeMainActivity();
}

@Component(modules = {AndroidInjectionModule.class,
        ActivityBuildersModule.class})
public interface ApplicationComponent {
    ...
}
```

ContributesAndroidInjector注解会生成如下代码，和我们手写的是一样的。
```java
@Module(
  subcomponents = ActivityBulidersModule_ContributeMainActivity.MainActivitySubcomponent.class
)
public abstract class ActivityBulidersModule_ContributeMainActivity {
  private ActivityBulidersModule_ContributeMainActivity() {}

  @Binds
  @IntoMap
  @ClassKey(MainActivity.class)
  abstract AndroidInjector.Factory<?> bindAndroidInjectorFactory(
      MainActivitySubcomponent.Builder builder);

  @Subcomponent
  @ActivityScope
  public interface MainActivitySubcomponent extends AndroidInjector<MainActivity> {
    @Subcomponent.Builder
    abstract class Builder extends AndroidInjector.Builder<MainActivity> {}
  }
}

```

4. 让你的Application实现HasActivityInjector并且@Inject DispatchingAndroidInjector<Activity>而后从方法activityInjector()（接口HasActivityInjector中的方法）返回：

```java
public class App extends Application implements HasActivityInjector {

    @Inject
    DispatchingAndroidInjector<Activity> dispatchingActivityInjector;

    @Override
    public void onCreate() {
        super.onCreate();
        DaggerApplicationComponent.create().inject(this);
    }

    @Override
    public AndroidInjector<Activity> activityInjector() {
        return dispatchingActivityInjector;
    }
}

```

5. 最后，在 Activity.onCreate() 方法中在super.onCreate()之前调用AndroidInjection.inject(this)

```java
public class MainActivity extends AppCompatActivity {

    private static final String TAG = MainActivity.class.getSimpleName();

    @Inject
    UserService userService;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        AndroidInjection.inject(this);

        Log.d(TAG, userService.getUserName());
    }
}
```

#### How did that work? 如何工作？

> AndroidInjection.inject() 从Application中获取了一个 DispatchingAndroidInjector<Activity>，并把activity实例传入方法 inject(Activity)中。 DispatchingAndroidInjector 根据activity的class来查找 AndroidInjector.Factory（即 YourActivitySubcomponent.Builder），创建 AndroidInjector (即YourActivitySubcomponent)， 然后把你的activity实例传入方法 inject(YourActivity)中。

AndroidInjection.inject(),获取App对象的dispatchingActivityInjector，调用inject(MainAcitivty)

```
public static void inject(Activity activity) {
    checkNotNull(activity, "activity");
    Application application = activity.getApplication();
    if (!(application instanceof HasActivityInjector)) {
      throw new RuntimeException(
          String.format(
              "%s does not implement %s",
              application.getClass().getCanonicalName(),
              HasActivityInjector.class.getCanonicalName()));
    }

    AndroidInjector<Activity> activityInjector =
        ((HasActivityInjector) application).activityInjector();
    checkNotNull(activityInjector, "%s.activityInjector() returned null", application.getClass());

    activityInjector.inject(activity);
  }
```
再看看DispatchingAndroidInjector的inject()方法，调用的是maybeInject()方法,
让后根据activity的class来查找 AndroidInjector.Factory（即 YourActivitySubcomponent.Builder），创建 AndroidInjector (即YourActivitySubcomponent)， 然后把你的activity实例传入方法 inject(YourActivity)中。

```
@Override
public void inject(T instance) {
    boolean wasInjected = maybeInject(instance);
    if (!wasInjected) {
      throw new IllegalArgumentException(errorMessageSuggestions(instance));
    }
}

public boolean maybeInject(T instance) {
    
    //获取MainActivitySubcomponentImpl
    Provider<AndroidInjector.Factory<?>> factoryProvider =
        injectorFactories.get(instance.getClass().getName());
    if (factoryProvider == null) {
      return false;
    }

    @SuppressWarnings("unchecked")
    AndroidInjector.Factory<T> factory = (AndroidInjector.Factory<T>) factoryProvider.get();
    try {
      AndroidInjector<T> injector =
          checkNotNull(
              factory.create(instance), "%s.create(I) should not return null.", factory.getClass());

      //调用MainActivitySubcomponentImpl的inject(MainActivity)
      injector.inject(instance);
      return true;
    } catch (ClassCastException e) {
      throw new InvalidInjectorBindingException(
          String.format(
              "%s does not implement AndroidInjector.Factory<%s>",
              factory.getClass().getCanonicalName(), instance.getClass().getCanonicalName()),
          e);
    }
  }
  
  //MainActivitySubcomponentImpl
  private final class MainActivitySubcomponentImpl
      implements ActivityBulidersModule_ContributeMainActivity.MainActivitySubcomponent {
    private MainActivitySubcomponentImpl(MainActivitySubcomponentBuilder builder) {}

    @Override
    public void inject(MainActivity arg0) {
      injectMainActivity(arg0);
    }

    private MainActivity injectMainActivity(MainActivity instance) {
      MainActivity_MembersInjector.injectUserService(instance, new UserService());
      return instance;
    }
  }
```

#### dagger.android.support 进一步简化

1. ApplicationComponent继承AndroidInjector<App>，并且添加Component.Builder,Builder类继承AndroidInjector.Builder<App>
```java
@Component(modules = {AndroidInjectionModule.class,
        ActivityBuildersModule.class})
public interface ApplicationComponent extends AndroidInjector<App> {

    @Component.Builder
    abstract class Builder extends AndroidInjector.Builder<App> {
    }

}
```

2. 我们的Application继承dagger.android.support.DaggerApplication，重写applicationInjector方法。
```java
public class App extends DaggerApplication {
  
    @Override
    protected AndroidInjector<App> applicationInjector() {
        return DaggerApplicationComponent.builder().create(this);
    }
}
```
3. Activity继承DaggerAppCompatActivity,无需调用AndroidInjection.inject()
```java
public class MainActivity extends DaggerAppCompatActivity {

    private static final String TAG = MainActivity.class.getSimpleName();

    @Inject
    UserService userService;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        Log.d(TAG, userService.getUserName());
    }
}

```









[Demo Github地址](https://github.com/zhy060307/dagger2)


参考：
https://www.jianshu.com/p/ad777c73b528
