---

layout: post 

title: "RxJava+Retrofit+MVP实践" 

date: 2017-07-10 15:09:22 +0800 

categories: Android

---

本篇文章会沿用MVP的思路，MVP模式可以看下之前的这篇文章：
http://blog.csdn.net/ethanhola/article/details/54589746
retrofit是对okhttp的又一层封装，从使用上来说可以使用okhttp的所有特性，也可以定制httpclient。
本文的网络请求来自gank.io, 要实现的功能就是使用rxjava处理网络请求的构造以及成功失败的返回处理。
### 1. mvp构建
```java
public interface ArticleContract {
    interface ArticleView {
        void showError(String msg);

        void onGetResult(ArticleBean.ArticleItemBean[] beans);
    }

    interface ArticlePresenter {
        void fetchArticles(String type);
    }

}

```
这里把view层和presenter层统一放在了一个接口里，这个接口更像是view和presenter的一个契约。
主要视图管理类：
```java
public class ArticleItemActivity extends AppCompatActivity implements ArticleContract.ArticleView
```
presenter实现类：
```java
public class ArticlePresenterImpl implements ArticleContract.ArticlePresenter
```
然后分别在v和p中初始化对象，这个可以具体看源码，源码地址在文章最后。
### 2. Retrofit的构建
首先是APIService的接口：
```java
public interface ApiService {
    String BASE_URL = "http://gank.io/api/data/";

    /**
     * 获取文章
     *
     * @param type 文章类型
     */
    @GET("{type}/10/1")
    Observable<ArticleBean> getArticles(@Path("type") String type);
}
```
为了使用rxjava，这里的返回数据得是Observable对象，单独使用retrofit默认返回的是Call对象，这里需要使用retrofit的一个扩展包转换一下转化代码在下一步的retrofit构建中：
```java
compile 'com.squareup.retrofit2:adapter-rxjava:2.0.0'
```
retrofit构建：
```java
 //拦截器,查看请求与返回
        HttpLoggingInterceptor interceptor = new HttpLoggingInterceptor();
        interceptor.setLevel(HttpLoggingInterceptor.Level.BODY);
        OkHttpClient client = new OkHttpClient.Builder().addInterceptor(interceptor).build();
        //构建apiService
        apiService = new Retrofit.Builder()
                .addConverterFactory(GsonConverterFactory.create())
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                .baseUrl(ApiService.BASE_URL)
                .client(client)
                .build()
                .create(ApiService.class);
```
上边加了一个拦截器，可以使用ide的monitor查看请求与返回的数据，也可以使用adb指令查看，过滤关键字为OkHttp:
```shell
adb logcat | grep OkHttp
```
这一步的构建中，addCallAdapterFactory方法就是上一步的Call转为Observable，使用扩展包的默认转化就可以了，另外addConverterFactory用于Json和Object的转化，使用默认Gson即可，也是扩展包的类，不过这两个都可以自己定制。
```groovy
compile 'com.squareup.retrofit2:converter-gson:2.0.0'
```
### 3. rxjava处理请求与返回
这个当然要在presenter层处理
```java
        apiService.getArticles(type)
                //io线程执行网络请求
                .subscribeOn(Schedulers.io())
                //返回数据在UI线程
                .observeOn(AndroidSchedulers.mainThread())
                //返回error过滤处理
                .filter(new Func1<ArticleBean, Boolean>() {
                    @Override
                    public Boolean call(ArticleBean articleBean) {
                        if (articleBean.error) {
                            articleView.showError("error");
                            return false;
                        }
                        if (articleBean.results != null) {
                            return true;
                        }
                        throw new NullPointerException("result is null");
                    }
                })
                //返回成功
                .subscribe(new Action1<ArticleBean>() {
                    @Override
                    public void call(ArticleBean articleBean) {
                        articleView.onGetResult(articleBean.results);
                    }
                });
```
返回结果使用了filter操作符进行过滤，可以根据自己的业务逻辑选择是否返回true，即是否继续向订阅者发消息（返回错误就没必要再发送了，可以直接响应v层的错误）。
这种代码写下来之后感觉逻辑很清晰，一步一步的消息传递都能看得到，这也是rxjava的好处吧。
不足之处请多多指教
本文源码：https://github.com/boomstack/LabAndroidLearn
提交节点为20170422 -- rxjava retrofit mvp
代码目录为：
app/src/main/java/com/boomstack/labandroidlearn/tech/mvp_rxjava_retrofit/