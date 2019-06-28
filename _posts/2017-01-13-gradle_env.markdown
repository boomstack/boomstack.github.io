---

layout: post 

title: "使用Gradle区分开发环境" 

date: 2017-02-22 11:55:21 +0800 

categories: Android

---

场景：公司服务器与app都会有test、beta、online等环境区分，这些可以在代码里手动修改，然后重新build新的apk文件，而且一个手机上只能装一个app，如果切换环境需要覆盖安装。本博客主要是使用build.gradle文件动态修改开发环境配置，核心是使用BuildConfig这个编译过程生的文件。
## buildType
gradle中可以指定编译类型，在build.gradle中可以配置buildType这个扩展，代码如下：
```groovy
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            buildConfigField("String", "API_TYPE", "\"RELEASE\"")
        }

        beta {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            applicationIdSuffix ".beta"
            buildConfigField("String", "API_TYPE", "\"BETA\"")
        }

        debug {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            applicationIdSuffix ".debug"
            buildConfigField("String", "API_TYPE", "\"DEBUG\"")
        }
    }
```
applicationIdSuffix是applicationId追加的后缀，目的是为了在同一台手机上安装不同环境的apk，applicationid是Android studio上对packagename的扩展，默认的applicationid等于packagename，对于一些三方sdk也是读取的applicationid，所以，加了applicationIdSuffix之后，某些sdk是不生效的，比如百度地图。更严重的，如果某些服务需要读取这个字段，比如海外项目要使用Google服务，将直接导致编译不通过，还需要手动修改Google服务的配合文件，这时候就不建议再使用这种方式了，弊大于利。  

buildConfigField是在BuildConfig文件里添加的字段，区分环境就可以读取API_TYPE这个字段。  

## productFlavors
字面意思可以猜测下意思，为产品添加风味，产品特点，它是依赖于buildType的，对每一种buildType进行设置。多渠道打包就可以通过这里设置，不再赘述，假如app需要免费版和专业版，也可以在这里设置，修改applicationId即可，便是两个app了。
```groovy
 productFlavors {
        free {
            //to set some values here.
        }
        pro {
            //to set some values here.
        }
    }
```
注意，要是同时添加了buildType和productFlavors，命令行打包耗时会增加，Android studio会编译m*n次，（m是buildType种类数，n是productFlavors种类数）如图：

![这里写图片描述](https://github.com/boomstack/boomstack.github.io/blob/master/assets/all/asdfasdf32432.png?raw=true)

要是只需要某个buildType和某个productFlavors就点击Build variants即可。

![这里写图片描述](https://github.com/boomstack/boomstack.github.io/blob/master/assets/all/sdfaewrew234.png?raw=true)  
这张图是只用buildType的variant，点击相应的项，待gradle同步完成后，BuildConfig这个文件就会做出相应的修改
![这里写图片描述](https://github.com/boomstack/boomstack.github.io/blob/master/assets/all/tret435212.png?raw=true)  
这张图是加入productFlavors之后的，可以看到，variant进行了组合。
以下是选择了freeBeta之后的BuildConfig文件：
```java
public final class BuildConfig {
  public static final boolean DEBUG = false;
  public static final String APPLICATION_ID = "com.boomstack.testvolley.beta";
  public static final String BUILD_TYPE = "beta";
  public static final String FLAVOR = "free";
  public static final int VERSION_CODE = 1;
  public static final String VERSION_NAME = "1.0";
  // Fields from build type: beta
  public static final String API_TYPE = "BETA";
}
```
然后在项目中需要区分环境的地方使用这里的一些字段就可以了。