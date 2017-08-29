#### 1. 依赖传递, aar默认不引入arr的依赖，但会自动引入jar的依赖

```groovy
compile('com.foo:FOO:1.0.0@aar') {
       transitive = true
}
```

#### 2. 上面方式不行就只能上传到JCenter或者maven repo

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

```
1、在对你的library打包后，检查build目录下publications/aar(或者jar或者mavenJar)/pom-default.xml文件是否存在
2、该pom文件内是否有添加dependencies项
```

```groovy
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

#### gradle build --refresh-dependencies

#### 3. 生成fatjar
- 方法一
```groovy
    task createJar(type: Jar) {
        from {
            List<File> allFiles = new ArrayList<>();
            configurations.compile.collect {
                for (File f : zipTree(it).getFiles()) {
                    if (f.getName().equals("classes.jar")) {
                        allFiles.addAll(zipTree(f).getAt("asFileTrees").get(0).getDir())
                    }
                }
            }
            allFiles.add(new File('build/intermediates/classes/debug'))
            allFiles // To return the result inside a lambda
        }
        archiveName('Trivisa.jar')
    }
```
错误:
```
Error:Execution failed for task ':app:transformClassesWithDexForDebug'.
> com.android.build.api.transform.TransformException: com.android.ide.common.process.ProcessException: java.util.concurrent.ExecutionException: com.android.dex.DexException: Multiple dex files define Landroid/support/v7/app/ActionBar$DisplayOptions;
```
- 方法二
```groovy
android.libraryVariants.all { variant ->
    def name = variant.buildType.name
    if (name.equals(com.android.builder.core.BuilderConstants.DEBUG)) {
        return; // Skip debug builds.
    }
    def task = project.tasks.create "jar${name.capitalize()}", Jar
    task.dependsOn variant.javaCompile
    //Include Java classes
    task.from variant.javaCompile.destinationDir
    //Include dependent jars with some exceptions
    task.from configurations.compile.findAll {
        it.getName() != 'android.jar' && !it.getName().startsWith('junit') && !it.getName().startsWith('hamcrest')
    }.collect {
        it.isDirectory() ? it : zipTree(it)
    }
    artifacts.add('archives', task);
}

运行jarRelease
```
错误:
```
Error:Execution failed for task ':app:transformClassesWithDexForDebug'.
> com.android.build.api.transform.TransformException: com.android.ide.common.process.ProcessException: java.util.concurrent.ExecutionException: com.android.dex.DexException: Multiple dex files define Landroid/support/annotation/AnimRes;
```

#### 4. 生成fataar
```groovy
    task sync_jars() << {
        //把所有依赖的.jar库都拷贝到build/aar/libs下
        copy {
            into buildDir.getPath() +"/aar/libs"
            from configurations.compile.findAll {
                it.getName().endsWith(".jar")
            }
        }
    }
    task sync_aars(dependsOn:':lib:assembleRelease') << {
        //把所有依赖的.aar库里包含的classes.jar都拷贝到build/aar/libs下，并重命名以不被覆盖
        def jar_name
        def aar_path
        def dest_dir = buildDir.getPath()+"/aar"
        configurations.compile.findAll {
            it.getName().endsWith(".aar")
        }.collect {
            aar_path = it.getPath()
            jar_name = "libs/"+it.getName().replace(".aar",".jar")
            copy {
                from zipTree(aar_path)
                into dest_dir
                include "**/*"
                rename 'classes.jar', jar_name
            }
        }
    }
    task fataar(dependsOn:[sync_aars, sync_jars]) << {
        /*task (obfuse_classes_jar, type: proguard.gradle.ProGuardTask) {
            //把build/aar/libs*//*.jar混淆后生成build/aar/classes.jar
            configuration "proguard.cfg"
            injars buildDir.getPath()+"/aar/libs"
            outjars buildDir.getPath()+"/aar/classes.jar"
            libraryjars "${System.getProperty('java.home')}/lib/rt.jar"
            libraryjars "${System.getProperty('java.home')}/Contents/Classes/classes.jar"
            libraryjars System.getenv("ANDROID_HOME")+"/platforms/android-26/android.jar"
        }.execute()*/
        task (gen_aar, type: Zip) {
            //把生成最终的aar包，注意libs目录需要被排除
            def dest_dir = buildDir.getPath()+"/aar/"
            baseName = "Trivisa"
            extension = "aar"
            destinationDir = file(buildDir.getPath())
            from dest_dir
            exclude "libs"
        }.execute()
    }
```
错误:
```
Error:Execution failed for task ':app:processDebugManifest'.
> Manifest merger failed : Attribute meta-data#android.support.VERSION@value value=(26.0.0-alpha1) from [com.android.support:appcompat-v7:26.0.0-alpha1] AndroidManifest.xml:27:9-38
  	is also present at [:Trivisa:] AndroidManifest.xml:27:9-31 value=(25.3.1).
  	Suggestion: add 'tools:replace="android:value"' to <meta-data> element at AndroidManifest.xml:25:5-27:41 to override.
```
