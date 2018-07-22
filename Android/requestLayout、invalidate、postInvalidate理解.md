������ӣ�https: //blog.csdn.net/a553181867/article/details/51583060

###### һ��requestLayout()����������ִ��һ����Ƶ���������

��View����requestLayout���������ǵ�ǰView�����ĸ�View��ͬʱ��������ύ�ͱ�ǣ�ֱ��ViewRootImpl������¼���ViewRootImpl������������̣���measure��ʼ��
����ÿһ�����б��λ��view������View������в��������֡����ƣ�ע�Ⲣ���������View��ִ����������
```java
void scheduleTraversals() {
	if (!mTraversalScheduled) {
		mTraversalScheduled = true;
		// ��MessageQueue��postһ��ͬ��������Ϣ�����ڵ�ס��֮����ӵ�ͬ����Ϣ��ִ�У����ǲ���Ӱ�쵽�첽��Ϣ������Message.setAsynchronous()����ͬ�����첽
		mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
		// ��Choreographerע����һ֡����ʱ�ļ�����ִ�в��������֡��������̣�ͬʱ�ڼ������Ƴ�ͬ��������Ϣ
		// ���mTraversalRunnable�л�ִ�л����������̣����View�ǵ�һ����ӵ�Window��������View.dispatchAttachedToWindow()
		// View.dispatchAttachedToWindow()�����л��֮ǰView.post()��ӵ���Ϣȡ��ִ�У���Ҳ����Ϊʲôpost()�п��Ի�ȡView�Ŀ�ߣ���Ϊ����View.post()��View��δ����ӵ�Window��֮ǰ��
		// View��Ѹ���Ϣ�����������Ա����HandlerActionQueue(����ṹΪ����)�У���ִ�л�����������֮�󣬲Ż�ȡ����ִ�в��Ƴ�
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
###### ����invalidate()

����View������invalidate�����󣬻�Ϊ��View���һ�����λ��ͬʱ��������������ˢ�£�������ͨ������ó�������Ҫ�ػ������ֱ�����ݵ�ViewRootImpl�У����մ���performTraversals���������п�ʼView���ػ�����(ֻ������Ҫ�ػ����ͼ)

###### ����postInvalidate()����View�Ѿ���ӵ�Window�У��Ż�ִ��ˢ�²�����������ͨ��Handler�ķ�ʽ����View.invalidate()ִ��ˢ�£������ܹ������߳��е���
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
###### �ġ����߳�Ϊʲô���ܸ���View��

Ϊ�˱��ֽ���ˢ�µ�ͬ��<br/>
��Viewû����ӵ�Window֮ǰ���ǿ��������̴߳����ͳ�ʼ��View��ֻ����View��ӵ�Window��ʱ�򣬻ᱣ�浱ǰ���̶߳���(Ҳ�������̣߳���Ϊ�������߳��а�View��ӵ�Window��)������֮�����Viewʱ���ص���View.scheduleTraversals()������������������˴�ʱ�ĵ����߳��Ƿ���֮ǰ������߳��Ƿ�һ�£���һ�����׳��쳣