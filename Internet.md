错误：
```
  java.net.UnknownHostException: Unable to resolve host "mp.weixin.qq.com": No address associated with hostname
      at java.net.InetAddress.lookupHostByName(InetAddress.java:470)
      at java.net.InetAddress.getAllByNameImpl(InetAddress.java:252)
      at java.net.InetAddress.getAllByName(InetAddress.java:215)
      at com.android.okhttp.internal.Network$1.resolveInetAddresses(Network.java:29)
      at com.android.okhttp.internal.http.RouteSelector.resetNextInetSocketAddress(RouteSelector.java:188)
      at com.android.okhttp.internal.http.RouteSelector.nextProxy(RouteSelector.java:157)
      at com.android.okhttp.internal.http.RouteSelector.next(RouteSelector.java:100)
      at com.android.okhttp.internal.http.HttpEngine.createNextConnection(HttpEngine.java:357)
      at com.android.okhttp.internal.http.HttpEngine.nextConnection(HttpEngine.java:340)
      at com.android.okhttp.internal.http.HttpEngine.connect(HttpEngine.java:330)
      at com.android.okhttp.internal.http.HttpEngine.sendRequest(HttpEngine.java:248)
      at com.android.okhttp.internal.huc.HttpURLConnectionImpl.execute(HttpURLConnectionImpl.java:433)
      at com.android.okhttp.internal.huc.HttpURLConnectionImpl.getResponse(HttpURLConnectionImpl.java:384)
      at com.android.okhttp.internal.huc.HttpURLConnectionImpl.getResponseCode(HttpURLConnectionImpl.java:497)
      at com.android.okhttp.internal.huc.DelegatingHttpsURLConnection.getResponseCode(DelegatingHttpsURLConnection.java:105)
      at com.android.okhttp.internal.huc.HttpsURLConnectionImpl.getResponseCode(HttpsURLConnectionImpl.java)
      at android.study.filedownload.MainActivity$1.run(MainActivity.java:)
      at java.lang.Thread.run(Thread.java:818)
  Caused by: android.system.GaiException: android_getaddrinfo failed: EAI_NODATA (No address associated with hostname)
      at libcore.io.Posix.android_getaddrinfo(Native Method)
      at libcore.io.ForwardingOs.android_getaddrinfo(ForwardingOs.java:55)
      at java.net.InetAddress.lookupHostByName(InetAddress.java:451)
  	... 17 more
```

错误原因及解决方法：
1、没有加网络访问权限，可以加上网络访问权限。

2、手机/模拟器启动后，切换了WiFi/4G等，可以重启下手机。

