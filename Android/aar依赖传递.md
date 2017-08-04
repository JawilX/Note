#### 依赖传递, aar默认不引入arr的依赖，但会自动引入jar的依赖

```gradle
compile('com.foo:FOO:1.0.0@aar') {
       transitive = true
}
```

#### 上面方式不行就只能上传到JCenter或者maven repo

#### 下面这是上传到private maven repo的方法

https://inthecheesefactory.com/blog/how-to-setup-private-maven-repository/en

https://jeroenmols.com/blog/2015/08/06/artifactory/

https://jeroenmols.com/blog/2015/08/13/artifactory2/

##### Note:

```
#systemProp.https.proxyPort=1087
#systemProp.http.proxyHost=127.0.0.1
#systemProp.https.proxyHost=127.0.0.1
#systemProp.http.proxyPort=1087
```

```gradle
publishing {
    publications {
        aar(MavenPublication) {
            groupId packageName
            version = libraryVersion
            artifactId project.getName()
            artifact("$buildDir/outputs/aar/${project.getName()}-release.aar")

            pom.withXml {
                def dependencies = asNode().appendNode('dependencies')
                configurations.getByName("_releaseCompile").getResolvedConfiguration().getFirstLevelModuleDependencies().each {
                    def dependency = dependencies.appendNode('dependency')
                    dependency.appendNode('groupId', it.moduleGroup)
                    dependency.appendNode('artifactId', it.moduleName)
                    dependency.appendNode('version', it.moduleVersion)
                }
            }
        }
    }
}
```
