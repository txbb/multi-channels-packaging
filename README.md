# multi-channels-packaging
This is an awesome tool(wrote by python) which can package multi channels apks for android


概述
---
每当发新版本的时候，Android客户端会分发到各大市场，比如：豌豆荚，91，小米等。为了统计这些市场的效果（活跃度，下载量等），需要有一种方法唯一标识它们。
每次发版本，产品部都会提供一个渠道列表，Android开发者会根据这些渠道相应地生成渠道包，随着渠道越来越多，客户端如果都是机械手动去打的话，会累死，程序猿都是聪明而懒惰的，客户端打渠道包的方法一直在演进和提高。
结合我自己的项目，我就对META-INF自动化打包作一下描述。
我们首先需要用工具（IDE、maven、ant等等）打一个签名包，这个包作为母包，我们用这个包为基础打各个渠道包用。这个包我们用解压缩工具打开，发现根目录会有一个META-INF目录，如下图所示:

![img](https://github.com/txbb/multi-channels-packaging/blob/master/imgs/img1.png?raw=true)

如果在META-INF目录内添加空文件，可以不用重新签名应用。因此，通过为不同渠道的应用添加不同的空文件，可以唯一标识一个渠道。
首先建立一个空目录(这边命名位packages), 将上面生成好的签名包拷贝至packages
在packages目录下新建一个渠道包配置文件(命名为channels_profile)，每个渠道id为一行，如下：

```
google
wandoujia
xiaomi
360
```

新建一个python脚本，用来给apk添加空的渠道文件，渠道名的前缀为mchannel_,如下是我的python脚本：

```
#！/usr/bin/env python
import zipfile
import shutil
import os
f = open('channel_cfg', 'r');
while True:
    line = f.readline().strip('\n')
    if len(line) == 0:
        break
    else:
        shutil.copy2({your parent apk}, '{your target apk prefix}' + line + '.apk')
        zipped = zipfile.ZipFile('{your target apk prefix}' + line + '.apk', 'a', zipfile.ZIP_DEFLATED)
	empty_channel_file = "META-INF/mchannel_{channel}".format(channel=line)
	fp = open(line, 'w')
	zipped.write(os.path.basename(line), empty_channel_file)
	fp.close
	os.remove(line)
f.close()
```
例子可以参照: [channel_py](https://github.com/txbb/multi-channels-packaging/blob/master/package_channels%2Fchannel_py)

我们原先统计渠道一般都是在AndroidManifest.xml中添加<meta-data>, 如果采用我这种多渠道打包，我们就不能用Manifest的方式了，我们需要手动获取渠道，进行统计,下面是我结合这种多渠道打包方式获取渠道方法（其实就是读取空文件下划线后缀）：

```
private static String CHANNEL;

    public static String getChannel(Context context) {
        if (CHANNEL == null) {
            CHANNEL = "play";
            ZipFile zipfile = null;
            try {
                zipfile = new ZipFile(context.getApplicationInfo().sourceDir);
                Enumeration<?> entries = zipfile.entries();
                while (entries.hasMoreElements()) {
                    ZipEntry entry = ((ZipEntry) entries.nextElement());
                    String entryName = entry.getName();
                    int i = entryName.indexOf("META-INF/mchannel_");
                    if (i > -1) {
                        CHANNEL = entryName.substring("META-INF/mchannel_".length());
                        break;
                    }
                }
            } catch (IOException e) {
                Log.w("msc", e);
            } finally {
                if (zipfile != null) {
                    try {
                        zipfile.close();
                    } catch (IOException ignored) {
                    }
                }
            }
        }
        return CHANNEL;
    }
```
我们第三方统计是用的LeanCloud, 那么我们以LeanCloud为例，我们上报渠道应该这样：

```
AVAnalytics.setAppChannel(CommonUtil.getChannel(this));
```
其他第三方统计肯定有类似的方法

这种打包方式非常快，现在我们每次需要打60个渠道包，5秒之内就搞定(亲测)，效率指数级提升

Developed By
---
 * Email: <mailto:yuleiming198701@gmail.com>
 * Blog: <http://archmages.github.io>
 * G+: <https://plus.google.com/113098790396354410175/posts>
 * 微博：[archmages](http://weibo.com/archmages)
 * QQ：512505302

License
---

    Copyright 2015 archmages

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
