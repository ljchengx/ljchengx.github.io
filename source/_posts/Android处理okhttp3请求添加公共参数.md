---
title: Android处理okhttp3请求添加公共参数
date: 2020-03-28 10:07:37
tags:
- 公共参数
- 拦截器
categories:
- Android
---


#### 1.问题描述：

由于前期项目没有增加移动端接口相关埋点信息，于是现在需要在调用相关的接口做公共参数的发送，服务端可以根据这部分固定的参数来增加埋点信息的处理。

#### 2.解决思路：

由于字段统一而且牵扯到的接口很多，如果一个一个处理的话工作量太大，而且后面如果新增接口还需要添加固定的这几个参数，如果牵扯修改的话，会更加麻烦。于是考虑okhttp3的拦截器去统一处理。

#### 3.遇到的问题：

开始处理的方式是获取url通过addQueryParameter方式来添加上，然后发现效果不是想要的，这样的方式会把参数直接加到url后面，而不是我们希望的加到已有参数的后面。于是继续探究。

#### 4.解决方式：

```java
/**
 * @ProjectName: trunk
 * @ClassName: urlInterceptor
 * @Description: okhttp3拦截器新增公共参数
 * @Author: ljchengx
 * @CreateDate: 2020/3/26 17:41
 */
public class BodyInterceptor implements Interceptor {
    @Override
    public Response intercept(Chain chain) throws IOException {

        Request request = chain.request();

        Request.Builder requestBuilder = request.newBuilder();

        if (request.body() instanceof FormBody) {
            FormBody.Builder newFormBody = new FormBody.Builder();
            FormBody oldFormBody = (FormBody) request.body();
            for (int i = 0; i < oldFormBody.size(); i++) {
                newFormBody.addEncoded(oldFormBody.encodedName(i), oldFormBody.encodedValue(i));
            }
            newFormBody.add("equipmentModel", Build.MODEL);
            newFormBody.add("equipmentApiVersion", Build.VERSION.SDK_INT+"");
            requestBuilder.method(request.method(), newFormBody.build());
        }
        Request newRequest = requestBuilder.build();
        return chain.proceed(newRequest);
    }

}
```



#### 5.参考链接：

- 