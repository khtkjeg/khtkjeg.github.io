---
layout: post
title: elasticsearch5.x x-pack产检license破解 
categories: elasticsearch
description: 每天记录一点点，快乐工作一辈子
keywords: elasticsearch x-pack
---

> 从5.0版本开始，Elastic将一些重要的插件整合成了X-Pack。免费的license只能使用一年，而且很多插件无法使用。如果想要试用，需要进行破解。

*1.* 首先完成原版X-Pack在Elastic上的安装 `./elasticsearch-plugin install x-pack`

*2.* 安装完成后再`elasticsearch5.x/plugins/x-pack/`目录下找到x-pack-5.x.0.jar

*3.* 这里使用JD-GUI是无法反编译的，我使用的是[Luyten](https://github.com/deathmarine/Luyten/releases/tag/v0.5.0)进行反编译

*4.* 将`org.elasticsearch/license/LicenseVerifier.class`反编译并保存出来,这个类是检查license完整性的类，我们使其始终返回true，就可以任意修改license并导入。将其改为

```java
package org.elasticsearch.license;

import java.nio.*;
import java.util.*;
import java.security.*;
import org.elasticsearch.common.xcontent.*;
import org.elasticsearch.common.io.*;
import java.io.*;

public class LicenseVerifier
{
    public static boolean verifyLicense(final License license, final byte[] encryptedPublicKeyData) {
        return true;
    }

    public static boolean verifyLicense(final License license) {
        return true;
    }
}
```

*5.* 然后需要重新编译class文件。注意这里我们无需编译整个工程，将原来的`x-pack-5.2.0.jar`和依赖包加入CLASSPATH，即可完成单个文件的编译。实际上只用到了3个依赖包，如果是用RPM或DEB安装的，直接运行：

```shell
javac -cp '/usr/share/elasticsearch/lib/elasticsearch-5.x.0.jar:/usr/share/elasticsearch/lib/lucene-core-6.4.0.jar:/usr/share/elasticsearch/plugins/x-pack/x-pack-5.x.0.jar' LicenseVerifier.java
```

*6.* 把`x-pack-5.x.0.jar`用压缩文件管理器打开，将里面的`LicenseVerifier.class`替换掉。再把破解了的jar包部署到各节点上，并重启集群。

*7.* 申请一个免费license。下载后修改，例如：

```json
{'license':{"uid":"helloworld","type":"platinum","issue_date_in_millis":1486598400000,"expiry_date_in_millis":2524579200999,"max_nodes":1000,"issued_to":"helloworld","issuer":"Web Form","signature":"helloworld","start_date_in_millis":1486598400000}}
```

*8.* 这里，`platinum`表示白金版，可以使用所有功能。其他的如`expiry_date_in_millis、max_nodes`等根据自己需要修改即可。

*9.* 把该`license`导入集群即可

```shell
curl -XPUT -u elastic 'http://localhost:9200/_xpack/license?acknowledge=true' -H "Content-Type: application/json" -d @license.json
```

*10.* 破解结果如下
![](/images/posts/elasticsearch/x-pack.png)