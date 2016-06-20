title:Android 网络请求：Retrofit 使用
---

# Android 网络请求：Retrofit 使用

[网络请求：retrofit+okhttp3](https://github.com/square/retrofit)

示例项目：

[有妹子的日报 MeiZiNews ](https://github.com/qq137712630/MeiZiNews)

总结：结合 RxJava 使用得十分的爽。如果没有用到 RxJava，个人感觉这样使用和用 OkHttp3 的封装差不多

---
##配置

[Retrofit 2.0 + OkHttp 3.0 配置](https://drakeet.me/retrofit-2-0-okhttp-3-0-config)

---

##转换器

[retrofit](http://square.github.io/retrofit/#restadapter-configuration)

[RxAndroid+Retrofit环境搭建](http://www.07net01.com/program/2016/02/1280821.html)

转换器可以被添加到支持其他类型。六兄弟模块适应流行的序列化库为您提供方便。
Converters can be added to support other types. Six sibling modules adapt popular serialization libraries for your convenience.

	Gson: com.squareup.retrofit2:converter-gson
	Jackson: com.squareup.retrofit2:converter-jackson
	Moshi: com.squareup.retrofit2:converter-moshi
	Protobuf: com.squareup.retrofit2:converter-protobuf
	Wire: com.squareup.retrofit2:converter-wire
	Simple XML: com.squareup.retrofit2:converter-simplexml
	Scalars (primitives, boxed, and String): com.squareup.retrofit2:converter-scalars

直接返回String类型需引入：ScalarsConverterFactory.create()


```Java

	retrofit = new Retrofit.Builder()
			.client(MyOkHttpClient.getMyOkHttpClient().getOkHttpClient())//设置不同的底层网络库
			.baseUrl(strBaseUrl)
			.addConverterFactory(ScalarsConverterFactory.create())//添加 String类型[ Scalars (primitives, boxed, and String)] 转换器
			.addCallAdapterFactory(RxJavaCallAdapterFactory.create())//添加 RxJava 适配器
			.build();

```
			
返回Gson类型需引入：GsonConverterFactory.create()

```Java

	retrofit = new Retrofit.Builder()
			.client(MyOkHttpClient.getMyOkHttpClient().getOkHttpClient())//设置不同的底层网络库
			.baseUrl(strBaseUrl)
			.addConverterFactory(GsonConverterFactory.create())//添加 json 转换器
			.addCallAdapterFactory(RxJavaCallAdapterFactory.create())//添加 RxJava 适配器
			.build();

```

---

##Retrofit 结合 RxJava使用

[Retrofit 结合 RxJava使用](http://blog.csdn.net/ttccaaa/article/details/50659234)

 1. 添加依赖
		
		compile 'com.squareup.retrofit2:retrofit:2.0.0-beta4'
		compile 'com.squareup.retrofit2:converter-gson:2.0.0-beta4'
		
		compile 'com.squareup.retrofit2:adapter-rxjava:2.0.0-beta4'
		
		compile 'io.reactivex:rxandroid:1.1.0'
		compile 'io.reactivex:rxjava:1.1.0'
		
	前两个是 Retrofit 和 Gson 的依赖，第三个是 Retrofit 中 RxJava 转换器的依赖，最后两个就是 RxJava 和 Rx Android 的依赖

 2. 编写 API

	使用 RxJava 的情况下，接口文件稍有修改，接口中的方法的返回类型不再是 Call 而是 Observable 类型：


```Java		

		public interface GitHub {
		
		    @GET("/repos/{owner}/{repo}/contributors")
		    Call<List<Contributor>> contributors(@Path("owner") String owner,@Path("repo") String repo);
		
		    //使用 RxJava 的方法,返回一个 Observable
		    @GET("/repos/{owner}/{repo}/contributors")
		    Observable<List<Contributor>> RxContributors(@Path("owner") String owner,@Path("repo") String repo);
		}

```

	结合 RxJava 使用的 接口就定义好了，模型类不需要变动，接下来直接进行网络请求

 3. 网络请求

	使用 RxJava 的 Retrofit 可以直接在 主线程中编写。	

```Java

		@Override
		protected void onCreate(Bundle savedInstanceState) {
		    super.onCreate(savedInstanceState);
		    setContentView(R.layout.activity_main);
		
		    Retrofit retrofit=new Retrofit.Builder()
		            .baseUrl("https://api.github.com")
		            .addConverterFactory(GsonConverterFactory.create())//添加 json 转换器
		            .addCallAdapterFactory(RxJavaCallAdapterFactory.create())//添加 RxJava 适配器
		            .build();
		
		    GitHub gitHub=retrofit.create(GitHub.class);
		    gitHub.RxContributors("square","retrofit")
		            .subscribeOn(Schedulers.io())
		            .observeOn(AndroidSchedulers.mainThread())
		            .subscribe(new Subscriber<List<Contributor>>() {
		                @Override
		                public void onCompleted() {
		                    Log.i("TAG","onCompleted");
		                }
		
		                @Override
		                public void onError(Throwable e) {
		
		                }
		
		                @Override
		                public void onNext(List<Contributor> contributors) {
		                   for (Contributor c:contributors){
		                       Log.i("TAG","RxJava-->"+c.getLogin()+"  "+c.getId()+"  "+c.getContributions());
		                   }
		                }
		            });
		}

```

---

##使用Retrofit和Okhttp实现网络缓存。无网读缓存，有网根据过期时间重新请求

[okhttp 请求设置官方文档](https://github.com/square/okhttp/wiki/Recipes)

[使用Retrofit和Okhttp实现网络缓存。无网读缓存，有网根据过期时间重新请求](http://www.jianshu.com/p/9c3b4ea108a7#)

[OuNews 新闻：RetrofitManager类下的 initOkHttpClient 方法](https://github.com/oubowu/OuNews)

 okhttp3.X，retrofit:2.0.0-beta4适用

 1. 配置okhttp中的Cache
 

```Java

		OkHttpClient okHttpClient;
		File cacheFile = new File(DemoActivity.this.getCacheDir(), "[缓存目录]");
		Cache cache = new Cache(cacheFile, 1024 * 1024 * 10); //100Mb
		okHttpClient = new OkHttpClient.Builder()
		     .cache(cache)
		     .build();
		
```

	栗子：
	

```Java

		// 完成缓存
		public class MyOkHttpClient {
		
		    //设缓存有效期为两天
		    protected static final long CACHE_STALE_SEC = 60 * 60 * 24 * 2;
		    //查询缓存的Cache-Control设置，为if-only-cache时只查询缓存而不会请求服务器，max-stale可以配合设置缓存失效时间
		    protected static final String CACHE_CONTROL_CACHE = "only-if-cached, max-stale=" + CACHE_STALE_SEC;
		    //查询网络的Cache-Control设置，头部Cache-Control设为max-age=0时则不会使用缓存而请求服务器
		    protected static final String CACHE_CONTROL_NETWORK = "max-age=0";
		
		    private String TAG = "MyOkHttpClient";
		
		    private OkHttpClient okHttpClient;
		    private static MyOkHttpClient myOkHttpClient;
		
		    public static MyOkHttpClient getMyOkHttpClient() {
		        if (myOkHttpClient == null) {
		            myOkHttpClient = new MyOkHttpClient();
		
		        }
		
		        return myOkHttpClient;
		    }
		
		    /**
		     * 初始化
		     *
		     * @param mContext
		     */
		    public void init(final Context mContext) {
		
		        if (okHttpClient != null) {
		            return;
		        }
		
		
		        Log.e(TAG, "初始化okHttpClient");
		        // 因为BaseUrl不同所以这里Retrofit不为静态，但是OkHttpClient配置是一样的,静态创建一次即可
		        File cacheFile = new File(
		                mContext.getCacheDir(),
		                "HttpCache"
		        ); // 指定缓存路径
		        Cache cache = new Cache(cacheFile, 1024 * 1024 * 100); // 指定缓存大小100Mb
		        // 云端响应头拦截器，用来配置缓存策略
		        Interceptor rewriteCacheControlInterceptor = new Interceptor() {
		            @Override
		            public Response intercept(Chain chain) throws IOException {
		                Request request = chain.request();
		                if (!NetUtil.isConnected(mContext)) {
		                    request = request.newBuilder()
		                            .cacheControl(CacheControl.FORCE_CACHE).build();
		                    Log.e(TAG, "no network");
		                }
		                Response originalResponse = chain.proceed(request);
		                if (NetUtil.isConnected(mContext)) {
		                    //有网的时候读接口上的@Headers里的配置，你可以在这里进行统一的设置
		                    String cacheControl = request.cacheControl().toString();
		                    return originalResponse.newBuilder()
		                            .header("Cache-Control", cacheControl)
		                            .removeHeader("Pragma").build();
		                } else {
		                    return originalResponse.newBuilder().header("Cache-Control",
		                            "public, only-if-cached," + CACHE_STALE_SEC)
		                            .removeHeader("Pragma").build();
		                }
		            }
		        };
		        okHttpClient = new OkHttpClient.Builder().cache(cache)
		                .addNetworkInterceptor(rewriteCacheControlInterceptor)
		                .addInterceptor(rewriteCacheControlInterceptor)
		                .connectTimeout(30, TimeUnit.SECONDS).build();
		
		    }
		
		    public OkHttpClient getOkHttpClient() {
		        return okHttpClient;
		    }
		
		    /**
		     * 根据网络状况获取缓存的策略
		     *
		     * @return
		     */
		    @NonNull
		    public static String getCacheControl(Context mContext) {
		        return NetUtil.isConnected(mContext) ? CACHE_CONTROL_NETWORK : CACHE_CONTROL_CACHE;
		    }
		
		}

```

 2. 配置请求头中的cache-control
	

```Java

	    //使用 RxJava 的方法,返回一个 Observable
	    @Headers("Cache-Control: public, max-age=3600")
	    @GET("/repos/{owner}/{repo}/contributors")
	    Observable<List<Contributor>> RxContributors(@Path("owner") String owner,@Path("repo") String repo);

```

 3. 配置请求 Retrofit 设置不同的底层网络库


```Java

        Retrofit retrofit = new Retrofit.Builder()
                .client(okHttpClient)
                .baseUrl("https://api.github.com")
                .addConverterFactory(GsonConverterFactory.create())//添加 json 转换器
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())//添加 RxJava 适配器
                .build();

```

 4. 开启打印连接日志

```Java

	    /**
	     * 开启打印连接日志
	     * @return
	     */
	    private HttpLoggingInterceptor getHttpLoggingInterceptor() {
	        boolean isDebug= false;//是否开启
	        HttpLoggingInterceptor interceptor = new HttpLoggingInterceptor();
	
	        if(isDebug)
	        {
	            interceptor.setLevel(HttpLoggingInterceptor.Level.BODY);
	        }
	
			//开启打印连接日志
	        return interceptor;
	    }

		OkHttpClient.Builder().addInterceptor(getHttpLoggingInterceptor())

		eg:
		
        okHttpClient = new OkHttpClient.Builder()
                .addInterceptor(getHttpLoggingInterceptor())
                .cache(cache)
                .addNetworkInterceptor(rewriteCacheControlInterceptor)
                .addInterceptor(rewriteCacheControlInterceptor)
                .connectTimeout(30, TimeUnit.SECONDS).build();

```

---

## Retrofit 如何使用Observable.from发出多条连接的？

以自己的开源项目 [MeiZiNews](https://github.com/qq137712630/MeiZiNews) 中的 ````` MzituImgListModel ```` 为栗子，大概是这3步：

 1. 先向网站请求参数链表
 2. 使用 flatMap，它可以返回 Observable，所以在 flatMap 里使用 Observable.from
 3. 再使用 flatMap，在它里面发出单次网络请求连接的 Observable，这时在它返回的就是单次请求的结果了


```Java

    /**
     * Retrofit 使用Observable.from发出多条连接
     * @param context
     * @param listener
     * @return
     */
    @Override
    public Subscription loadWeb(final Context context, final OnModelListener<ImgItem> listener) {

        MyStringRetrofit.getMyStringRetrofit().init(context, MeiZiApi.MZITU_API);
        final MzituApi mzituApi = MyStringRetrofit.getMyStringRetrofit().getCreate(MzituApi.class);


        Observable observable = mzituApi.RxImgList(
                MyOkHttpClient.getCacheControl(context),
                imgId,
                page
        ).flatMap(
                new Func1<String, Observable<Integer>>() {

                    @Override
                    public Observable<Integer> call(String s) {
                        // 第一次请求先请求页数，从而获得要发的链接数

                        Elements mElements = JsoupUtil.getMzituImgPage(s);
                        Elements tempElements = mElements.select("span");
                        String size = tempElements.get(tempElements.size() - 2).text();
                        if (TextUtils.isEmpty(size)) {
                            return null;
                        }

                        ArrayList<Integer> numList = new ArrayList<Integer>();

                        for (int i = 1; i <= Integer.parseInt(size); i++) {
                            numList.add(i);
                        }
                        return Observable.from(numList);
                    }
                }
        ).flatMap(
                new Func1<Integer, Observable<String>>() {
                    @Override
                    public Observable<String> call(Integer integer) {
                    //   发出单次网络请求连接
                        return mzituApi.RxImgList(
                                MyOkHttpClient.getCacheControl(context),
                                imgId,
                                integer + ""
                        );
                    }
                }
        ).map(
                new Func1<String, ImgItem>() {
                    @Override
                    public ImgItem call(String s) {
                    //   处理每次请问的结果

                        ImgItem img = new ImgItem();
                        Elements mElements = JsoupUtil.getMzituImgItem(s);
                        Element tempElement = mElements.first();

                        img.setImgUrl(tempElement.select("img").attr("src"));
                        img.setUrl(tempElement.select("img").attr("src"));
                        img.setStory_title(tempElement.select("img").attr("alt"));
                        return img;
                    }
                }
        );

        return RxJavaUtil.rxIoAndMain(
                observable,
                new Subscriber<ImgItem>() {

                    @Override
                    public void onCompleted() {
                        listener.onCompleted();
                    }

                    @Override
                    public void onError(Throwable e) {
                        listener.onError(e.toString());
                        DebugUtil.debugLogErr(e, e.toString());
                    }

                    @Override
                    public void onNext(ImgItem imgItem) {
                        listener.onSuccess(imgItem);
                    }
                }
        );
    }

```

---

## 问题集

 - [Retrofit error URL query string must not have replace block](http://stackoverflow.com/questions/24610243/retrofit-error-url-query-string-must-not-have-replace-block)

---

## 其他笔记

@Path:路径

@Query:查询条件，如：     

	xx=yy

如果有多个查询条件可以使用: ` @QueryMap ` ;

---

最后，推荐下自己的开源项目：

[有妹子的日报:https://github.com/qq137712630/MeiZiNews](https://github.com/qq137712630/MeiZiNews)

[APK下载：http://www.wandoujia.com/apps/com.ms.meizinewsapplication](http://www.wandoujia.com/apps/com.ms.meizinewsapplication)

它使用了多个开源框架，并在 ReadMe 中都有标明其框架的使用在哪个方面，遇到了什么奇怪的问题，中文的哦，喜欢的， Star 一下。

---

### 我的公众号

![小成小店](/img/wx_er_wei_ma.jpg)