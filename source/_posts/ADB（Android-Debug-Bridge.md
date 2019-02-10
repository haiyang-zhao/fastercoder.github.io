title: ADB（Android Debug Bridge)
date: 2018-07-10 09:47:20
categories: Android
tags: ADB
---

## ADB（Android Debug Bridge)
Android调试桥（adb），它是一个通用命令行工具，可以管理、调试Emulator(模拟机)或Device(安卓真机)，并且提供对 Unix shell的访问。它是一个C/S架构的应用程序，由三部分组成

* Client端 ：运行在开发机器中, 即你的开发PC机. 用来发送adb命令。
* Deamon守护进程 ：该组件在设备上运行命令。后台程序在每个模拟器或设备实例上作为后台进程运行。
* Server端 ：作为一个后台进程运行在开发机器中, 即你的开发PC机。 用来管理PC中的Client端和手机的Deamon之间的通信。



## 安装
* android_sdk/platform-tools/

```
export ANDROID_HOME=/Users/zhaohaiyang/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools
```
* brew cask install android-platform-tools

<!--more-->

## adb 的工作方式

* 启动一个 adb 客户端时，此客户端首先检查是否有已运行的 adb 服务器进程。如果没有，它将启动服务器进程。当服务器启动时，它与本地 TCP 端口 5037 绑定，并侦听从 adb 客户端发送的命令—所有 adb 客户端均使用端口 5037 与 adb 服务器通信。
* 服务器设置与所有运行的模拟器/设备实例的连接。它通过扫描 5555 到 5585 之间（模拟器/设备使用的范围）的奇数号端口查找模拟器/设备实例。服务器一旦发现 adb 后台程序，它将设置与该端口的连接。
* 服务器和模拟器/设备连接时使用两个(一组)端口进行。一个奇数的5555，先建立adb server与adb daemon的调试专用的连接；一个为偶数的5554，再建立与Emulator（或Device）实例的连接。
adb server通过扫描5555—5585之间的奇数端口搜索adb daemon，进行adb 连接；而用相应的偶数端口(奇数端口号-1，如5555对应5554)进行Emulator或Device实例的连接。
![image](https://upload-images.jianshu.io/upload_images/1888909-2a4a68b3d96ee4b3.png)

## 开启adb调试

* 在开发者选项中打开USB调试 ：Settings > About phone 并点按 Build number 七次，就可以找到开发者选项。
* 连接USB线

## 通过 WLAN 连接到设备

* PC和设备连接同一WLAN。
* USB连接设备。
* 设置目标设备以侦听端口 5555 上的 TCP/IP 连接。

```
adb tcpip 5555
```
* 连接至设备，通过 IP 地址识别此设备。
```
adb connect 192.168.3.102
```
* 请确认您的主计算机已连接至目标设备：

```
~ adb devices
List of devices attached
192.168.3.102:5555    device
```

## 查询设备
在发出 adb 命令之前，知道哪些模拟器/设备实例已连接到 adb 服务器会很有帮助。您可以使用 devices 命令生成已连接的模拟器/设备的列表

adb devices

输出的格式类似如下:

List of devices attached
serial_number state

* 序列号(serial_number): 一个由 adb 创建的字符串,序列号的格式为 type-console-port
* 状态(state)
* offline — 实例未连接到 adb 或不响应。
* device — 实例现在已连接到 adb 服务器。
* no device — 未连接模拟器/设备。

## 设置端口转发

使用 forward 命令设置任意端口转发 — 将对特定PC端口的请求转发到模拟器/设备实例上的其他端口

```
adb forward tcp:6100 tcp:7100 //adb forward <local> <remote>

```

## adb 命令参考


#### 目标设备

* -d : 将 adb 命令发送至唯一连接的 USB 设备,如果连接了多个 USB 设备，将返回错误。
* -e : 将 adb 命令发送至唯一运行的模拟器实例,如果有多个模拟器实例在运行，将返回错误。
* **-s serial_number** : 连接指定的设备/模拟器。

adb -s 192.168.3.102:5555 install .....

#### 常规
* devices :
* help :
* version:

#### 调试
* **logcat [option] [filter-specs]**
```
adb logcat -s TAG //按TAG过滤
adb logcat *:V[/D/I/W/E/F/S] //指定Level,S 表示为不输出该标签的日志
adb logcat > ~/log.txt //输出到文件
adb logcat | grep "^..Activity" //正则匹配
```
* bugreport : 将 dumpsys、dumpstate 和 logcat 数据输出到屏幕，以用于报告错误。
* jdwp :输出给定设备上可用的 JDWP 进程的列表。

#### 数据
* install : adb install xx.apk
* pull : adb pull /sdcard/foo.txt foo.txt
* push : adb push foo.txt /sdcard/foo.txt
#### 端口和网络连接
* forward :
端口规范可使用以下架构：
* tcp:port_number
* local:unix_domain_socket_name
* dev:character_device_name
* jdwp:pid
#### 脚本
* get-serialno : 输出 adb 实例序列号字符串
* get-state : 输出模拟器/设备实例的 adb 状态。
* wait-for-device : 阻止执行，直至设备处于在线状态，即直至此实例状态为 device。

adb wait-for-device install app.apk
#### 服务器
* start-server
* kill-server

#### Shell
* **shell** :命令二进制文件存储在模拟器或设备的文件系统中，其路径为 /system/bin/。
* 按 Control + D 或输入 exit 退出。

## 调用 Activity Manager (am)
在 adb shell 中，可以使用 Activity Manager (am) 工具发出命令以执行各种系统操作，如启动 Activity、强行停止进程、广播 intent、修改设备屏幕属性及其他操作

#### start [options] intent
启动指定启动 intent 指定的 Activity。
```
adb shell am start -n wg.roomis.launcher/.splash.SplashActivity
```
#### startservice [options] intent
启动 intent 指定的 Service。
```
adb shell
su
am startservice -n wg.roomis.launcher/.LauncherService -a wg.roomis.action.sync_time //需要root权限
```
#### broadcast [options] intent
发出广播 intent。
```
adb shell am broadcast -a wg.roomis.CARD_ENTER -e CardNo "A56BDEC7"
```
#### force-stop
强行停止与 package（应用的包名称）关联的所有应用。
```
adb shell am force-stop wg.roomis.launcher
```
#### kill
终止与 package（应用的包名称）关联的所有进程。此命令仅终止可安全终止且不会影响用户体验的进程。
```
adb shell am kill wg.roomis.launcher
```
#### kill-all
终止所有后台进程。

#### display-size [reset|width * height]
替换模拟器/设备显示尺寸。此命令对于在不同尺寸的屏幕上测试您的应用非常有用，它支持使用大屏设备模仿小屏幕分辨率（反之亦然）。
```
adb shell am display-size 1280x800
```
#### display-density [dpi]
替换模拟器/设备显示密度。此命令对于在不同密度的屏幕上测试您的应用非常有用，它支持使用低密度屏幕在高密度环境环境上进行测试（反之亦然）。
```
adb shell am display-density 480

```

## 调用软件包管理器 Package Manager(pm)
在 adb shell 中，可以使用软件包管理器 (pm) 工具发出命令，以对设备上安装的应用软件包进行操作和查询。

#### list packages
list packages [options] filter: 输出所有软件包，或者，仅输出包名称包含 filter 中的文本的软件包。

选项有:
* -f：查看它们的关联文件。
* -d：进行过滤以仅显示已停用的软件包。
* -e：进行过滤以仅显示已启用的软件包。
* -s：进行过滤以仅显示系统软件包。
* **-3：进行过滤以仅显示第三方软件包。**
* -i：查看软件包的安装程序。
* -u：也包括卸载的软件包。

```
adb shell pm list pacakges -3
```
#### path <package>
输出给定 package 的 APK 的路径。
```
~ adb shell pm path wg.roomis.launcher
package:/data/app/wg.roomis.launcher-2/base.apk
package:/data/app/wg.roomis.launcher-2/split_lib_dependencies_apk.apk
package:/data/app/wg.roomis.launcher-2/split_lib_slice_0_apk.apk
package:/data/app/wg.roomis.launcher-2/split_lib_slice_1_apk.apk
package:/data/app/wg.roomis.launcher-2/split_lib_slice_2_apk.apk
package:/data/app/wg.roomis.launcher-2/split_lib_slice_3_apk.apk
package:/data/app/wg.roomis.launcher-2/split_lib_slice_4_apk.apk
package:/data/app/wg.roomis.launcher-2/split_lib_slice_5_apk.apk
package:/data/app/wg.roomis.launcher-2/split_lib_slice_6_apk.apk
package:/data/app/wg.roomis.launcher-2/split_lib_slice_7_apk.apk
package:/data/app/wg.roomis.launcher-2/split_lib_slice_8_apk.apk
package:/data/app/wg.roomis.launcher-2/split_lib_slice_9_apk.apk
```

#### install [options] <path>
选项：

* -l：安装具有转发锁定功能的软件包。
* **-r：重新安装现有应用，保留其数据。**
* -t：允许安装测试 APK。
* -i installer_package_name：指定安装程序软件包名称。
* -s：在共享的大容量存储（如 sdcard）上安装软件包。
* -f：在内部系统内存上安装软件包。
* -d：允许版本代码降级。
* -g：授予应用清单中列出的所有权限。
#### uninstall [options] <package>
选项：

-k：移除软件包后保留数据和缓存目录。

#### clear <package>
删除与软件包关联的所有数据。

```
adb shell pm clear wg.roomis.launcher
```

## 其他

#### 屏幕截图 screencap <filename>
```
~ adb shell screencap /sdcard/screen.png
~ adb pull /sdcard/screen.png
```
#### 录制视频 screenrecord (API>=19 Android 4.4)
screenrecord [options] <filename>
options:
* --help :显示命令语法和选项
* --size widthxheight :    设置视频大小：1280x720。默认值是设备的原生显示分辨率（如果支持），如果不支持，则使用 1280x720。为实现最佳结果，请使用设备的 Advanced Video Coding (AVC) 编码器支持的大小。
* --bit-rate <rate>:    设置视频的视频比特率（以兆比特每秒为单位）。默认值为 4Mbps。您可以增加比特率以提升视频质量，但这么做会导致影片文件变得更大。以下示例将录制比特率设为 6Mbps：

screenrecord --bit-rate 6000000 /sdcard/demo.mp4

* --time-limit <time> :设置最大录制时长（以秒为单位）。默认值和最大值均为 180（3 分钟）。
* --rotate :将输出旋转 90 度。此功能是实验性的。
* --verbose :显示命令行屏幕上的日志信息。如果您不设置此选项，则运行时此实用程序不会显示任何信息。

```
➜  ~ adb shell screenrecord --verbose /sdcard/demo.mp4
Main display is 1280x800 @57.45fps (orientation=0)
Configuring recorder for 1280x800 video/avc at 4.00Mbps
Content area is 1280x800 at offset x=0 y=0
^C
➜  ~ adb pull /sdcard/demo.mp4
```
注：
* 不能录音频
* 不支持在录制时旋转屏幕

#### adb shell input

* adb shell input text : 向获得焦点的EditText控件输入内容,

adb shell input text "roomis-k12"

* adb input keyevent : 该命令主要是向系统发送一个按键指令，实现模拟用户在键盘上的按键动作

```
adb shell input keyevent HOME
adb shell input keyevent BACK
adb shell input keyevent KEYCODE_POWER//26
```
KeyCode:android.view.KeyEvent.java

* adb shell input tap : 向设备发送一个点击操作的指令，参数是<x> <y>坐标

adb shell input tap 100 100

#### adb shell dumpsys

* adb shell dumpsys activity [activites|service|providers|intents| broadcasts|processes]
* adb shell dumpsys cpuinfo
* adb shell dumpsys package
```
adb shell dumpsys package wg.roomis.launcher | grep versonCode //查看版本号
```
* adb shell dumpsys window
```
adb shell dumpsys window displays //查看分辨率
```
#### 压力测试 adb shell monkey

```
adb shell monkey -p wg.roomis.launcher -s 500 --ignore-crashes --ignore-timeouts --monitor-native-crashes -v -v 10000 > ~/monkey_log.txt
//产生时间序列的种子值：500 忽略程序崩溃 、 忽略超时 、 监视本地程序崩溃 、 详细信息级别为2 ， 产生 10000个事件 。
```
## ROOMIS相关

#### Roomis常用文件路径：
* bindConfig文件：/sdcard/roomis/bindConfig
* deviceSn文件：/sdcard/roomis/deviceSn
* Log目录:  /sdcard/roomis/log
* Database文件: /data/data/wg.roomis.launcher/databases/roomisDb.db
* 偏好文件：/data/data/wg.roomis.launcher/shared_prefs/wg.roomis.launcher_preferences.xml
* webview缓存目录：/data/data/wg.roomis.launcher/cache


#### adb命令
* 查看设备 :adb devices
* 连接设备：adb connect 192.168.11.255
* 安装 adb :install rooomis.apk
* 覆盖安装: adb install -r roomis.apk
* 卸载:adb uninstall wg.roomis.launcher
* 导出roomis log：adb pull  /sdcard/roomis/log/roomis.log ~/
* 导出anr log: adb pull /data/anr/traces.txt ~/
* 导出数据库:adb pull /data/data/wg.roomis.launcher/databases/roomisDb.db ~/
* 模拟刷卡：adb shell am broadcast -a wg.roomis.CARD_ENTER -e CardNo "A56BDEC7"
* 开锁：adb shell am broadcast -a wg.roomis.actions.UNLOCK_PASSWORD
* 检查更新:adb shell am broadcast -a wg.roomis.actions.WAKE_UP_UPDATE
* 查看版本号：adb shell dumpsys package wg.roomis.launcher | grep  versionCode
* 同步时间：
```
adb shell
su
am startservice -n wg.roomis.launcher/.LauncherService -a wg.roomis.action.sync_time
```

#### sqlite3
```
cd data/data/wg.roomis.launcher/databases
sqlite3 roomisDb.db;
.table
select * from ApiConfigEntity;

```










