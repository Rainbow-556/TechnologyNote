相关链接：https: //blog.csdn.net/a553181867/article/details/51583060

###### 一、requestLayout()，请求重新执行一遍绘制的三大流程

子View调用requestLayout方法，会标记当前View和它的父View，同时逐层向上提交和标记，直到ViewRootImpl处理该事件，ViewRootImpl会调用三大流程，从measure开始，
对于每一个含有标记位的view及其子View都会进行测量、布局、绘制，注意并不会对整个View树执行三大流程
```java
void scheduleTraversals() {
	if (!mTraversalScheduled) {
		mTraversalScheduled = true;
		// 往MessageQueue中post一个同步屏障消息，用于挡住在之后被添加的同步消息的执行，但是不会影响到异步消息，调用Message.setAsynchronous()设置同步和异步
		mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
		// 在Choreographer注册下一帧到来时的监听，执行测量、布局、绘制流程，同时在监听中移除同步屏障消息
		// 这个mTraversalRunnable中会执行绘制三大流程，如果View是第一次添加到Window中则会调用View.dispatchAttachedToWindow()
		// View.dispatchAttachedToWindow()方法中会把之前View.post()添加的消息取出执行，这也就是为什么post()中可以获取View的宽高，因为是在View.post()在View还未被添加到Window上之前，
		// View会把该消息储存在自身成员变量HandlerActionQueue(储存结构为数组)中，在执行绘制三大流程之后，才会取出来执行并移除
		mChoreographer.postCallback(
			Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
		if (!mUnbufferedInputDispatch) {
			scheduleConsumeBatchedInput();
		}
		notifyRendererOfFramePending();
		pokeDrawLockIfNeeded();
	}
}
```
###### 二、invalidate()

当子View调用了invalidate方法后，会为该View添加一个标记位，同时不断向父容器请求刷新，父容器通过计算得出自身需要重绘的区域，直到传递到ViewRootImpl中，最终触发performTraversals方法，进行开始View树重绘流程(只绘制需要重绘的视图)

###### 三、postInvalidate()，当View已经添加到Window中，才会执行刷新操作，最终是通过Handler的方式调用View.invalidate()执行刷新，所以能够在子线程中调用
```java
public void postInvalidateDelayed(long delayMilliseconds) {
	// We try only with the AttachInfo because there's no point in invalidating
	// if we are not attached to our window
	final AttachInfo attachInfo = mAttachInfo;
	if (attachInfo != null) {
		attachInfo.mViewRootImpl.dispatchInvalidateDelayed(this, delayMilliseconds);
	}
}
```
###### 四、子线程为什么不能更新View？

为了保持界面刷新的同步<br/>
在View没有添加到Window之前，是可以在子线程创建和初始化View，只是在View添加到Window的时候，会保存当前的线程对象(也就是主线程，因为是在主线程中把View添加到Window的)，而在之后更新View时，回调用View.scheduleTraversals()方法，这个方法里检查了此时的调用线程是否与之前保存的线程是否一致，不一致则抛出异常