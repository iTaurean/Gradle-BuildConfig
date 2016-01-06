
## 应用场景
通常情况下我们的apps发布后也就是release模式下log是不显示的，debug模式下是显示log的，但是在特殊情况下我们测试release包的时候需要log的时候，就无法使用BuildConfig.DEBUG来达到要求，因为在release模式下自动设置为false，debug模式下是true，这个时候我们需要自定义可控制的log开关。

## Android Studio 对应的BuildConfig.java位置
在Studio中生成的目录： /app/build/generated/source/buildConfig/ 文件下的产品目录里面，找到想要的包名下会自动生成BuildConfig.java文件。我们可以看看下release模式下该文件的内容：

```
public final class BuildConfig {
  public static final boolean DEBUG = Boolean.parseBoolean("true");
  public static final String APPLICATION_ID = "com.hiifit.health";
  public static final String BUILD_TYPE = "debug";
  public static final String FLAVOR = "_360";
  public static final int VERSION_CODE = 36;
  public static final String VERSION_NAME = "2.3.3";
  // Fields from build type: debug
  public static final boolean MY_DEBUG = false;
}
```

## 自定义BuildConfig字段
在我们的build.gradle里面加入如下代码：

```
buildTypes {
        release {
            // 不显示Log, 在java代码中的调用方式为：BuildConfig.MY_DEBUG
            buildConfigField "boolean", "MY_DEBUG", "false"
 
            minifyEnabled true
            zipAlignEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
 
            signingConfig signingConfigs.release
        }
 
        debug {
            // 显示Log
            buildConfigField "boolean", "MY_DEBUG", "true"
 
            versionNameSuffix "-debug"
            minifyEnabled false
            zipAlignEnabled false
            shrinkResources false
            signingConfig signingConfigs.debug
        }
    }

```

*语法为：*

```
buildConfigField "boolean", "MY_DEBUG", "true"
```

上述语法就定义了一个*boolean*类型的*MY_DEBUG*字段，值为*true*，之后我们就可以在程序中使用*BuildConfig.MY_DEBUG*字段来判断我们所处的api环境。例如:

```
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
 
        setContentView(R.layout.activity_main);
 
        CommonUtils.getVersionName(this);
 
        initViews();
 
        if(BuildConfig.MY_DEBUG) {
            Log.i("TAG", "MainActivity.onCreate()");
        }
    }

```

# Issue

但是如果有库工程的，Android Studio中gradle配置的库library只能使用release参数。所以在库library下的gradle中使用buildConfigField来配置，是不起作用的。这是一个已知的bug，还没被修复。相关链接：
- [stackoverflow](http://stackoverflow.com/questions/20176284/buildconfig-debug-always-false-when-building-library-projects-with-gradle)
- [Issue](https://code.google.com/p/android/issues/detail?id=52962)
