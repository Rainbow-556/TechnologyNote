相关链接：https://github.com/OpenDevTeam/OpenBox/blob/master/topic/%5BAndroid%E5%BA%95%E5%B1%82%E8%A7%A3%E6%9E%90%5D%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3AMS%E6%A1%86%E6%9E%B6.md <br/>

1、WindowManager负责调用系统的WindowManagerService，而WindowManager的所有操作代理给了WindowManagerGlobal类去处理<br/>
2、ActivityThread负责单向调用ActivityManagerService的服务，而AMS通过客户端的ApplicationThread再发送Handler消息给App，使用以上的方式来实现了互相通信的模式<br/>
3、H类的performLaunchActivity()调用Activity.attach()，而attach()方法中创建PhoneWindow，setContentView()方法把布局加载到DecorView中，此时Window还没有被添加到WMS，在H.performResumeActivity()中调用Activity.makeVisible()才真正的创建ViewRootImpl把DecorView和PhoneWindow关联起来，并通过WindowManager把DecorView和ViewRootImpl保存在WindowManagerGlobal的集合中