---
date: 2017-06-17
layout: 'post'
tags:
    - "retrofit"
    - 'java'
status: 'public'
---

# Retrofit中的用户认证（Authentications）

## 1. Basic Authentication
用 Basic Authentication 方式认证只需要直接告诉服务器用户名和密码就可以了，这个信息应该被放在请求头部的Authentication字段中，并且通过base64编码(user:password)

Retrofit中可以通过`Credentials`来得到认证码:

```java
String auth = Credentials.basic(username, password);
```

在请求头中添加这个请求码，可以如下定义：

```
@GET("userinfo")
Call<UserInfo> getUserInfo(@Header("Authorization") String auth)
```

这样，在每次请求的时候都必须提供这个认证码，如果传入的认证码为空，则忽略。
然而使用这种方式，不得不在所有需要认证的请求定义中加入这个参数，使用起来不太方便。不过也有一劳永逸的方法，就是使用`Interceptor`，它是`okhttp`中提供的一种对请求进行再包装的机制：

> Interceptors are a powerful mechanism that can monitor, rewrite, and retry calls. Here's a simple interceptor that logs the outgoing request and the incoming response.

可以如下使用`Interceptor`来添加认证码：

```java
class AuthenticationInterceptor implements Interceptor {

    private String auth;

    public void setAuth(String auth) {
        this.auth = auth;
    }
    
    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();
        if (auth != null) {
            Request.Builder builder = request.newBuilder()
                    .header("Authorization", auth);
            request = builder.build();
        }
        return chain.proceed(request);
    }
}
```
然后可以像下面这样，将这个`Interceptor`配置到`retrofit`中:

```
AuthenticationInterceptor interceptor = new AuthenticationInterceptor();
retrofit = new Retrofit.Builder()
                .baseUrl("....")
                .client(new OkHttpClient.Builder().addInterceptor(interceptor).build())
                .build();
```
之后就可以通过这个`interceptor`来管理你的认证信息了。当然，实际使用的时候，`interceptor`内部的逻辑会更复杂一些。

## 2. Digest Authentication

`Digest Authentication`比`Basic Authentication`要稍微麻烦一些。客户端初次访问服务器时，并不携带密码。此时服务器会返回一个随机生成的字符串（nonce）。然后客户端再将这个字符串与密码结合在一起进行MD5编码，将编码以后的结果发送给服务器作为验证信息。这样，每次认证码都不一样，可以避免遭到`Replay Attack`。

然而，`okhttp`并不能直接支持这种认证模式。如果需要使用这种认证，可以参考[okhttp-digest](https://github.com/rburgst/okhttp-digest)。

