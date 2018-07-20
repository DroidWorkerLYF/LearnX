# Retrofit
代码基于[基于2.4.0版本](http://square.github.io/retrofit/)

结合官方的代码实例走一遍

```
public interface GitHubService {
  @GET("users/{user}/repos")
  Call<List<Repo>> listRepos(@Path("user") String user);
}
```
首先是创建一个GithubService，`Refrofit`使用接口的形式来定义api。  
在源码的http package下，定义了许多注解，包括@Get,@Post等。

```
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com/")
    .build();
```
接着创建一个Retrofit的实例。  
这里是使用的Builder模式来构建一个Retrofit实例，`Retrofit.Builder`的构造函数其实需要一个`Platform`参数。

```
private static Platform findPlatform() {
	try {
		Class.forName("android.os.Build");
		if(Build.VERSION.SDK_INT != 0) {
			return new Android();
		}
	} catch(ClassNotFoundException ignored) {
	}
	try {
		Class.forName("java.util.Optional");
		return new Java8();
	} catch(ClassNotFoundException ignored) {
	}
	return new Platform();
}
```
根据是Android平台还是Java平台，返回了对应的`Platform`，这两个对象都是`Platform`的子类，Override了部分方法。`Android Platform`这边提供了默认的Callback executor(主线程执行)和默认的CallAdapterFactory。不过这都可以在builder中自行替换。

```
	public Retrofit build() {
      if (baseUrl == null) {
        throw new IllegalStateException("Base URL required.");
      }

      okhttp3.Call.Factory callFactory = this.callFactory;
      if (callFactory == null) {
        callFactory = new OkHttpClient();
      }

      Executor callbackExecutor = this.callbackExecutor;
      if (callbackExecutor == null) {
        callbackExecutor = platform.defaultCallbackExecutor();
      }

      // Make a defensive copy of the adapters and add the default Call adapter.
      List<CallAdapter.Factory> callAdapterFactories = new ArrayList<>(this.callAdapterFactories);
      callAdapterFactories.add(platform.defaultCallAdapterFactory(callbackExecutor));

      // Make a defensive copy of the converters.
      List<Converter.Factory> converterFactories =
          new ArrayList<>(1 + this.converterFactories.size());

      // Add the built-in converter factory first. This prevents overriding its behavior but also
      // ensures correct behavior when using converters that consume all types.
      converterFactories.add(new BuiltInConverters());
      converterFactories.addAll(this.converterFactories);

      return new Retrofit(callFactory, baseUrl, unmodifiableList(converterFactories),
          unmodifiableList(callAdapterFactories), callbackExecutor, validateEagerly);
    }
``` 

接下来创建GitHubService的实例。`create`方式用到了动态代理。
```
GitHubService service = retrofit.create(GitHubService.class);
```

```
 public <T> T create(final Class<T> service) {
    Utils.validateServiceInterface(service);
    if (validateEagerly) {
      eagerlyValidateMethods(service);
    }
    return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
        new InvocationHandler() {
          private final Platform platform = Platform.get();

          @Override public Object invoke(Object proxy, Method method, @Nullable Object[] args)
              throws Throwable {
            // If the method is a method from Object then defer to normal invocation.
            if (method.getDeclaringClass() == Object.class) {
              return method.invoke(this, args);
            }
            // 根据前面Android Platform的定义这里默认是false
            if (platform.isDefaultMethod(method)) {
              return platform.invokeDefaultMethod(method, service, proxy, args);
            }
            ServiceMethod<Object, Object> serviceMethod =
                (ServiceMethod<Object, Object>) loadServiceMethod(method);
            // retrofit使用的是okhttp，OKHttpCall实现了Call接口
            OkHttpCall<Object> okHttpCall = new OkHttpCall<>(serviceMethod, args);
            return serviceMethod.adapt(okHttpCall);
          }
        });
  }
```
在`loadServiceMethod`方法中，为每个method根据注解创建了一个`ServiceMethod`对象。

```
	public ServiceMethod build() {
      callAdapter = createCallAdapter();
      responseType = callAdapter.responseType();
      if (responseType == Response.class || responseType == okhttp3.Response.class) {
        throw methodError("'"
            + Utils.getRawType(responseType).getName()
            + "' is not a valid response body type. Did you mean ResponseBody?");
      }
      responseConverter = createResponseConverter();
      ......省略
    }
```
这里就负责创建的call adapter(比如我们外面指定了`RxJava2CallAdapterFactory`)和converter。`CallAdapter`是`Call`的代理，也就是前面的`OKHttpCall`。

```
Call<List<Repo>> repos = service.listRepos("octocat")
```
最后这里可以同步或者异步的去请求网络。就是通过动态代理调用到了实际的执行方法。