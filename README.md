# WandFix


当你的项目已经上线，一个BUG被发现却已经为时已晚，这时候悔恨自责都已经于事无补，甚至还会被伙伴们打上不靠谱的标签，这时候你想不想像哈利波特一样，魔法杖一挥BUG瞬间消除，没错拿着WindFix你就可以实现这一点。  

## About

WandFix是一个可以使用注解来注入实现类对象的库，基于java APT+反射+classloader，因此不需要使用new来初始化对象。另外它还可以动态加载指定.apk .dex .jar .class文件，使用注入设置优先级的方式来选择注入class类的来源，因此利用此种方法，该项目还可以使得被注解的类获得热修复的能力！  

使用WandFix只需要对变量名添加注解即可实现对象的注入。而热修复功能的实现既是用对象注入的这种途径，通过更换热修复包替换实现类，来达到每次注入的对象都为最后改动过的，实现热修复的目的。当然，如果您不想使用热修复功能，只是想方便的注入对象，直接添加注解也是可以的，因为如果它在热修复包中没有找到对应的实现类的话，会尝试从项目本地来加载对应的类，保证程序的稳定。

本项目Demo效果预览：

![demo预览](./preview/demo.gif)


优点：
- 类似于黄油刀可以直接对成员变量添加@InjectObject("实现类全名")注解,来绑定热修复包中的实现类。
- 无需关闭应用即可使修复包生效。
- 与mvp模式搭配使用效果最佳。
- 可以自己定义需要热修复的类。
- 可以自己配置dex加密算法，保护dex文件的安全。
- 可以通过注解单独设置某个对象是否禁用双亲委托。
- Activity动态代理功能，2019-2-19 新增

目前只支持java类文件的替换，资源文件及布局文件替换相关正在研究中，后续会支持。

## 使用方法：

```
git clone https://github.com/miqt/WandFix.git
```

添加依赖：

```
compile project(':wand')
annotationProcessor project(':wand-compiler')
```

代码调用：


```java
public class MainActivity extends AppCompatActivity implements Wand.MotorListener {

    @InjectObject(
            //指向类的全名
            value = "com.miqt.demo.presenter.AppPresenterImpl",
            //设置双亲委托
            //项目开发中建议使用PROJECT，优先应用本地类库。
            //项目发布时，应修改为NEVER，优先应用热修复包中的类库。
            level = ParentalEntrustmentLevel.NEVER)
    AppPresenter ap;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //可以在application中初始化一次即可
        Wand.get().init(this).listener(this);
    }

    public void getStr(View view) {
        //使用注解注入对象必须要调用这一行
        ClassInstall.inject(this);
        //或者带参数的构造方法
        //Map<String, Object[]> pramHouse=new HashMap<>();
        //pramHouse.put("com.miqt.demo.presenter.AppPresenterImpl",new Object[]{"hello"});
        //ClassInstall.inject(this,pramHouse);

        //也可以不使用注解注入的方式初始化对象
        //ap= ObjectFactory.make("com.miqt.demo.presenter.AppPresenterImpl"/*,构造参数*/);
        //ap= ObjectFactory.make(AppPresenterImpl.class/*,构造参数*/);

        String str = ap.getStr();
        Toast.makeText(this, str, Toast.LENGTH_SHORT).show();
    }
}
```

相关注解用法及作用说明：

1. @InjectObject 添加到成员变量上，对该成员变量注入对象
2. @AddToFixPatch 添加到类上，无需参数，表示该类是热修复相关的类，在编译期生成打包脚本的时候会将打了这个注释的类添加到打包列表中，如果有@InjectObject注解指定过的类，也可以不加。

## Other

[如何创建并应用热修复包](https://github.com/miqt/WandFix/wiki/%E5%A6%82%E4%BD%95%E5%88%9B%E5%BB%BA%E5%B9%B6%E5%BA%94%E7%94%A8%E7%83%AD%E4%BF%AE%E5%A4%8D%E5%8C%85)

具体更多用法请移步[Wiki](https://github.com/miqt/WandFix/wiki)

> 欢迎提出问题和宝贵意见。如果您觉得这个项目还不错，就点个star吧(￣▽￣)~*

