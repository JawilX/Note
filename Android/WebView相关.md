## [WebView Chrome Official FAQ](https://developer.chrome.com/multidevice/webview/overview)

## [WebView Official Dev Document](https://developer.android.com/reference/android/webkit/WebView.html)


## WebView加载本地Web文件
#### 1. 访问assets下dist文件夹的index.html文件

```java
mWebView.loadUrl("file:///android_asset/dist/index.html");
```

#### 2. html文件里面引用js文件必须用相对路径./

#### 3. Android要调用的js方法必须放在html文件里面, 暂时不知道怎么调用js文件里面的js方法

```java
mWebView.loadUrl(“javascript:methodName(parameterValues)”)
```
