---
title: 多渠道打包，不同的包名
date: 2019-01-17 15:09:23
tags: android
---

每个Android项目都有唯一的一个applicationId，在商店市场中，此ID用来标识你的应用。  

当我们需要打不同特性的版本，如普通版，专业版，而这2个版本是需要在手机里共存，这时候我们需要在不同的渠道中修改applicationId，在渠道的配置的方式如下

```

 productFlavors {

        pro{
            applicationId = "com.example.myapp.pro"
        }
        
         normal{

        }
    }

```
如我们的applicationId为**com.example.myapp**，当我们在打pro渠道的时候， applicationId会被替换为 **com.example.myapp.pro**  

### 注意
**Installation error: INSTALL_FAILED_CONFLICTING_PROVIDER** 

当我们有使用到FileProvider的时候，会有以上的错误，因为2个应用的provider同名了，2个应用需要用不同的provider名字区分，名字也可以在build中动态配置，步骤如下

1. manifest中使用占位符

```
        <provider
            android:name="android.support.v4.content.FileProvider"
            android:authorities="com.example.myapp.${provider_name}"
            android:exported="false"
            android:grantUriPermissions="true">
```

2. 在渠道中定义变量
```

 productFlavors {

        pro{
            applicationId = "com.example.myapp.pro"
            manifestPlaceholders = [provider_name:"pro"]
        }
        
    }

```

这样就可以在手机中同时安装2个渠道的包了！