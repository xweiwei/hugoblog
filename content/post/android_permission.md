+++
css = []
date = "2017-07-07T10:59:22+08:00"
description = ""
draft = true
highlight = true
metaimage = "/path/to/img.png"
metavideo = "/path/to/video.mp4"
nodisqus = false
notoc = false
scripts = []
sharebuttons = true
tags = ["Android", "Permission"]
title = "查看Android设备上的Permission"

+++


[TOC]

# 免责说明
为临时整理的文档，用来快速查阅的。

# Android permission level

* normal：低风险权限，只要申请了就可以使用（在AndroidManifest.xml中添加<uses-permission>标签），安装时不需要用户确认；
* dangerous：高风险权限，安装时需要用户的确认才可使用；
* signature：只有当申请权限的应用程序的数字签名与声明此权限的应用程序的数字签名相同时（如果是申请系统权限，则需要与系统签名相同），才能将权限授给它；
* signatureOrSystem：签名相同，或者申请权限的应用为系统应用（在system image中）。

上述四类权限级别同样可用于自定义权限中。如果开发者需要对自己的应用程序（或部分应用）进行访问控制，则可以通过在AndroidManifest.xml中添加<permission>标签，将其属性中的protectionLevel设置为上述四类级别中的某一种来实现。

#　查看权限

adb可以获取到设备上存在的permission信息，一种是通过pm, 另外一种是dumpsys package的方式。
pm获取的信息比较干净，基本就是permission的本身以及和组的联系。
dumpsys package方式获取的就更加的详细，是package与permission相关的信息，更有实用的价值。

## pm 命令获取和permission有关的信息

```
pm list permission-groups: prints all known permission groups.

pm list permissions: prints all known permissions, optionally only
  those in GROUP.  Options:
    -g: organize by group.
    -f: print all information.　　　　　　　　　　　　　　　
    -s: short summary.
    -d: only list dangerous permissions.
    -u: list only the permissions users will see.
```
#### 查看设备上存在的权限组

```
weiwei@WPS:~$ adb shell pm list permission-groups
permission group:android.permission-group.CONTACTS
permission group:android.permission-group.PHONE
permission group:android.permission-group.CALENDAR
permission group:android.permission-group.CAMERA
permission group:android.permission-group.SENSORS
permission group:android.permission-group.LOCATION
permission group:android.permission-group.STORAGE
permission group:com.sina.weibo.permission-group
permission group:android.permission-group.MICROPHONE
permission group:android.permission-group.SMS
```


#### 查看设备上已经有的权限，包括系统定义和应用自定义

```
weiwei@WPS:~$ adb shell pm list permissions
All Permissions:

permission:android.permission.REAL_GET_TASKS
permission:android.permission.ACCESS_CACHE_FILESYSTEM
permission:android.permission.REMOTE_AUDIO_PLAYBACK
permission:android.permission.INTENT_FILTER_VERIFICATION_AGENT
permission:android.permission.BIND_INCALL_SERVICE
permission:android.permission.WRITE_SETTINGS
....
....
....
```

#### 按组查看详细的权限

```
weiwei@WPS:~$ adb shell pm list permissions -g|more
All Permissions:

group:android.permission-group.CONTACTS
  permission:android.permission.WRITE_CONTACTS
  permission:android.permission.GET_ACCOUNTS
  permission:android.permission.READ_CONTACTS

group:android.permission-group.PHONE
  permission:android.permission.READ_CALL_LOG
  permission:android.permission.READ_PHONE_STATE
  permission:android.permission.ACCESS_IMS_CALL_SERVICE
  permission:android.permission.CALL_PHONE
  permission:android.permission.WRITE_CALL_LOG
  permission:android.permission.USE_SIP
  permission:android.permission.PROCESS_OUTGOING_CALLS
  permission:com.android.voicemail.permission.ADD_VOICEMAIL

group:android.permission-group.CALENDAR
  permission:android.permission.READ_CALENDAR
  permission:android.permission.WRITE_CALENDAR

group:android.permission-group.CAMERA
  permission:android.permission.CAMERA

group:android.permission-group.SENSORS
  permission:android.permission.BODY_SENSORS
  permission:android.permission.USE_FINGERPRINT

group:android.permission-group.LOCATION
  permission:android.permission.ACCESS_FINE_LOCATION
  permission:android.permission.ACCESS_COARSE_LOCATION

group:android.permission-group.STORAGE
  permission:android.permission.READ_EXTERNAL_STORAGE
  permission:android.permission.WRITE_EXTERNAL_STORAGE

group:com.sina.weibo.permission-group
  permission:com.sina.weibo.permission.USER

group:android.permission-group.MICROPHONE
  permission:android.permission.RECORD_AUDIO

group:android.permission-group.SMS
  permission:android.permission.READ_SMS
  permission:android.permission.RECEIVE_WAP_PUSH
  permission:android.permission.RECEIVE_MMS
  permission:android.permission.RECEIVE_SMS
  permission:android.permission.SEND_SMS
  permission:android.permission.READ_CELL_BROADCASTS

ungrouped:
  permission:android.permission.REAL_GET_TASKS
  permission:android.permission.ACCESS_CACHE_FILESYSTEM
  permission:android.permission.REMOTE_AUDIO_PLAYBACK
  permission:android.permission.INTENT_FILTER_VERIFICATION_AGENT
  permission:android.permission.BIND_INCALL_SERVICE
  permission:android.permission.WRITE_SETTINGS
  permission:android.permission.CONTROL_KEYGUARD
  permission:android.permission.CONFIGURE_WIFI_DISPLAY
  permission:android.permission.ACCESS_WIMAX_STATE
  permission:android.permission.SET_INPUT_CALIBRATION
  permission:android.permission.RECOVERY
  permission:android.permission.TEMPORARY_ENABLE_ACCESSIBILITY
  permission:android.permission.SET_PROCESS_LIMIT
  permission:android.permission.FRAME_STATS
  permission:android.permission.BRICK
  permission:com.tencent.qqsecure.INNER_BROCAST
  permission:android.permission.RESTART_PACKAGES
  permission:android.permission.USE_CREDENTIALS
  permission:android.permission.BIND_KEYGUARD_APPWIDGET
  permission:com.gionee.cloud.permission.SEND
  permission:android.permission.BIND_DEVICE_ADMIN
  permission:android.permission.MODIFY_AUDIO_SETTINGS
  permission:android.permission.ACCESS_CHECKIN_PROPERTIES
```

#### 显示权限详细信息

```
weiwei@WPS:~$ adb shell pm list permissions -f|more
All Permissions:

+ permission:android.permission.REAL_GET_TASKS
  package:android
  label:null
  description:null
  protectionLevel:signature|privileged
+ permission:android.permission.ACCESS_CACHE_FILESYSTEM
  package:android
  label:null
  description:null
  protectionLevel:signature|privileged

...


+ permission:com.meituan.PASSPORT
  package:com.sankuai.meituan
  label:美团用户信息
  description:null
  protectionLevel:signature
+ permission:android.permission.ACCESS_DRM_CERTIFICATES
  package:android
  label:null
  description:null
  protectionLevel:signature|privileged
+ permission:com.tencent.mm.permission.C2D_MESSAGE
  package:com.tencent.mm
  label:null
  description:null
  protectionLevel:signature
+ permission:com.tencent.news.permisson.ACTION
  package:com.tencent.news
  label:null
  description:null
  protectionLevel:signature

...

+ permission:com.jinli.c2u.permission.C2D_MESSAGE
  package:com.gionee.aora.market
  label:null
  description:null
  protectionLevel:normal
+ permission:android.permission.START_PRINT_SERVICE_CONFIG_ACTIVITY
  package:com.android.printspooler
  label:start print service configuration activities
  description:Allows the holder to start the configuration activities of a print service. Should never be needed for normal apps.
  protectionLevel:signature

...

```

#### 显示权限摘要

```
weiwei@WPS:~$ adb shell pm list permissions -s|more
All Permissions:

通讯录: 修改您的通讯录, 查找设备上的帐户, 读取您的通讯录

电话: 读取通话记录, 读取手机状态和身份, 使用即时通讯通话服务, 直接拨打电话号码, 写入通话记录, 拨打/接听SIP电话, 重新设置外拨电话的路径, 添加语音邮件

日历: 读取日历活动和机密信息, 添加或修改日历活动，并在所有者不知情的情况下向邀请对象发送电子邮件

相机: 拍摄照片和视频

身体传感器: 人体传感器（如心跳速率检测器）, 使用指纹硬件

位置信息: 精确位置（基于GPS和网络）, 大致位置（基于网络）

存储空间: 读取您的SD卡中的内容, 修改或删除您的SD卡中的内容

微博: 监听微博帐户

麦克风: 录音

短信: 读取您的讯息（短信或彩信）, 接收讯息 (WAP), 接收讯息（彩信）, 接收讯息（短信）, 发送和查看短信, 读取小区广播消息

ungrouped:
null, null, null, null, null, 修改系统设置, null, null, 建立或中断 WiMAX 网络连接, null, null, null, null, null, null, null, 关闭其他应用, null, null, null, null, 更改您的音频设置, null, null, null, null, “
扰”模式使用权限, null, null, null, null, null, null, null, null, null, null, null, null, access all print jobs, null, null, null, null, 在其他应用之上显示内容, 绑定到运营商服务, 使用QQ浏览器, Broadcast phone
 account registration, null, null, null, null, null, OMAPI System Terminal, Broadcast the call type/duration information, null, null, null, 将蓝牙设备列入访问权限白名单。, null, null, null, null, null, null,
 控制近距离通信, null, null, Process phone account registration, null, null, null, null, null, 发送下载通知。, null, null, null, null, 更改网络连接性, null, null, 让应用始终运行, null, 启用和停用同步, null, 
null, null, null, null, null, 开机启动, null, null, null, 安装 DRM 内容。, null, null, null, null, 设置时区, null, 展开/收拢状态栏, 卸载快捷方式, 管理个人资料和设备所有者, null, null, null, null, null, null,
 与蓝牙设备配对, null, 允许接收WLAN多播, null, null, null, 设置闹钟, null, null, null, null, 检索正在运行的应用, null, null, null, null, 完全的网络访问权限, null, 发射红外线, null, 对正在运行的应用重新排序, 
null, null, 访问蓝牙设置, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, 计算应用存储空间, null, null, null, null, null, 访问所有系统下载内容, null, null, nul
l, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, remove installed secure services, null, null, null, install developer trustlet
, null, null, 获取额外的位置信息提供程序命令, null, null, 访问下载管理器。, 发送持久广播, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, nul
l, 连接WLAN网络和断开连接, null, 读取应用的未读信息, null, 读取安装会话, null, null, null, null, 使用下载管理器。, null, null, null, null, null, null, 控制闪光灯, null, null, 查看网络连接, null, null, 访问 D
RM 内容。, null, 停用屏幕锁定, null, Register to handle the broadcasted call type/duration information, null, null, null, null, null, null, 性能检测, null, 设置壁纸, null, null, null, null, null, null, null,
 null, null, 美团用户信息, null, null, null, null, 关闭其他应用, null, null, null, null, null, null, null, 读取同步统计信息, null, null, null, null, 保留下载缓存中的空间, null, null, null, null, null, null, 
null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, 调整您的壁纸大小, null, 读取同步设置, null, null, null, null, null, null, n
ull, null, null, null, null, 控制振动, null, null, null, null, Broadcast that a change happened to the call log., null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, nul
l, start print service configuration activities, null, null, null, null, null, null, 查看WLAN连接, null, null, null, 更改 WiMAX 状态, null, null, 请求安装文件包, null, null, 安装快捷方式, null, null, null, n
ull, null, null, null, null, null, null, 防止手机休眠, null, null, 高级下载管理器功能。, null, 访问电子邮件服务提供商数据, null, null, null, null, null, Bind OMAPI Terminal, null, null, null

```

#### 只显示dangerous权限

```
weiwei@WPS:~$ adb shell pm list permissions -d
Dangerous Permissions:


```
！查看的手机上没有Dangerous权限


#### dangerous和Normal权限


```
weiwei@WPS:~$ adb shell pm list permissions -u
Dangerous and Normal Permissions:

permission:android.permission.ACCESS_WIMAX_STATE
permission:android.permission.RESTART_PACKAGES
permission:android.permission.USE_CREDENTIALS
permission:com.gionee.cloud.permission.SEND
permission:android.permission.MODIFY_AUDIO_SETTINGS
permission:android.permission.ACCESS_NOTIFICATION_POLICY
permission:com.gionee.setupwizard.permission.SECURE_SERVICE
permission:com.ted.sdk.yellow.provider.permission.READ
permission:android.permission.MANAGE_ACCOUNTS
permission:com.tencent.mtt.permission.SDK
permission:com.mediatek.systemupdate.sysoper.permission.ACCESS_SERVICE
permission:android.permission.NFC
permission:android.permission.CHANGE_NETWORK_STATE
permission:android.permission.PERSISTENT_ACTIVITY
permission:android.permission.WRITE_SYNC_SETTINGS
permission:com.gionee.anti.stolen.permission.C2D_MESSAGE
permission:android.permission.RECEIVE_BOOT_COMPLETED
permission:android.permission.SUBSCRIBED_FEEDS_READ
permission:android.permission.INSTALL_DRM
permission:com.gionee.client.permission.MMOAUTH_CALLBACK
permission:android.permission.SET_TIME_ZONE
permission:android.permission.EXPAND_STATUS_BAR
permission:com.android.launcher.permission.UNINSTALL_SHORTCUT
permission:android.permission.REQUEST_IGNORE_BATTERY_OPTIMIZATIONS
permission:android.permission.READ_PROFILE
permission:com.ted.sdk.yellow.provider.permission.WRITE
permission:android.permission.BLUETOOTH
permission:android.permission.CHANGE_WIFI_MULTICAST_STATE
permission:com.android.alarm.permission.SET_ALARM
permission:android.permission.GET_TASKS
permission:android.permission.SUBSCRIBED_FEEDS_WRITE
permission:android.permission.AUTHENTICATE_ACCOUNTS
permission:android.permission.INTERNET
permission:android.permission.TRANSMIT_IR
permission:android.permission.REORDER_TASKS
permission:com.android.browser.permission.READ_HISTORY_BOOKMARKS
permission:android.permission.BLUETOOTH_ADMIN
permission:android.permission.ADVANCED_WIDGET_API
permission:android.permission.WRITE_SOCIAL_STREAM
permission:android.permission.GET_PACKAGE_SIZE
permission:com.cmcc.ccs.WRITE_CCS_MESSAGE
permission:android.permission.WRITE_PROFILE
permission:ctrip.android.view.push
permission:getui.permission.GetuiService.com.achievo.vipshop
permission:com.tencent.mm.ext.permission.SPORT
permission:com.sina.permission.SINA_PUSH
permission:com.gd.mobicore.pa.permission.DEVELOPER_PERMISSION
permission:android.permission.ACCESS_LOCATION_EXTRA_COMMANDS
permission:android.permission.BROADCAST_STICKY
permission:com.tencent.meri.permission.BACK_ENGINE
permission:android.permission.WRITE_SMS
permission:com.gionee.client.permission.MM_MESSAGE
permission:android.permission.CHANGE_WIFI_STATE
permission:com.android.launcher.permission.READ_MISS_INFO
permission:android.permission.READ_INSTALL_SESSIONS
permission:com.sina.weibo.permission.NOUSER_BROADCAST
permission:gn.com.android.permission.UPGRADE
permission:android.permission.FLASHLIGHT
permission:android.permission.ACCESS_NETWORK_STATE
permission:android.permission.DISABLE_KEYGUARD
permission:android.permission.SET_WALLPAPER
permission:android.permission.KILL_BACKGROUND_PROCESSES
permission:android.permission.WRITE_USER_DICTIONARY
permission:android.permission.READ_SYNC_STATS
permission:com.qihoo.antivirus.update.permission.qvs_sdk.com.gionee.systemmanager
permission:android.permission.SET_WALLPAPER_HINTS
permission:android.permission.READ_SYNC_SETTINGS
permission:com.android.browser.permission.WRITE_HISTORY_BOOKMARKS
permission:android.permission.VIBRATE
permission:com.gionee.cloud.permission.RECEIVE
permission:android.permission.READ_USER_DICTIONARY
permission:com.sina.weibo.permission.WEIBO_SDK_PERMISSION
permission:com.jinli.c2u.permission.C2D_MESSAGE
permission:com.cmcc.ccs.READ_CCS_MESSAGE
permission:android.permission.ACCESS_WIFI_STATE
permission:com.tencent.meri.permission.FORE_SERVICE
permission:android.permission.CHANGE_WIMAX_STATE
permission:android.permission.REQUEST_INSTALL_PACKAGES
permission:com.android.launcher.permission.INSTALL_SHORTCUT
permission:android.permission.READ_SOCIAL_STREAM
permission:android.permission.WAKE_LOCK
permission:com.gionee.encryptspace.permission.ACCESS_TOKEN_SERVICE

```

## 通过dumpsys package 获取与permission相关的信息

dumpsys package 能获取到的信息比pm的更为详细


```
    perm[issions]: dump permissions
    permission [name ...]: dump declaration and use of given permission

    check-permission <permission> <package> [<user>]: does pkg hold perm?
```

#### 获取所有permissions信息

```
weiwei@WPS:~$ adb shell dumpsys package perm|more
Permissions:
  Permission [android.permission.REAL_GET_TASKS] (8f56ef):
    sourcePackage=android
    uid=1000 gids=null type=0 prot=signature|privileged
    perm=Permission{d0adb25 android.permission.REAL_GET_TASKS}
    packageSetting=PackageSetting{b9e5929 android/1000}
  Permission [android.permission.ACCESS_CACHE_FILESYSTEM] (818ed0e):
    sourcePackage=android
    uid=1000 gids=[2001] type=1 prot=signature|privileged
    perm=Permission{c8e3aab android.permission.ACCESS_CACHE_FILESYSTEM}
    packageSetting=PackageSetting{b9e5929 android/1000}
  Permission [android.permission.REMOTE_AUDIO_PLAYBACK] (f19c011):
    sourcePackage=android
    uid=1000 gids=null type=0 prot=signature
    perm=Permission{9b14508 android.permission.REMOTE_AUDIO_PLAYBACK}
    packageSetting=PackageSetting{b9e5929 android/1000}
  Permission [android.permission.DOWNLOAD_WITHOUT_NOTIFICATION] (e979714):
    sourcePackage=com.android.providers.downloads
    uid=10001 gids=null type=0 prot=normal
    perm=Permission{fb304a1 android.permission.DOWNLOAD_WITHOUT_NOTIFICATION}
    packageSetting=PackageSetting{c0a4741 com.android.providers.downloads/10001}
  Permission [android.permission.INTENT_FILTER_VERIFICATION_AGENT] (f2492a6):
    sourcePackage=android
    uid=1000 gids=null type=0 prot=signature|privileged
    perm=Permission{2836c6 android.permission.INTENT_FILTER_VERIFICATION_AGENT}
    packageSetting=PackageSetting{b9e5929 android/1000}

...

  Permission [com.sankuai.common.PERMISSION] (8a4ed1):
    sourcePackage=com.sankuai.meituan
    uid=10124 gids=null type=0 prot=signature
    perm=Permission{6e8d215 com.sankuai.common.PERMISSION}
    packageSetting=PackageSetting{a385470 com.sankuai.meituan/10124}

...

AppOp Permissions:
  AppOp Permission android.permission.WRITE_SETTINGS:
    com.android.providers.telephony
    com.gionee.amisystem
    com.android.providers.calendar
    com.android.providers.media
    com.softsim.control
    com.gionee.dataghost
    com.qiyi.video
    com.amigo.chameleon
    com.iflytek.gionee.ringdiyclient
    com.sankuai.meituan
    com.gionee.ringtone
    com.iflytek.speechsuite
    com.mediatek.engineermode
    com.oupeng.max.sdk
    com.amigo.search
    com.gionee.anti.stolen

...

AppOp Permissions:
  AppOp Permission android.permission.WRITE_SETTINGS:
    com.android.providers.telephony
    com.gionee.amisystem
    com.android.providers.calendar
    com.android.providers.media
    com.softsim.control
    com.gionee.dataghost
    com.qiyi.video
    com.amigo.chameleon
    com.iflytek.gionee.ringdiyclient
    com.sankuai.meituan
    com.gionee.ringtone
    com.iflytek.speechsuite
    com.mediatek.engineermode
    com.oupeng.max.sdk
    com.amigo.search
    com.gionee.anti.stolen
...

  AppOp Permission android.permission.PACKAGE_USAGE_STATS:
    com.gionee.amisystem
    com.mediatek.engineermode
    cn.richinfo.dm
    com.android.providers.applications
    android
    com.tencent.qqpimsecure
    com.gionee.aora.market
    com.android.settings
    com.gionee.softmanager

```


#### 显示特定权限的详细信息

包括权限所有者和声明使用者

```
weiwei@WPS:~$ adb shell dumpsys package permission com.gionee.cloud.permission.SEND|more
Permissions:
  Permission [com.gionee.cloud.permission.SEND] (20f4d26):
    sourcePackage=com.gionee.cloud.gpe
    uid=10074 gids=null type=0 prot=normal
    perm=Permission{fe0485a com.gionee.cloud.permission.SEND}
    packageSetting=PackageSetting{8266a8b com.gionee.cloud.gpe/10074}

Packages:
  Package [com.appsipper.demo] (b2ec739):
    userId=1000
    sharedUser=SharedUserSetting{1aa278b android.uid.system/1000}
    pkg=Package{cfe9300 com.appsipper.demo}
    codePath=/system/app/Amigo_AppSipper
    versionCode=23 targetSdk=23
    versionName=1.1.0.zp
    splits=[base]
    applicationInfo=ApplicationInfo{e016068 com.appsipper.demo clone=false}
    flags=[ SYSTEM HAS_CODE ALLOW_CLEAR_USER_DATA ALLOW_BACKUP ]
    pkgFlagsEx=[ ]
    dataDir=/data/user/0/com.appsipper.demo
    supportsScreens=[small, medium, large, xlarge, resizeable, anyDensity]
    timeStamp=2017-03-24 13:21:42
    firstInstallTime=2017-03-24 13:21:42
    lastUpdateTime=2017-03-24 13:21:42
    signatures=PackageSignatures{83b3126 [391b167]}
    installPermissionsFixed=false installStatus=1
    pkgFlags=[ SYSTEM HAS_CODE ALLOW_CLEAR_USER_DATA ALLOW_BACKUP ]
    requested permissions:
    install permissions:
      com.gionee.cloud.permission.SEND: granted=true
    User 0:  installed=true hidden=false stopped=false notLaunched=false enabled=0
  Package [com.goodix.fingerprint] (df7b62d):
    userId=1000
    sharedUser=SharedUserSetting{1aa278b android.uid.system/1000}
    pkg=Package{d81b398 com.goodix.fingerprint}
    codePath=/system/app/GFManager
    versionCode=4 targetSdk=20
    versionName=1.0.04
    splits=[base]
    applicationInfo=ApplicationInfo{481f7f1 com.goodix.fingerprint clone=false}
    flags=[ SYSTEM HAS_CODE PERSISTENT ALLOW_CLEAR_USER_DATA ALLOW_BACKUP ]
    pkgFlagsEx=[ ]
    dataDir=/data/user/0/com.goodix.fingerprint
    supportsScreens=[small, medium, large, xlarge, resizeable, anyDensity]
    timeStamp=2017-03-24 13:21:42
    firstInstallTime=2017-03-24 13:21:42
    lastUpdateTime=2017-03-24 13:21:42
    signatures=PackageSignatures{4d3c257 [9aa7144]}
    installPermissionsFixed=false installStatus=1
    pkgFlags=[ SYSTEM HAS_CODE PERSISTENT ALLOW_CLEAR_USER_DATA ALLOW_BACKUP ]
    requested permissions:
    install permissions:
      com.gionee.cloud.permission.SEND: granted=true
    User 0:  installed=true hidden=false stopped=false notLaunched=false enabled=0

...

Shared users:
  SharedUser [android.uid.systemui] (e957c4a):
    userId=10002
    install permissions:
      com.gionee.cloud.permission.SEND: granted=true
    User 0: 
      gids=[3002, 1023, 1015, 3003, 3001, 3006]
      runtime permissions:
  SharedUser [android.uid.system] (1aa278b):
    userId=1000
    install permissions:
      com.gionee.cloud.permission.SEND: granted=true
    User 0: 
      gids=[3002, 1023, 1015, 3003, 3001, 3005, 1007, 3006, 4000]
      runtime permissions:

```

#### 查看某个package是否有特定的permission


有
```
weiwei@WPS:~$ adb shell dumpsys package check-permission com.gionee.cloud.permission.SEND com.gionee.softmanager
0

```

没有
```
weiwei@WPS:~$ adb shell dumpsys package check-permission com.gionee.cloud.permission.SEND com.gionee.anti.stolen
-1
```

