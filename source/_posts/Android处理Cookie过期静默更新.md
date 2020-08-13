---
title: Android处理Cookie过期静默更新
date: 2020-03-21 15:44:34
tags:
- Android
- Cookie
- 拦截器
categories:
- Android
---

#### 1.问题描述：

​	项目中在请求接口时在header上带上Cookie信息 一直没有处理过期时间，最近需要在原来的基础上添加请求Cookie过期后静默更新的功能。目前使用的是retrofit2+okhttp3+rxjava2的网络框架。



#### 2.解决思路：

​	因为添加Cookie使用的是Okhttp3的拦截器处理的方式 需要准备在之前的Interceptor上面改造，其实处理起来也不是很复杂，需要确定的思路是在请求是做统一拦截，根据和服务端约定好的规则来判断是否处理Cookie过期，如果过期之前需要在同步请求一个新的接口拿到最新的Cookie信息放到原来的请求上，重新完成请求。

#### 3.遇到的问题：

需要和服务端协调好规则，如何判断失效，同时需要注意要重写一个新的请求

#### 4.解决方式：

``` java
public class AddCookiesInterceptor implements Interceptor {

    private Handler mHandler = new Handler();
    private String resultStr;

    private Context context;

    public AddCookiesInterceptor(Context context) {
        super();
        this.context = context;

    }

    @Override
    public Response intercept(Chain chain) throws IOException {

        Request.Builder builder = chain.request().newBuilder();
        //这里使用本地存的cookie来添加到每个请求的header 方式有很多  如本地数据库保存 这里举一个例子
        String cookie = CookieSp.get();

        if (!TextUtils.isEmpty(cookie)) {
            builder.addHeader("Cookie", cookie);
        }
        Request request = builder.build();
        Response response = chain.proceed(request);

        //判断是否Cookie过期
        if (isCookieExpired(response)) {

            KLog.e("AddCookiesInterceptor", "无感刷新Cookie,然后重新请求数据");

            //同步请求方式，获取最新的Cookie
            String newCookie = getNewCookie();
            KLog.e("AddCookiesInterceptor", newCookie+"新Cookie");

            //使用新的Cookie，创建新的请求
            if (null != newCookie && newCookie.length() > 0) {
                Request newRequest = chain.request()
                        .newBuilder()
                        .header("Cookie", newCookie)
                        .build();
                //重新请求上次的接口
                return chain.proceed(newRequest.newBuilder().build());
            }
        }
        return chain.proceed(request);
    }

    /**
     *  获取新的Cookie
     */
    private String getNewCookie() {

        //根据服务端约定好的新接口 获取新的Cookie
        String url = HttpConstant.BASE_URL + "/api/login/newlogin?userName=123&pwd=123";

        KLog.e("AddCookiesInterceptor", "重新请求---" + url);

        //新建立一个请求
        HttpsUtils.SSLParams sslParams = HttpsUtils.getSslSocketFactory();
        OkHttpClient client =  new OkHttpClient.Builder()
                .hostnameVerifier(HttpsUtils.UnSafeHostnameVerifier)
                .sslSocketFactory(sslParams.sSLSocketFactory, sslParams.trustManager)
                .connectionPool(new ConnectionPool(8, 10, TimeUnit.SECONDS))
                .build();
        Request request = new Request.Builder().url(url).build();
        okhttp3.Call call = client.newCall(request);
        StringBuffer cookie = new StringBuffer();

        try {
            Response response = call.execute();
            RequestCode data = new Gson().fromJson(response.body().string(), RequestCode.class);

            //这里根据约定好的规则获取新的Cookie 这里是根据头部信息拿到新的Cookie 保存在本地
            if (data.getCode() == 1) {
                if (!response.headers("Set-Cookie").isEmpty()) {
                    List<String> list = response.headers("Set-Cookie");
                    for (String str :
                            list) {
                        cookie.append(str).append(";");
                    }
                    CookieSp.put(cookie.toString());
                }

            } else {
                KLog.e("AddCookiesInterceptor", "新Cookie获取失败---退出登录" );
                AppManager.getAppManager().finishAllActivity();
                //这里处理你的退出重新登录的逻辑
            }
        } catch (IOException e) {
            KLog.e("AddCookiesInterceptor",e.toString());
            e.printStackTrace();
        }
        return cookie.toString();
    }


    /**
     *  根据返回值判断Cookie是否过期
     */
    private boolean isCookieExpired(Response response) {
        try {
            resultStr = response.body().string();
            RequestCode requestCode = new Gson().fromJson(resultStr, RequestCode.class);
            KLog.e("AddCookiesInterceptor", requestCode.getCode() + "----requestCode");
            //根据服务端约定的规则来判断是否过期
            if (requestCode.getCode() == -300) {
                KLog.e("AddCookiesInterceptor", "----requestCode,Cookie登录过期了");
                return true;
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return false;
    }
}
```



#### 5.参考链接：

- https://www.jianshu.com/p/66b59ad1fdc1