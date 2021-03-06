参考：
https://developer.android.com/topic/performance/graphics/manage-memory<br/>

###### 一、计算inSampleSize，2的幂
```java
public static int calculateInSampleSize(BitmapFactory.Options options, int reqWidth, int reqHeight) {
    // Raw height and width of image
    final int height = options.outHeight;
    final int width = options.outWidth;
    int inSampleSize = 1;
    if (height > reqHeight || width > reqWidth) {

        final int halfHeight = height / 2;
        final int halfWidth = width / 2;
    
        // Calculate the largest inSampleSize value that is a power of 2 and keeps both
        // height and width larger than the requested height and width.
        while ((halfHeight / inSampleSize) >= reqHeight
                && (halfWidth / inSampleSize) >= reqWidth) {
            inSampleSize *= 2;
        }
    }
    return inSampleSize;
}
```
###### 二、常规特点

1、Android 3.0之前，Bitmap的像素数据(pixel data)是储存在native内存(native memory)中，而Bitmap本身是储存在Dalvik heap中，<br/>
Android 3.0~7.1 Bitmap的像素数据和Bitmap一样都储存在Dalvik heap中，而8.0及以后又把Bitmap的像素数据改为储存在native heap中<br/>
2、inBitmap，复用已有Bitmap的内存空间，4.4之前必须
```java
static boolean canUseForInBitmap(Bitmap candidate, BitmapFactory.Options targetOptions) {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
        // From Android 4.4 (KitKat) onward we can re-use if the byte size of
        // the new bitmap is smaller than the reusable bitmap candidate
        // allocation byte count.
        int width = targetOptions.outWidth / targetOptions.inSampleSize;
        int height = targetOptions.outHeight / targetOptions.inSampleSize;
        int byteCount = width * height * getBytesPerPixel(candidate.getConfig());
        return byteCount <= candidate.getAllocationByteCount();
    }
    // On earlier versions, the dimensions must match exactly and the inSampleSize must be 1
    return candidate.getWidth() == targetOptions.outWidth
            && candidate.getHeight() == targetOptions.outHeight
            && targetOptions.inSampleSize == 1;
}
/**
 * A helper function to return the byte usage per pixel of a bitmap based on its configuration.
 */
static int getBytesPerPixel(Config config) {
    if (config == Config.ARGB_8888) {
        return 4;
    } else if (config == Config.RGB_565) {
        return 2;
    } else if (config == Config.ARGB_4444) {
        return 2;
    } else if (config == Config.ALPHA_8) {
        return 1;
    }
    return 1;
}
```
3、Config.ARGB_8888格式，一个像素占用4个字节。Config.RGB_565格式，一个像素占用2个字节<br/>

###### 三、加载大图使用BitmapRegionDecoder，可以指定加载该图片的某个Rect矩形区域