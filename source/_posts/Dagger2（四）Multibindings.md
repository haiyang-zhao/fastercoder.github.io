title: Dagger2（四）Multibindings
date: 2019-02-10 15:43:05
categories: Dagger2
tags:  Dagger2
---

### Multibindings

> Dagger allows you to bind several objects into a collection even when the objects are bound in different modules using multibindings.

Dagger的Multibindings功能可以生成集合，无需直接依赖其他单独的绑定。  

#### Set multibindings

可以将一个或多个值绑定到一个Set中。

* IntoSet,提供单一的值。
* ElementsIntoSet，提供subset。
```java
@Module
public class SetModuleA {

    @Provides
    @IntoSet
    public String provideOneString() {
        return "ABC";
    }

    @Provides
    @ElementsIntoSet
    public Set<String> provideSomeStrings() {
        return new HashSet<>(Arrays.asList("DEF", "GHI"));
    }

}
```
然后注入Set
```java
public class FourthActivity extends AppCompatActivity {

    @Inject
    Set<String> mySet;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_third);
        DaggerMultiBindingsComponent
                .create().inject(this);

    }

    public void setBindings(View view) {
        Log.d("Inject mySet value is", mySet.toString());
    }
}
Logcat:com.zhy.dagger2 D/Inject mySet value is: [ABC, DEF, GHI]

```
Daager生成的代码：

DaggerMultiBindingsComponent的关键代码：
```java
public final class DaggerMultiBindingsComponent implements MultiBindingsComponent {
  private SetModuleA setModuleA;
  //....
  private Set<String> getSetOfString() {
    return SetBuilder.<String>newSetBuilder(2)
        .add(SetModuleA_ProvideOneStringFactory.proxyProvideOneString(setModuleA))
        .addAll(SetModuleA_ProvideSomeStringsFactory.proxyProvideSomeStrings(setModuleA))
        .build();
  }

  @Override
  public void inject(FourthActivity fourthActivity) {
    injectFourthActivity(fourthActivity);
  }

  private FourthActivity injectFourthActivity(FourthActivity instance) {
    FourthActivity_MembersInjector.injectMySet(instance, getSetOfString());
    return instance;
  }
}

```
可以看到getSetOfString()用来组装Set.

##### Qualifier限定要注入的Set
```java
@Module
public class SetModuleB {
    
    @Provides
    @ElementsIntoSet
    @MyQualifier
    public Set<String> provideSpecialSomeStrings() {
        return new HashSet<>(Arrays.asList("Hello", "Dagger2"));
    }
}

public class FourthActivity extends AppCompatActivity {

    private static final String TAG = FourthActivity.class.getSimpleName();
    @Inject
    Set<String> mySet;

    @Inject
    @MyQualifier
    Lazy<Set<String>> myOnlySet;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_third);
        DaggerMultiBindingsComponent
                .create().inject(this);

    }

    public void setBindings(View view) {
        Log.d(TAG, "Inject mySet value is" + mySet.toString());
        Log.d(TAG, "Inject myOnlySet value is" + myOnlySet.get().toString());

    }
}
Logcat:
2019-02-10 07:06:22.761 17005-17005/com.zhy.dagger2 D/FourthActivity: Inject mySet value is[ABC, DEF, GHI]
2019-02-10 07:06:22.761 17005-17005/com.zhy.dagger2 D/FourthActivity: Inject myOnlySet value is[Hello, Dagger2]

```
* Lazy<Set<String>>不能写成 Set<Lazy<String>>

#### Map multibindings
使用@IntoMap

##### Simple map keys
* String,Class<?>,boxed primitives 作为key。
* 枚举Enmu，更具体参数化类 作为key。
```java
@Module
public class MapModuleA {

    @Provides
    @IntoMap
    @StringKey("foo")
    public Long provideStringKeyValue() {
        return 100L;
    }


    @Provides
    @IntoMap
    @LongKey(200)
    public String providePrimitivesKeyValue() {
        return "value for LongKey";
    }

    @Provides
    @IntoMap
    @ClassKey(MyClassKey.class)
    public String provideClassKeyValue() {
        return "value for MyClassKey";
    }
}

@Component(modules = {SetModuleA.class,
        SetModuleB.class,
        MapModuleA.class})
public interface MultiBindingsComponent {

    void inject(FourthActivity fourthActivity);

    Map<String, Long> longsByString();

    Map<Long, String> stringsByLong();

    Map<Class<?>, String> stringsByClass();
}

//FourthActivity
public void mapBindings(View view) {
        Log.d(TAG, "component.longsByString is" + component.longsByString());
        Log.d(TAG, "component.stringsByLong is" + component.stringsByLong());
        Log.d(TAG, "component.stringsByClass is" + component.stringsByClass());

    }
Logcat:
2019-02-10 07:33:47.832 17796-17796/com.zhy.dagger2 D/FourthActivity: component.longsByString is{foo=100}
2019-02-10 07:33:47.833 17796-17796/com.zhy.dagger2 D/FourthActivity: component.stringsByLong is{200=value for LongKey}
2019-02-10 07:33:47.835 17796-17796/com.zhy.dagger2 D/FourthActivity: component.stringsByClass is{class com.zhy.dagger2.multibindings.MyClassKey=value for MyClassKey}

```
对于具有枚举键或更具体参数化类的映射，要自定义key，使用@MapKey注解标注。

```java
public enum Color {
    RED, GREEN, BLUE
}

@MapKey
public @interface ColorKey {
    Color value();
}

@MapKey
public @interface MyNumberClassKey {
    Class<? extends Number> value();
}

@Module
public class MapModuleB {
    @Provides
    @IntoMap
    @ColorKey(Color.BLUE)
    static String provideColorValue() {
        return "value for BLUE";
    }

    @Provides
    @IntoMap
    @MyNumberClassKey(BigDecimal.class)
    static String provideBigDecimalValue() {
        return "value for BigDecimal";
    }
}

@Component(modules = {MapModuleB.class})
public interface MultiBindingsComponent {

    void inject(FourthActivity fourthActivity);

    Map<Color, String> myEnumStringMap();

    Map<Class<? extends Number>, String> stringsByNumberClass();
}

//FourthActivity
Log.d(TAG, "component.myEnumStringMap is" + component.myEnumStringMap());
Log.d(TAG, "component.stringsByNumberClass is" + component.stringsByNumberClass());

Logcat:
2019-02-10 07:56:39.563 18668-18668/com.zhy.dagger2 D/FourthActivity: component.myEnumStringMap is{BLUE=value for BLUE}
2019-02-10 07:56:39.563 18668-18668/com.zhy.dagger2 D/FourthActivity: component.stringsByNumberClass is{class java.math.BigDecimal=value for BigDecimal}
```

##### Complex map keys

Key可以是多种形式的组合，这个时候@MapKey(unwrapValue = false)，unwrapValue值要设为false。
实际开发用到很少。


#### Declaring multibindings

使用@Multibinds在抽象类的Module中声明。

```java
@Module
public abstract class MultibindsModule {

    @Multibinds
    abstract Set<Foo> aSet();

    @Multibinds
    @MyQualifier
    abstract Set<Foo> aQualifiedSet();

    @Multibinds
    abstract Map<String, Foo> aMap();

    @Multibinds
    @MyQualifier
    abstract Map<String, Foo> aQualifiedMap();

    @Provides
    static Object usesMultibindings(Set<Foo> set, @MyQualifier Map<String, Foo> map) {
        return "";
    }
}

```

#### Inherited subcomponent multibindings

* ChildComponent包含ParentComponent中multibindings的set或map

```java
@Subcomponent(modules = ChildModule.class)
public interface ChildComponent {
    Set<String> strings();

    Map<String, String> stringMap();
}

@Module
public class ChildModule {
    @Provides
    @IntoSet
    static String string3() {
        return "child string 3";
    }

    @Provides
    @IntoSet
    static String string4() {
        return "child string 4";
    }

    @Provides
    @IntoMap
    @StringKey("c")
    static String stringC() {
        return "child string C";
    }

    @Provides
    @IntoMap
    @StringKey("d")
    static String stringD() {
        return "child string D";
    }
}



@Component(modules = ParentModule.class)
public interface ParentComponent {

    Set<String> strings();

    Map<String, String> stringMap();

    ChildComponent childComponent();
}

@Module
class ParentModule {
    @Provides
    @IntoSet
    static String string1() {
        return "parent string 1";
    }

    @Provides
    @IntoSet
    static String string2() {
        return "parent string 2";
    }

    @Provides
    @IntoMap
    @StringKey("a")
    static String stringA() {
        return "parent string A";
    }

    @Provides
    @IntoMap
    @StringKey("b")
    static String stringB() {
        return "parent string B";
    }
}

//FourthActivity
public void subComponentsBindings(View view) {

    ParentComponent parentComponent = DaggerParentComponent.create();
    ChildComponent childComponent = parentComponent.childComponent();

    Log.d(TAG, " parentComponent.strings is" + parentComponent.strings());
    Log.d(TAG, " parentComponent.stringMap is" + parentComponent.stringMap());

    Log.d(TAG, " childComponent.strings is" + childComponent.strings());
    Log.d(TAG, " childComponent.stringMap is" +childComponent.stringMap());
}

Logcat:
com.zhy.dagger2 D/FourthActivity:  parentComponent.strings is[parent string 1, parent string 2]
com.zhy.dagger2 D/FourthActivity:  parentComponent.stringMap is{a=parent string A, b=parent string B}
com.zhy.dagger2 D/FourthActivity:  childComponent.strings is[parent string 1, parent string 2, child string 4, child string 3]
com.zhy.dagger2 D/FourthActivity:  childComponent.stringMap is{a=parent string A, b=parent string B, c=child string C, d=child string D}

```


[Demo Github地址](https://github.com/zhy060307/dagger2)
