###### 一、RecyclerView

1、缓存机制<br/>
缓存的是ViewHolder，由内部类Recycler控制，Recycler中的RecyclerViewPool保存的是被重置状态的ViewHolder，使用的是SparseArray结构，Key为ItemType，Value为ArrayList<ViewHolder>列表Recycler中的ArrayList<ViewHolder> mCachedViews集合保存的是最近被回收的ViewHolder，默认大小为2，mCachedViews中保存的ViewHolder都是信息齐全的，复用时不需要重新绑定数据，复用时需要验证Holder的有效性(position等)
当存满时，会把第0个元素移除并添加到RecyclerViewPool中<br/>
复用时：先去mCachedViews集合中根据adapterPosition查找ViewHolder，找到直接返回，不需要重新绑定数据(这是一个优化点)，没找到则去RecyclerViewPool中找，找到则重新绑定数据，没找到则重新创建ViewHolder。RecyclerViewPool可以给多个RecyclerView共享<br/>
2、局部更新<br/>
adapter.notifyItemChanged(int position, Object payload)，会调用带有List<Object> payloads的onBindViewHolder(VH holder, int position, List<Object> payloads)，如果payloads为空，则要全部刷新，如果不空则根据业务需求进行该ViewHolder的局部刷新<br/>
3、如果布局的<br/>

4、DiffUtil原理，继承DiffUtil.Callback传入新旧两个list数据集，DiffUtil.calculateDiff()调用之后，新旧两个list的差异会合并到新list中，然后改变Adapter的数据源，把DiffResult更新到Adapter上去，使用示例：
```java
List oldList = mAdapter.getData();
DiffResult result = DiffUtil.calculateDiff(new MyCallback(oldList, newList));
mAdapter.setData(newList);
result.dispatchUpdatesTo(mAdapter);
```
###### 二、ListView

1、缓存机制<br/>
缓存的是View，由RecycleBin控制。在第一次layout()之前会fillFromTop生成一整屏的View保存在View[] mActiveViews数组中，第二次layout时会把ListView的所有子View移除，并从mActiveViews快速取出View添加到ListView，如果mActiveViews中缓存的view不够，则会调用Adapter.getView()来获取，ListView每次layout()时，都会移除所有子View，并在layout之后剩余的View都会被移除并添加到ArrayList<View>[] mScrapViews中。<br/>
ArrayList<View>[] mScrapViews是一个根据ViewType作为index来保存每个ViewType的已不在屏幕的View，所以getItemViewType()返回的值必须是从0开始到getViewTypeCount() - 1之间的连续数值，否则无法根据ViewType找到对应的ArrayList<View><br/>

2、局部更新<br/>
保存所有ViewHolder，holder中绑定了一个数据，通过把最新的数据传到adapter去局部更新，然后遍历holder集合去匹配该数据(数据一般带有id)，若找到直接更新holder的View<br/>

###### 三、ListView和RecyclerView的内存优化

1、如果itemView包含图片，因为ListView的itemView每次进入到屏幕都会调用getView()，在ListView中可以在ImageView.onDetchFromWindow()中把ImageView的图片引用置为空，如果是在RecyclerView中，由于RV中缓存的是ViewHolder，在复用ViewHolder时，可能不会重新绑定数据，所以可以在Adapter.onViewRecycled()中把图片引用置为空<br/>
2、滚动时不加载图片<br/>
3、ListView不要在getView()中设置view的点击事件，可以在LV的ViewHolder中绑定一个数据，在ViewHolder的构造方法中设置View的点击事件，这样可以避免每次getView()时创建OnClickListener对象，RecyclerView同样也可以这样做<br/>
4、绑定数据的方法中尽量不做复杂的数据处理和创建对象