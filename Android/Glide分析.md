###### Glide����ԭ��

Ĭ��ֻ����ת�������Resultͼ��������ԭͼ������ʹ��DiskCacheStrategyȥ����<br/>
1��Glide.with()�����Ĳ������Դ�Activity��FragmentActivity��Fragment��Context����host���Ͳ�ͬ��ȡ��ͬ��RequestManagerRetriever.get()ȥ��ȡÿ��host�ϰ󶨵�RequestManager������������߳��з������������õ�Application��Ϊhost���봫Activityʱ����һ�λ�ȡRequestManagerʱ�����ȼ�鵱ǰFragmentManager���Ƿ�����RequestManagerFragment����������ڣ���ᴴ��һ��RequestManagerFragment��ӵ���Activity�У�Ȼ�����fragment����RequestManager�����������ֱ�ӷ���
ͬһ��host��Ӧ��Ψһһ��RequestManager��RequestManager�����Լ�������Request��֧����ͣ�Ϳ�ʼRequest<br/>
2����θ�host���������ڹ���������<br/>
�ڵ�ǰhost��addһ��Fragment����Fragment�����һ����֮��Ӧ��RequestManager����manager�����������host����ͼƬ������(��ͣ��ȡ�������Ե�)<br/>
3������ԭ��<br/>
�ڴ滺�棺<br/>
Result(ת�����ͼ)��Key���ɹ���<br/>
	�����ͼƬ��С��10����Ϣ����<br/>
Original(ԭͼ)��Key���ɹ���<br/>
	��ʵ����ͼƬ��url<br/>
�ڴ滺���Ϊ������<br/>
	һ������Ϊ��ʹ�õ�ͼƬ����LruResourceCache������LRU�㷨��<br/>
	��һ����HashMap<Key, WeakReference<EngineResource<?>>> ActiveResourceCache���棬��������������ڱ�ʹ�õ�ͼƬ<br/>
LruResourceCache��<br/>
	��ʾ�������õ�ͼƬ���棬û�б�ʹ�ã������������л���󣬻�Ѹ�Resource�Ƴ�������һ���ֻ�ǵ���get()����ȡ���Ƴ�����ӵ�ActiveResourceCache��<br/>
ActiveResourceCache��<br/>
	��ʾ���ڱ�ʹ�õ�ͼƬ���棬��ͨ��HashMap<Key, WeakReference<EngineResource>>��ʵ�ֵģ�EngineResource.acquire()�������ڼ�¼���ü����ģ�ʹ��ActiveResourceCache����������ʹ���е�ͼƬ�����Ա�����ЩͼƬ���ᱻLruCache�㷨���յ�����EngineResource��acquireΪ0ʱ�����ȻὫ����ͼƬ��activeResources���Ƴ���Ȼ���ٽ���put��LruResourceCache���У�����Ҳ��ʵ��������ʹ���е�ͼƬʹ�������������л��棬����ʹ���е�ͼƬʹ��LruCache�����л���Ĺ��ܡ�<br/>
��ȡ��������̣�<br/>
Check the memory cache and provide the cached resource if present<br/>
Check the current set of actively used resources and return the active resource if present<br/>
Check the current set of in progress loads and add the cb to the in progress load if present Start a new load<br/>
���̻��棺<br/>
����DiskLruCacheʵ�֣��ȵ���decodeResultFromCache()����ת������ͼ����������򷵻أ������ٴ�decodeSourceFromCache()����ԭͼ��������У��򷵻أ���������������<br/>

###### Glide����

1����֪Activity��Fragment���������ڣ�ͨ����Activity�����һ�����ص�Fragment������Fragment�����һ�����ص�ChildFragment��ʵ�ּ����������������ڣ���������������ͣ�����·���ͼƬ��������ͬʱҲ������Activity��onTrimMemory()��onLowMemory()�����������ͷ��ڴ滺�棬��Ҫ���ͷ��ڴ浱��δ��ʹ�õ�LruResourceCache��LruBitmapPool<br/>
2����Ĭ�ϵĴ��̻��������ֻ����Result��Ҳ����ֻ����ת����Ľ��ͼ��������ԭͼ������һ��Ϊͬһ��ImageView�������ͼƬʱ����Ϊ�ߴ��url��û�䣬ֱ�ӴӴ��̼�����ʾ�ͺ��ˣ�����Ҫ���ߴ�ת�������ͬһ��ͼƬurlҪ�����ص������ߴ粻ͬ��ImageView�ϣ����ͼƬҪ�������ηֱ�ת����������ͬ�ĳߴ��ٱ��浽�����С����Ե���Glide�Ĵ��̻������ΪDiskCacheStrategy.ALL��Source��Result<br/>
3��֧��Gif 