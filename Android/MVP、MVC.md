###### MVP概述

- Model

  按照不同的业务模块分包，网络请求的数据源可用Retrofit和RxJava配合使用

- View

  提供生命周期回调、UI事件给Presenter

- Presenter

  根据View的生命周期回调和UI事件来执行各种业务逻辑，从Model获取数据，再把数据设置给View。

###### 优化

当一个界面非常复杂时，View和Presenter的代码同时也会变得臃肿，可以考虑把原有的分成几个子View和子Presenter来处理