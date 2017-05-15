---
title: OkHttp 更新 Token 的解决方案
date: 2016-12-01 20:10:07
tags:
categories: Android
---

# 前言
一般应用在用户登录以后，会保存 token 的相关信息，以便进行接下来的相关请求，这里就涉及到一个 token 有效期的问题。每次打开应用的时候，本地判断 token 是否有效，无效则需要重新获取新的 token。但是本地验证也会存在一些问题：
1. 用户手机时间被调整
2. 进入应用一段时间以后，token 失效

因此需要根据请求返回进行 token 的有效期判断，下面提供两种解决方案。

# Authenticator

利用 OkHttp 提供的 [Authnenticator](https://github.com/square/okhttp/wiki/Recipes#handling-authentication) 接口，OkHttp 会在返回为 `401` 的情况下重新进行请求，但是请注意，只有 HTTP 返回的状态码为 `401` 的时候，才会调用该接口。如果后台接口设计合乎规范，可以使用该方法。

```java
public class TokenAuthenticator implements Authenticator {
    @Override
    public Request authenticate(Route route, Response response) throws IOException {
      
       	// 去获取新的 token，采用同步请求方式
        String newToken = service.refreshToken.execute().body();

       // 保存新的 token
	 Config.saveToken(newToken);

        return response.request().newBuilder()
                .addHeader("Authorization",newToken)
                .build();
    }
}
```

当然，不是所有请求都把 token 放在 `Header` 里面，比如放在 `Query` 参数里面，那么我们可以这样做：
```java
public class TokenAuthenticator implements Authenticator {
    @Override
    public Request authenticate(Route route, Response response) throws IOException {
      
        // 去获取新的 token，采用同步请求方式
        String newToken = call.execute().body();

        // 保存新的 token
	Config.saveToken(newToken);

        Request originalRequest = response.request();
       
        HttpUrl url = originalRequest.url()
                .newBuilder()
                .setQueryParameter("Token", newToken)
                .build();

        return response.request().newBuilder()
                .url(url)
                .build();

    }
}
```
关键点
	`Request originalRequest = response.request();`
	`HttpUrl url = originalRequest.url();`

如果 token 放在 `FormBody` 里面，可以通过 `originalRequest.body()` 进行操作
如果通过 `JSON` 的方式，暂时没有找到解决的方法。
最后我们给 OkHttp 设置 `Authenticator`

```java
 	OkHttpClient.Builder builder = new OkHttpClient.Builder();
 	builder.authenticator(new TokenAuthenticator());
	 ...
```
但是，并不是所有接口都这样设计，如果没有返回 `401` 的状态码，而是返回诸如以下这种类型数据：
```json
{
	"message":"UNAUTHORIZED",
	"code":401
}
```

如果是这样的自定义数据格式的，那么使用 Authenticator 这样的方式便不行。

# Interceptor

使用 OkHttp 的拦截器 [Intercaptor](https://github.com/square/okhttp/wiki/Interceptors)

```java
public class TokenInterceptor implements Interceptor {

    public static final Charset UTF_8 = Charset.forName("UTF-8");

    @Override
    public Response intercept(Chain chain) throws IOException {

        Request originalRequest = chain.request();
        Response originalResponse = chain.proceed(originalRequest);

		// 获取返回的数据字符串
        ResponseBody responseBody = originalResponse.body();
        BufferedSource source = originalResponse.body().source();
        source.request(Integer.MAX_VALUE);
        Buffer buffer = source.buffer();
        Charset charset = UTF_8;
        MediaType contentType = responseBody.contentType();
        if (contentType != null) {
            charset = contentType.charset();
        }
        String bodyString = buffer.clone().readString(charset);
		
	// 如果 token 已经过期
        if (token is expired) {
            String newToken = service.refreshToken.execute().body();

            // 保存新的 token
	    Config.saveToken(newToken);

	   // 添加到 Query 参数
            HttpUrl url = chain.request().url()
                    .newBuilder()
                    .setQueryParameter("Token", newToken)
                    .build();

            Request newRequest = chain.request().newBuilder()
                    .url(url)
                    .build();

		// 添加到 Header
             Request newRequest = originalRequest.newBuilder()
             		.header("Authorization",newToken)
             		.build();

            originalResponse.body().close();
            return chain.proceed(newRequest);

        }

        return originalResponse;
    }
```
关键点
 	`Request originalRequest = chain.request();`
	`Response originalResponse = chain.proceed(originalRequest);`

注意，按照文档关于 [ResponseBody](https://square.github.io/okhttp/3.x/okhttp/okhttp3/ResponseBody.html) 的说明
> ** The response body can be consumed only once. **

> This class may be used to stream very large responses. For example, it is possible to use this class to read a response that is larger than the entire memory allocated to the current process. It can even stream a response larger than the total storage on the current device, which is a common requirement for video streaming applications.

> Because this class does not buffer the full response in memory, the application may not re-read the bytes of the response. Use this one shot to read the entire response into memory with bytes() or string(). Or stream the response with either source(), byteStream(), or charStream().

请求返回的数据有可能过大，所以没有将所有数据缓存在内存里面，应用不能再去读取这些数据  

所以在处理返回的数据的时候，可以使用上面代码里面的方法处理返回的数据（也有其他的方法），也可以为了省事，使用 `response.Body().string()` ,但是切记最后返回要重新请求
`return chain.procees(resquest)`

```java
		Request originalRequest = chain.request();
        Response originalResponse = chain.proceed(originalRequest);

        // 获取返回的数据字符串
        ResponseBody responseBody = originalResponse.body();

        // do something

        return chain.proceed(request);
```

同样的，为 OkHttp 设置 `Interceptor`
```java
	OkHttpClient.Builder builder = new OkHttpClient.Builder();
	builder.addInterceptor(new TokenInterceptor());
	...
```

# 总结
- `	Authenticator` 适用于状态码返回 `401` 的情况
- `Interceptor` 使用范围更广，不仅可以处理 token 失效的多种情况，同时可以为网络请求设置拦截器，实现自己想要的功能
# 参考链接
- [OkHttp Authnenticator](https://github.com/square/okhttp/wiki/Recipes#handling-authentication)
- [Okhttp Intercaptor](https://github.com/square/okhttp/wiki/Interceptors)
- [OkHttp ResponseBody](https://square.github.io/okhttp/3.x/okhttp/okhttp3/ResponseBody.html)
