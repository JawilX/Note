## 1. 注册Bintray账号, 获取API Key, 创建自己的maven仓库

* JCenter由[Bintray公司](https://bintray.com)在维护, 因此需要注册个Bintray账号https://bintray.com/signup/oss
* 登陆后在首页右上角点击用户名选项下的Edit Profile, 然后点击左下角的API Key, 这里一般需要再次提交一次密码, 然后右边的图标可以复制这个API Key
* 点击首页的`Add New Repository  `按下图填写并Create

![image](https://github.com/xuuuu/Pictures/blob/master/bintray_create_maven_repository.png?raw=true)

## 2. 项目配置

* 打开项目根目录下的`local.properties`文件(如果没有就新建一个)，输入你的Bintray账号的信息

```
bintray.user= [your name]
bintray.apikey= [your api key]
```

* 在最外层build.gradle添加插件, 代码如下

```gradle
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.3.1'

        // 版本号跟gradle的版本号相关
        classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.5'
        classpath 'com.github.dcendents:android-maven-gradle-plugin:1.5'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        jcenter()
    }
}
```

* 打开库项目中的build.gradle文件, 根据一下内容进行修改, 其中注释了`#CONFIG#`的地方都可以根据实际情况进行修改。

```gradle
apply plugin: 'com.android.library'

apply plugin: 'com.github.dcendents.android-maven'
apply plugin: 'com.jfrog.bintray'

android {
    compileSdkVersion 25
    buildToolsVersion '25.0.0'
    resourcePrefix "[libary_name]_"                      // #CONFIG# // resource prefix

    defaultConfig {
        minSdkVersion 17
        targetSdkVersion 25
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
}

version = "1.0.0"                                        // #CONFIG# // project version
def siteUrl = ''  // #CONFIG# // project homepage 可以为空
def gitUrl = ''   // #CONFIG# // project git url 可以为空
def issueUrl = '' // #CONFIG# // project issue url 可以为空
group = "com.geartech.gcordsdk"  // #CONFIG# // Maven Group ID for the artifact (pageckage name is ok)


//generate javadoc and jar

install {
    repositories.mavenInstaller {
        // generates POM.xml with proper parameters
        pom {
            project {
                packaging 'aar'
                name 'Library Name'               // #CONFIG# // project title
                url siteUrl
                // Set your license
                licenses {
                    license {
                        name 'The Apache Software License, Version 2.0'
                        url 'http://www.apache.org/licenses/LICENSE-2.0.txt'
                    }
                }
                developers {
                    developer {
                        id 'user_id'          // #CONFIG# // your user id (you can write your nickname)
                        name 'name'           // #CONFIG# // your user name
                        email 'email'         // #CONFIG# // your email
                    }
                }
                scm {
                    connection gitUrl
                    developerConnection gitUrl
                    url siteUrl
                }
            }
        }
    }
}

task sourcesJar(type: Jar) {
    from android.sourceSets.main.java.srcDirs
    classifier = 'sources'
}

// aidl文件不能生成javadoc, 如果不含有aidl文件可以取消下面的注释
/*task javadoc(type: Javadoc) {
    source = android.sourceSets.main.java.srcDirs
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
}
task javadocJar(type: Jar, dependsOn: javadoc) {
    classifier = 'javadoc'
    from javadoc.destinationDir
}*/
artifacts {
//    archives javadocJar
    archives sourcesJar
}

//bintray upload

Properties properties = new Properties()
properties.load(project.rootProject.file('local.properties').newDataInputStream())
bintray {
	// 这里就使用到了你之前在local.properties中添加的信息
    user = properties.getProperty("bintray.user")
    key = properties.getProperty("bintray.apikey")
    configurations = ['archives']
    pkg {
        repo = "maven"
        name = "gcordsdk"                    // #CONFIG# project name in jcenter
        desc = ''
        websiteUrl = siteUrl
        vcsUrl = gitUrl
        issueTrackerUrl = issueUrl
        licenses = ["Apache-2.0"]
        labels = ['android']
        publish = true
        publicDownloadNumbers = true
    }
}
```

## 3. 上传到Bintray

选择工具栏中的`Sync projects with Gradle files`对项目进行重建, 点开AS的Terminal, 默认在当前项目路径下, 执行`./gradlew bintrayUpload`命令, 第一次可能需要下载gradle, 在上传的过程中一般会在97%卡住一会, 如果卡住时间过长, 可以Ctrl+C结束命令再重试

## 4. Add to JCenter

进入[Bintray](https://bintray.com)官网, 点进你创建的maven仓库, 再点击你上传上来的库, 把屏幕拖下去, 点击右下方`Linked to`右边的`Add to JCenter`, 然后什么都不用写直接提交, 每天半夜12点半准时审核通过(= = 美国时间上班了), 通过后会给你发个通知.

现在就可以直接点击Bintray你的项目下面`Maven build settings`中的`Gradle`, 直接复制这个依赖到项目就可以用了. 以后更新version了, 重复第三步提交一遍会自动更新到JCenter.

## 5. 注意

关联JCenter后如果想删除这个项目, 可以点这个项目页面下面的`Edit`, 然后先点Version List删除所有的版本, 然后在点Package Details删除这个项目, 这样能删除对应JCenter里面的包, 不然如果直接删除当前项目, JCenter里面的包会一直存在, 这个时候你如果想要再创建相同项目名或者相同GroupID的项目就会提示已经存在, 这个时候唯一的办法就是在Bintray下面搜索到这个项目证明你是它的creator, 如果审核通过, 这个项目就会重新回到你当前账号的maven包下.