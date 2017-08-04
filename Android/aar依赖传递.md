#### 依赖传递, aar默认不引入arr的依赖，但会自动引入jar的依赖

```gradle
compile('com.foo:FOO:1.0.0@aar') {
       transitive = true
}
```

#### 上面方式不行就只能上传到JCenter或者private maven repo

https://inthecheesefactory.com/blog/how-to-setup-private-maven-repository/en

##### Note:

```
#systemProp.https.proxyPort=1087
#systemProp.http.proxyHost=127.0.0.1
#systemProp.https.proxyHost=127.0.0.1
#systemProp.http.proxyPort=1087
```
