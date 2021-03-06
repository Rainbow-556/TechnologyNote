###### Glide基本原理

默认只缓存转换过后的Result图，不缓存原图，可以使用DiskCacheStrategy去控制<br/>
1、Glide.with()方法的参数可以传Activity，FragmentActivity，Fragment，Context根据host类型不同获取不同的RequestManagerRetriever.get()去获取每个host上绑定的RequestManager，如果是在子线程中发起的请求，则会拿到Application作为host传入传Activity时，第一次获取RequestManager时，会先检查当前FragmentManager中是否已有RequestManagerFragment，如果不存在，则会创建一个RequestManagerFragment添加到该Activity中，然后给该fragment保存RequestManager，如果存在则直接返回
同一个host对应着唯一一个RequestManager，RequestManager管理自己的所有Request，支持暂停和开始Request<br/>
2、如何跟host的生命周期关联起来？<br/>
在当前host下add一个Fragment，该Fragment会持有一个与之对应的RequestManager，该manager是用来管理该host所有图片的请求(暂停，取消，重试等)<br/>
3、缓存原理<br/>
内存缓存：<br/>
Result(转换后的图)的Key生成规则：<br/>
	会根据图片大小等10种信息生成<br/>
Original(原图)的Key生成规则：<br/>
	其实就是图片的url<br/>
内存缓存分为两级：<br/>
	一个闲置为被使用的图片缓存LruResourceCache，基于LRU算法的<br/>
	另一个是HashMap<Key, WeakReference<EngineResource<?>>> ActiveResourceCache缓存，缓存的是所有正在被使用的图片<br/>
LruResourceCache：<br/>
	表示正在闲置的图片缓存，没有被使用，当在这里命中缓存后，会把该Resource移除，不像一般的只是调用get()来获取，移除后并添加到ActiveResourceCache中<br/>
ActiveResourceCache：<br/>
	表示正在被使用的图片缓存，是通过HashMap<Key, WeakReference<EngineResource>>来实现的，EngineResource.acquire()方法用于记录引用计数的，使用ActiveResourceCache来缓存正在使用中的图片，可以保护这些图片不会被LruCache算法回收掉。当EngineResource的acquire为0时，首先会将缓存图片从activeResources中移除，然后再将它put到LruResourceCache当中，这样也就实现了正在使用中的图片使用弱引用来进行缓存，不在使用中的图片使用LruCache来进行缓存的功能。<br/>
获取缓存的流程：<br/>
Check the memory cache and provide the cached resource if present<br/>
Check the current set of actively used resources and return the active resource if present<br/>
Check the current set of in progress loads and add the cb to the in progress load if present Start a new load<br/>
磁盘缓存：<br/>
基于DiskLruCache实现，先调用decodeResultFromCache()查找转换过的图，如果命中则返回，否则再从decodeSourceFromCache()查找原图，如果命中，则返回，否则发送网络请求<br/>

###### Glide优势

1、感知Activity和Fragment的生命周期，通过在Activity中添加一个隐藏的Fragment或者在Fragment中添加一个隐藏的ChildFragment来实现监听宿主的生命周期，根据生命周期暂停和重新发起图片加载请求，同时也监听了Activity的onTrimMemory()和onLowMemory()两个方法来释放内存缓存，主要是释放内存当中未被使用的LruResourceCache和LruBitmapPool<br/>
2、在默认的磁盘缓存策略是只缓存Result，也就是只缓存转换后的结果图，而不是原图，在下一次为同一个ImageView加载这个图片时，因为尺寸和url都没变，直接从磁盘加载显示就好了，不需要做尺寸转换，如果同一个图片url要被加载到两个尺寸不同的ImageView上，这个图片要下载两次分别转换成两个不同的尺寸再保存到磁盘中。可以调整Glide的磁盘缓存策略为DiskCacheStrategy.ALL、Source、Result<br/>
3、支持Gif 