- Scroller.startScroll(int startX, int startY, int dx, int dy, int duration)
- Scroller.fling(int startX, int startY, int velocityX, int velocityY, int minX, int maxX, int minY, int maxY)

```java
private Scroller mScroller = new Scroller(context);
public void zoomIn() {
     // Revert any animation currently in progress
     mScroller.forceFinished(true);
     // Start scrolling by providing a starting point and
     // the distance to travel
     // 该方法只是设置滚动的参数，并未立即开始滚动
     mScroller.startScroll(0, 0, 100, 0);
     // Invalidate to request a redraw
     // 该方法会触发computeScroll()，而在computeScroll()中会继续判断没有滚动完成则会继续调用invalidate()，直到滚动结束
     invalidate();
}

// View.computeScroll()
public void computeScroll() {
    // Scroller.computeScrollOffset()是根据时间的流逝来计算当前currX和currY，同时可以设置Interpolator
    if (mScroller.computeScrollOffset()) {
     	// Get current x and y positions
     	int currX = mScroller.getCurrX();
     	int currY = mScroller.getCurrY();
        scrollTo(currX, currY);
        invalidate();
 	}
}
```

