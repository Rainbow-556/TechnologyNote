###### 一、静态广播(在清单文件注册的广播)

在系统启动、应用安装、卸载的时候，PackageManagerService会扫描手机中所有安装的App的清单文件中静态注册的Broadcast，并把它保存早PMS中

###### 二、动态广播

是通过ContextImpl.registerBroadcast()方法去调用ActivityManagerNative去注册的，广播的Filter规则信息保存在AMS中，App进程中保存与Filter对应的BroadcastReceiver对象，在发送广播时，是调用ActivityThread的H Handler去处理的回调，所以广播的onReceive()方法是运行在主线程中的 