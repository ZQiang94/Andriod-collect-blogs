# Andriod-collect-blogs
blog address: http://seewhy.leanote.com/post/Android-6.0-Changes

原文链接:http://developer.android.com/about/versions/marshmallow/android-6.0-changes.html

伴随着新特性和功能，Android 6.0 (API level 23) 还包含了一些系统以及API行为上的变化。本文着重指出了一些你（开发者）需要了解，并在开发应用时予以考虑的关键变化。

如果你之前发布过Android应用，那么请注意，平台的这些变化会影响到你的应用。
运行时权限（Runtime Permissions)
本次发布引入了一个新的权限模型，让用户可以在运行时直接管理应用权限。这个模型为用户提供了更好的权限可见性和可控性，同时简化（streamling）了应用安装和自动升级的过程。用户可以对已安装的应用逐个赋予（grant）或者取消（revoke）权限。

如果你的应用目标平台是Android 6.0 (API level 23) 或更高，则必须确保在运行时检查权限和请求权限。通过调用新增的checkSelfPermission()方法可以确认应用是否被赋予了某个权限。而要想获取权限，可以调用新增的requestPermissions()方法。即使应用的目标平台不是Android 6.0，也应该在新的权限模型下对其进行测试。

更多关于如何在应用中支持新权限模型的细节，请参见使用系统权限。如果想了解如何评估新权限模型对应用的影响，可以参考权限最佳实践。

休眠和应用待机（Doze and App Standby）
本次发布了针对空闲的设备和应用引入了新的省电优化方案。这些特性会对所有应用产生影响，所以务必在这些新的模式下对你的应用进行测试。

休眠（Doze） 
如果用户的设备没有插线（unplug）并被静置在一旁，那么当屏幕熄灭一段时间后，设备将进入休眠模式，即试图让系统保持在休眠状态下。在这种模式下，设备将定期恢复正常操作一小段时间，让应用可以进行同步，而系统也可以执行未完成（pending）的操作。
应用待机（App Standby） 
应用待机让系统可以在用户没有主动使用某个应用时，将该应用判定为空闲。如果用户在一段时间内没有点击某个应用，系统就会做出这样的判断。而如果此时设备没有插线，那么那些被认定为空闲的应用，将会被系统禁用网络访问，同步操作和工作线程（job）也会被挂起。
更多关于省电方面的变化，请参见针对休眠和应用待机的优化。

移除Apache HTTP客户端（Apache HTTP Client Removal）
Android 6.0移除了对Apache HTTP客户端的支持。如果你的应用使用了该客户端，并且目标平台是Android 2.3 (API level 9) 或者更高，那么请用HttpURLConnection类来替代。这个API效率更高，因为它使用了透明压缩（transparent compression）和响应缓存（response caching）来减少对网络的占用，并且降低了功耗。如果想继续使用Apache HTTP的接口，就必须先在build.gradle文件中声明以下编译时依赖：

android {
    useLibrary 'org.apache.http.legacy'
}
BoringSSL
Android正在从OpenSSL迁移到BoringSSL库。如果你的应用中使用了Android NDK，那么不要链接任何不属于NDK API的加密库，比如libcrypto.so和libssl.so。这些库不是公共API，有可能在不引起注意的情况下，在不同的发布版和设备之间发生变化，或者崩溃。另外，这些库有可能使你暴露在安全风险面前。所以，应该修改本地代码，通过JNI调用Java的加密API，或者静态链接某个你选择的加密库，而不是链接上述库。

访问硬件标识（Access to Hardware Identifier）
为了给用户提供更好的数据保护，从本次发布开始，Android不再支持应用通过Wi-Fi和蓝牙接口，以编程的方式访问设备本地硬件标识。现在，WifiInfo.getMacAddress()和BluetoothAdapter.getAddress()这两个方法将返回常量02:00:00:00:00:00。

如果想通过蓝牙和Wi-Fi扫描来访问邻近其他设备的硬件标识，应用必须有ACCESS_FIN_LOCATION或者ACCESS_CORSE_LOCATION权限：

WifiManager.getScanResults()
BluetoothDevice.ACTION_FOUND
BluetoothLeScanner.startScan()
注意：如果运行Android 6.0 (API level 23) 的设备启动了一个后台的Wi-Fi或者蓝牙扫描，对外部设备来说，看到的会是从一个随机MAC地址发起的操作。
通知（Notifications）
本次发布移除了Notification.setLatestEventInfo()方法，取而代之的是用Notification.Builder来创建通知。如果要重复更新某个通知，可以重用（reuse）Notification.Builder实例，调用build()来获得更新后的Notification实例。

另外，命令

adb shell dumpsys notification
将不再输出通知的文本。如果想输出通知对象中的文本，需要执行：

adb shell dumpsys notification --noredact
音频管理器的变化（AudioManager Changes）
不再支持通过AudioManager类直接设置特定流的音量或者将其静音，setStreamSolo()方法已被弃用（deprecated）。替代方法是调用requestAudioFocus()方法。类似的，setStreamMute()方法也已被弃用，替代方法是调用adjustStreamVolume()，并传入ADJUST_MUTE或者ADJUST_UNMUTE。

文本选择（Test Selection）
现在，当用户在你的应用中选择文本时，你可以在一个浮动工具栏上显示诸如剪切，复制和粘贴这样的文本选择动作。这里实现的用户交互跟“在单独视图中启用上下文动作栏”一文中介绍的上下文动作栏（contextual action bar）类似。

text-selection.gif

如果想为文本选择实现一个浮动工具栏，需要在现有应用中做以下修改：

在View或者Activity对象中，将ActionMode的调用中从startActionMode(Callback)改为startActionMode(Callback, ActionMode.TYPE_FLOATING)；
将已有的ActionMode.Callback实现改为从ActionMode.Callback2派生；
重写onGetContentRect()方法，提供内容Rect对象（例如文本选择区域的矩形）在视图中的坐标；
如果矩形位置失效，而且这是唯一一个无效的元素，那么就调用invalidateContentRect()方法；
如果你使用了22.2版的Android支持库 ，那么要注意，浮动工具栏不是向后兼容的。默认情况下，appcompat会接管对ActionMode对象的控制，这会使浮动工具栏显示不出来。为了在AppCompatActivity中支持ActionMode，需要先调用getDelegate()，然后在该方法返回的AppCompatDelegate对象上调用setHandleNativeActionModesEnabled()方法，并将输入参数设为false。这个调用会将ActionMode的控制权交还给框架（framework）。在运行Android 6.0 (API level 23) 的设备上，框架可以支持ActionBar或者浮动工具栏模式，而在Android 5.1 (API level 22) 或者更低本的Android上，只支持ActionBar模式。

浏览器书签的变化（Browser Bookmark Changes）
本次发布不再支持全局书签（global bookmarks），android.provider.Browser.getAllBookmarks()和android.provider.Browser.saveBookmark()方法已被移除，READ_HISTORY_BOOKMARKS和WRITE_HISTORY_BOOKMARKS权限也同样被移除。如果你的应用目标平台是Android 6.0 (API level 23) 或者更高，那么就不要通过全局提供器来访问书签，或者使用书签的权限。相反，应用应该自行将书签数据保存在内部。

Android密钥存储的变化（Android Keystore Changes）
随着本次的发布，Android密钥存储提供器不再支持DSA，但仍支持ECDSA。

当安全锁屏被（比如用户或设备管理器）禁用或者重置时，将不再删除那些待机(at rest)时无需加密的密钥。而那些待机时也需要加密的密钥在这种情况下则会被删除。

Wi-Fi和网络的变化（Wi-Fi and Networking Changes）
本次发布在Wi-Fi和网络接口方面，引入了以下这些行为上的变化：

应用现在只能修改本自己创建的WifiConfiguration对象的状态，而不能修改或者删除用户或者其他应用创建的WifiConfiguration对象；
以前，如果有应用通过enableNetwork()，强制设备连接到某个特定的Wi-Fi网络，而且设置了disableAllOthers=true，那么设备将会从其他网络（比如蜂窝数据）断开。而在本次发布中，设备将不再从其他网络断开。如果应用的targetSdkVersion是20或者更低，那么它会被锁定（pinned）到所选择的Wi-Fi网络。如果应用的targetSdkVersion是21或者更高，则要使用多网络API（比如openConnection()，bindSocket()和新的bindProcessToNetwork()方法），来确保该应用的网络流量被发往所选择的网络。
相机服务的变化（Camera Service Changes）
本次发布，将相机服务中对共享资源的访问模型，从之前的“先到先服务”改为“基于进程优先级服务”。服务行为的变化包括：

对相机子系统资源的访问，包括打开和配置相机设备，会基于客户应用进程的“优先级”进行评级。用户可见应用的进程或者位于前台的窗口（activities）通常被赋予更高优先级，以确保其能更可靠地获得和使用相机资源；
低优先级的活动相机客户端，在更高优先级的应用试图使用相机时，会被“驱逐”（evicted）。在已废弃的Camera API中，会调用onError()。而在Camera2 API中，则会调用被驱逐的客户端的onDisconnected()方法；
如果设备上有合适的相机硬件，那么不同的应用进程将可以同时打开并各自使用不同的相机设备。但是，现在相机服务会检测那些可能造成明显的性能恶化，或者降低已打开相机设备功能的多进程访问场景，并禁止这种情况发生。这个变化会导致低优先级的客户端，即使在没有其他应用试图直接访问它的相机设备时也可能被“驱逐”。
修改当前用户，会造成之前用户帐号的应用里的活动相机客户端被驱逐。对相机的访问，仅限于属于当前设备用户的配置。在实际使用中，这意味着比如说一个“访客”帐户，在用户切换到另一个不同的帐户后，不能留下使用了相机子系统的活动进程。
运行时（Runtime）
ART运行时现在正确地实现了newInstance()方法的访问规则。这个变化修正了之前版本中Dalvik没有正确检查访问权限的问题。如果你的应用使用了newInstance()方法，而且想覆盖原有的权限检查，可以调用setAccessible()方法，并将输入参数设为true。如果应用使用了v7 appcompat库或者v7 recyclerview库，则必须将这两个库升级到最新版本。否则，就要升级XML中引用的所有自定义类，确保它们的构造函数都是可访问的。

本次发布还升级了动态链接器（dynamic linker）的行为。现在，动态连接器可以区分库的soname和它的路径（公开bug 6670），并实现了基于soname的查找。之前没有正确设置DT_NEEDED条目（通常设成了构建机器文件系统上的绝对路径）却能正常工作的应用，现在可能会加载失败。

另外，dlopen(3) RTLD_LOCAL标志也被正确地实现了。需要注意的是，由于RTLD_LOCAL是默认值，所以没有显式使用RTLD_LOCAL的dlopen(3)调用都可能受影响（除非应用显式使用了RTLD_GLOBAL标志）。使用RTLD_LOCAL标志加载的符号，对于后续通过dlopen(3)加载的库来说是不可见的（跟DT_NEEDED条目所引用的情况相反）。

在以前版本的Android上，如果应用请求系统加载一个需要代码重定位（text relocations）的共享库，系统会显示一个警告，但仍然允许库被加载。从本次发布开始，如果应用的目标SDK版本是23或者更高，系统将拒绝加载这样的库。在应用中输出dlopen(3)失败的日志，并将dlerror(3)返回的问题描述文本包含在其中，将有助于检测库加载失败的问题。想了解更多对代码重定位的处理方法，请参见这份指南。

APK校验（APK Validation）
现在，平台将对APK进行更严格的校验。如果manifest中声明了某个文件，但在APK中没有这个文件，那么这个APK将被视为已损坏。不管移除了什么内容，APK都必须重新签名。

USB连接（USB Connection）
现在通过USB端口连接设备，默认将是“仅充电”模式。如果想通过USB连接来访问设备和设备上的内容，需要用户显式地为这种交互授权。如果你的应用支持用户通过USB端口与设备进行交互，那么就需要考虑到这种交互需要被显式开启。

Android for Work的变化（Android for Work Changes）
本次发布包含了以下这些Android for Work相关的变化：

私人环境（personal context）里的工作联系人：现在，当用户查看过往通话时，Google的拨号记录会显示工作联系人。将setCrossProfileCallerIdDisabled()设为true可以在Google拨号记录中隐藏工作联系人。只有将setBluetoothContactSharingDisabled()设为false（默认为true），才能通过蓝牙在其他设备上同时显示工作联系人和私人联系人；
移除Wi-Fi配置：通过配置所有者（Profile Owner）添加的Wi-Fi配置（例如，通过调用addNetwork()），在其所属的工作配置被删除时，会被移除；
Wi-Fi配置锁定：由活动的设备所有者（Device Owner）创建的Wi-Fi配置，如果其WIFI_DEVICE_OWNER_CONFIGS_LOCKDOWN不为0，将不能被用户修改或者删除。但用户还是可以创建和修改自己的Wi-Fi配置。而活动设备所有者则有特权，可以编辑或者移除任意Wi-Fi配置——包括哪些不是由其创建的配置；
DevicePolicyManager特定API的行为变化： 
调用setCameraDisabled()方法只会影响调用者自己的camera。在受管理配置（managed profile）中调用该方法，不会影响在主用户（primary user）中运行的相机应用；
另外，现在配置所有者（Profile Owners）也可以像设备所有者那样使用setKeyguardDisabledFeatures()方法；
配置所有者可以设置以下键盘锁限制： 
KEYGUARD_DISABLE_TRUST_AGENT和KEYGUARD_DISABLE_FINGERPRINT，这会影响到当前配置的父用户的键盘锁设置；
KEYGUARD_DISABLE_UNREDACTED_NOTIFICATIONS，这只影响受管理配置里的应用所产生的通知；
createAndInitializeUser()和createUser()方法被废弃；
EXTRA_PROVISIONING_DEVICE_ADMIN_PACKAGE_CHECKSUM现在默认是SHA-256。为了向后兼容，目前仍支持SHA-1，但未来将不再对其支持；
EXTRA_PROVISIONING_DEVICE_ADMIN_SIGNATURE_CHECKSUM现在只支持SHA-256；
存在于Android 6.0 (API level 23) 的设备初始化接口现在已被移除；
移除了EXTRA_PROVISIONING_RESET_PROTECTION_PARAMETERS，因此不能通过编程，让NFC bump provisioning解锁一个出厂时被设为保护的设备；
现在，通过NFC连接受管理设备时，可以用EXTRA_PROVISIONING_ADMIN_EXTRAS_BUNDLE来将数据传递到设备所有者应用；
Android for Work的接口针对M运行时权限做了优化，包括工作配置，助手层（assist layer）以及其他一些改动。新的DevicePolicyManager权限接口不影响M之前的应用；
如果设置流程是通过ACTION_PROVISION_MANAGED_DEVICE Intent发起的，当用户从其中的同步部分（synchronous part）退出时，系统将返回RESULT_CANCELED结果码
其他API的变化： 
数据使用：android.app.usage.NetworkUsageStats类被改名为NetworkStats；
全局设置的变化： 
不能通过setGlobalSettings()修改以下设置： 
BLUETOOTH_ON
DEVELOPMENT_SETTINGS_ENABLED
MODE_RINGER
NETWORK_PREFERENCE
WIFI_ON
可以通过setGlobalSettings()修改一下设置： 
WIFI_DEVICE_OWNER_CONFIGS_LOCKDOWN
Pre: No Post

Next: No Post
