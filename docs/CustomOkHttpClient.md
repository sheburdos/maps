# Using custom OkHttpClient [ANDROID ONLY]

It is possible to inject instance of the OkHttpClient. It can be used to add custom interceptors to the client. Example use case - add StethoInterceptor for debugging network requests.

## Enabling OkHttpClientProvider

In order to work, enable use of OkHttpClientProvider. By default, it is disabled and mapbox will use own instance of OkHttpClient. 

```js
import MapboxGL from "@react-native-mapbox-gl/maps";

MapboxGL.setUseOkHttpClientProvider(true);
```

## Setting up custom OkHttpClient

Customized instance of the OkHttpClient should be initialized and set in android app sources code. 

#### **`MainApplication.java`** 
```java
	
/***/
	
+import com.facebook.react.modules.network.ReactCookieJarContainer;
+import okhttp3.OkHttpClient;
+import com.facebook.react.modules.network.OkHttpClientProvider;
+import com.facebook.react.modules.network.OkHttpClientFactory;
	
/***/

public class MainApplication extends Application implements ReactApplication {

	/***/

	@Override
	public void onCreate() {
		super.onCreate();
		SoLoader.init(this, /* native exopackage */ false);

	+	OkHttpClientProvider.setOkHttpClientFactory(new OkHttpClientFactory() {
	+	  @Override
	+	  public OkHttpClient createNewNetworkModuleClient() {
	+		OkHttpClient.Builder builder = new OkHttpClient.Builder()
	+			.addNetworkInterceptor(new MyCustomInterceptor());
	+		return builder.build();
	+	  }
	+	});
	}
}
```

## Limitations and drawbacks

- Android only
- Setting custom headers, using MapboxGL.addCustomHeader(...) recreates OkHttpClient. It will not work if UseOkHttpClientProvider set to TRUE. Insted you can inject your own custom headers interceptor when creating OkHttpClient instance

## Example: enabling network logging with Stetho

[Stetho](http://facebook.github.io/stetho/#integrations) - A debug bridge for Android applications

#### **`app/build.gradle`** 
```gradle
dependencies {
    
	/***/

	implementation "com.facebook.stetho:stetho:1.5.1"
	implementation "com.facebook.stetho:stetho-okhttp3:1.5.1"

	/***/
}
```

#### **`MainApplication.java`** 
```java
	
/***/
	
import com.facebook.react.modules.network.ReactCookieJarContainer;
import okhttp3.OkHttpClient;
import com.facebook.react.modules.network.OkHttpClientProvider;
import com.facebook.react.modules.network.OkHttpClientFactory;
	
/***/

public class MainApplication extends Application implements ReactApplication {

	/***/

	@Override
	public void onCreate() {
		super.onCreate();
		SoLoader.init(this, /* native exopackage */ false);

		// if (BuildConfig.DEBUG) {
		Stetho.initializeWithDefaults(this);

		OkHttpClientProvider.setOkHttpClientFactory(new OkHttpClientFactory() {
		  @Override
		  public OkHttpClient createNewNetworkModuleClient() {
			OkHttpClient.Builder builder = new OkHttpClient.Builder()
				.connectTimeout(0, TimeUnit.MILLISECONDS)
				.readTimeout(0, TimeUnit.MILLISECONDS)
				.writeTimeout(0, TimeUnit.MILLISECONDS)
				.cookieJar(new ReactCookieJarContainer())
				.addNetworkInterceptor(new StethoInterceptor());
			return builder.build();
		  }
		});
	}
}
```

After that you should see network requests from mapbox map component in a Chrome DevTools network tab.








