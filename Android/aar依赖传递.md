#### 依赖传递, aar默认不引入arr的依赖，但会自动引入jar的依赖
```gradle
compile('com.foo:FOO:1.0.0@aar') {
       transitive = true
}
```
