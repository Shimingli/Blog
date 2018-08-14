---
title: 基于Glide4.7.1二次封装
date: 2018-04-22 16:26:48
tags: 基于Glide4.7.1二次封装
---
### [github](https://github.com/Shimingli/ImageLoader)

![image.png](https://upload-images.jianshu.io/upload_images/5363507-285dae46159d767e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!--  more  -->
使用方法
```
     String url="https://upload-images.jianshu.io/upload_images/5363507-476c8bb17b124d22.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240";
        //        //圆形图片
        ImageLoaderV4.getInstance().displayCircleImage(this, "http://imgsrc.baidu.com/imgad/pic/item/267f9e2f07082838b5168c32b299a9014c08f1f9.jpg", mImageView_1, R.mipmap.ic_launcher_round);
        //圆角图片
        ImageLoaderV4.getInstance().displayRoundImage(this, "http://imgsrc.baidu.com/imgad/pic/item/267f9e2f07082838b5168c32b299a9014c08f1f9.jpg", mImageView_2, R.mipmap.ic_launcher_round, 40);
        //模糊图片
        ImageLoaderV4.getInstance().displayBlurImage(this, "http://imgsrc.baidu.com/imgad/pic/item/267f9e2f07082838b5168c32b299a9014c08f1f9.jpg", mImageView_3, R.mipmap.ic_launcher_round, 10);

        //本地图片 不做处理
        ImageLoaderV4.getInstance().displayImageInResource(this, R.mipmap.test, mImageView_4);
        // TODO: 2018/4/20  以下的三种不介意使用，需要解决一些问题  start
        //本地图片，模糊处理
        ImageLoaderV4.getInstance().displayImageInResource(this, R.mipmap.test, mImageView_5, new BlurBitmapTranformation( 200));
        //本地图片，裁圆角处理
        ImageLoaderV4.getInstance().displayImageInResource(this, R.mipmap.test, mImageView_6, new GlideCircleTransformation());
        //四周倒角处理
        ImageLoaderV4.getInstance().displayImageInResource(this, R.mipmap.test, mImageView_7, new RoundBitmapTranformation( 40));
        // TODO: 2018/4/20  以上的三种不介意使用，需要解决一些问题  start


        //使用的是另外一种方法，指定传入的那种的方法 ,还可以不断的扩展，不断的扩展，这是Gilded提供的一些操作，牛逼 ，牛逼 这是对本地图片的操作
        ImageLoaderV4.getInstance().displayImageInResourceTransform(this, R.mipmap.test, mImageView_8, new CenterInside(), R.mipmap.test);
        ImageLoaderV4.getInstance().displayImageInResourceTransform(this, R.mipmap.test, mImageView_9, new CircleCrop(), R.mipmap.test);
        ImageLoaderV4.getInstance().displayImageInResourceTransform(this, R.mipmap.test, mImageView_10, new FitCenter(), R.mipmap.test);
        ImageLoaderV4.getInstance().displayImageInResourceTransform(this, R.mipmap.test, mImageView_11, new RoundedCorners(10), R.mipmap.test);

       //对网络图片的操作
        ImageLoaderV4.getInstance().displayImageByNet(this,url,mImageView_12, R.mipmap.test,new CenterInside());
        ImageLoaderV4.getInstance().displayImageByNet(this,url,mImageView_13, R.mipmap.test,new CircleCrop());
        ImageLoaderV4.getInstance().displayImageByNet(this,url,mImageView_14, R.mipmap.test,new FitCenter());
        ImageLoaderV4.getInstance().displayImageByNet(this,url,mImageView_15, R.mipmap.test,new RoundedCorners(10));


```

实现效果：监听图片的下载进度,注意事项需要在监听返回键的时候，取消请求
```
 ImageLoaderV4.getInstance().disPlayImageProgress(this, "http://img.zcool.cn/community/0142135541fe180000019ae9b8cf86.jpg@1280w_1l_2o_100sh.png", mImageView_8, R.mipmap.test, R.mipmap.test, new OnGlideImageViewListener() {
            @Override
            public void onProgress(int percent, boolean isDone, GlideException exception) {
                System.out.println("shiming percent="+percent);
                System.out.println("shiming isDone="+isDone);
                mProgress.setText("我在主线程，进度是多少=="+percent+"%");

                if (isDone){
                    mProgress.setText("我在主线程，进度是多少==100%");
                }
            }
        });


        ImageLoaderV4.getInstance().disPlayImageProgressByOnProgressListener(this, "http://img.zcool.cn/community/0142135541fe180000019ae9b8cf86.jpg@1280w_1l_2o_100sh.png", mImageView_7, R.mipmap.test, R.mipmap.test, new OnProgressListener() {
            @Override
            public void onProgress(String imageUrl, long bytesRead, long totalBytes, boolean isDone, GlideException exception) {
                System.out.println("shiming bytesRead="+bytesRead);
                System.out.println("shiming totalBytes="+totalBytes);
                mProgress7.setText("我在主线程，进度是多少==+bytesRead"+bytesRead+"totalBytes="+totalBytes);

                if (isDone){
                    mProgress7.setText("我在主线程，进度是多少==100%");
                }
            }
        });
```
![image.png](https://upload-images.jianshu.io/upload_images/5363507-986fb8ede4e9e455.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/5363507-dbb259f34f2bcc35.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
使用内部提供的图片处理
![image.png](https://upload-images.jianshu.io/upload_images/5363507-24f9ce31041e0bdc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
加入依赖
```
 compile 'com.github.bumptech.glide:glide:4.7.1'
 annotationProcessor 'com.github.bumptech.glide:compiler:4.7.1'
 compile "com.github.bumptech.glide:okhttp3-integration:4.5.0"
```
构建AppGlideModule
![image.png](https://upload-images.jianshu.io/upload_images/5363507-305073f5732aa42f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
初始化registerComponents()方法，使用okhttp加入拦截器
```
  /**
     OkHttp 是一个底层网络库(相较于 Cronet 或 Volley 而言)，尽管它也包含了 SPDY 的支持。
     OkHttp 与 Glide 一起使用可以提供可靠的性能，并且在加载图片时通常比 Volley 产生的垃圾要少。
     对于那些想要使用比 Android 提供的 HttpUrlConnection 更 nice 的 API，
      或者想确保网络层代码不依赖于 app 安装的设备上 Android OS 版本的应用，OkHttp 是一个合理的选择。
      如果你已经在 app 中某个地方使用了 OkHttp ，这也是选择继续为 Glide 使用 OkHttp 的一个很好的理由，就像选择其他网络库一样。
     添加 OkHttp 集成库的 Gradle 依赖将使 Glide 自动开始使用 OkHttp 来加载所有来自 http 和 https URL 的图片
     * @param context
     * @param glide
     * @param registry
     */
    @Override
    public void registerComponents(Context context, Glide glide, Registry registry) {
        super.registerComponents(context, glide, registry);
        registry.replace(GlideUrl.class, InputStream.class, new OkHttpUrlLoader.Factory(ProgressManager.getOkHttpClient()));
    }
```

 默认情况下，Glide使用 LruResourceCache ， 这是 MemoryCache 接口的一个缺省实现，使用固定大小的内存和 LRU 算法。 LruResourceCache 的大小由 Glide 的 MemorySizeCalculator 类来决定，这个类主要关注设备的内存类型，设备 RAM 大小，以及屏幕分辨率。应用程序可以自定义 MemoryCache 的大小， 具体是在它们的 AppGlideModule 中使用applyOptions(Context, GlideBuilder) 方法配置 MemorySizeCalculator
```
   //Glide会自动合理分配内存缓存，但是也可以自己手动分配。
//    @Override
//    public void applyOptions(@NonNull Context context, @NonNull GlideBuilder builder) {
//        super.applyOptions(context, builder);
//        //1、setMemoryCacheScreens设置MemoryCache应该能够容纳的像素值的设备屏幕数，
//        // 说白了就是缓存多少屏图片，默认值是2。
//        //todo 建议不要设置，使用glide 默认
////        MemorySizeCalculator calculator = new MemorySizeCalculator.Builder(context)
////                .setMemoryCacheScreens(2)
////                .build();
////        builder.setMemoryCache(new LruResourceCache(calculator.getMemoryCacheSize()));
//
//        //2。也可以直接覆写缓存大小：todo 建议不要设置，使用glide 默认
////        int memoryCacheSizeBytes = 1024 * 1024 * 20; // 20mb
////        builder.setMemoryCache(new LruResourceCache(memoryCacheSizeBytes));
////        3.甚至可以提供自己的 MemoryCache 实现：
////        builder.setMemoryCache(new YourAppMemoryCacheImpl());
//    }
```

Bitmap 池Glide 使用 LruBitmapPool 作为默认的 BitmapPool  LruBitmapPool 是一个内存中的固定大小的 BitmapPool，使用 LRU 算法清理。默认大小基于设备的分辨率和密度，同时也考虑内存类和 isLowRamDevice 的返回值。 具体的计算通过 Glide 的 MemorySizeCalculator 来完成，与 Glide 的 MemoryCache 的大小检测方法相似。
 
```
//    @Override
//    public void applyOptions(@NonNull Context context, @NonNull GlideBuilder builder) {
//        super.applyOptions(context, builder);
//    //1 .应用可以在它们的 AppGlideModule 中定制 [BitmapPool] 的尺寸，
//        // 使用 applyOptions(Context, GlideBuilder) 方法并配置 MemorySizeCalculator
////        MemorySizeCalculator calculator = new MemorySizeCalculator.Builder(context)
////                .setBitmapPoolScreens(3)
////                .build();
////        builder.setBitmapPool(new LruBitmapPool(calculator.getBitmapPoolSize()));
//
//        //2、应用也可以直接复写这个池的大小：
//
////        int bitmapPoolSizeBytes = 1024 * 1024 * 30; // 30mb
////        builder.setBitmapPool(new LruBitmapPool(bitmapPoolSizeBytes));
//
//
//        //3、甚至可以提供 BitmapPool 的完全自定义实现：
//
////        builder.setBitmapPool(new YourAppBitmapPoolImpl());
//    }
```

磁盘缓存Glide 使用 DiskLruCacheWrapper 作为默认的 磁盘缓存 . DiskLruCacheWrapper 是一个使用 LRU 算法的固定大小的磁盘缓存。默认磁盘大小为 250 MB ， 位置是在应用的 缓存文件夹 中的一个 特定目录 。

```
  @Override
    public void applyOptions(@NonNull Context context, @NonNull GlideBuilder builder) {
        super.applyOptions(context, builder);
        //假如应用程序展示的媒体内容是公开的（从无授权机制的网站上加载，或搜索引擎等），
        //那么应用可以将这个缓存位置改到外部存储：
//        builder.setDiskCache(new ExternalDiskCacheFactory(context));


         //2、无论使用内部或外部磁盘缓存，应用程序都可以改变磁盘缓存的大小：
//        int diskCacheSizeBytes = 1024 *1024 *100;  //100 MB
//        builder.setDiskCache(new InternalDiskCacheFactory(context, diskCacheSizeBytes));


        //3、应用程序还可以改变缓存文件夹在外存或内存上的名字：
//        int diskCacheSizeBytes = 1024 * 1024 *100; // 100 MB
//        builder.setDiskCache(new InternalDiskCacheFactory(context, cacheFolderName, diskCacheSizeBytes));


        //4、应用程序还可以自行选择 DiskCache 接口的实现，并提供自己的 DiskCache.Factory 来创建缓存。
        // Glide 使用一个工厂接口来在后台线程中打开 磁盘缓存 ，
        // 这样方便缓存做诸如检查路径存在性等的IO操作而不用触发 严格模式 。
//        builder.setDiskCache(new DiskCache.Factory() {
//            @Override
//            public DiskCache build() {
//                return new YourAppCustomDiskCache();
//            }
//        });
        
    }
```
```
/**
 * author： Created by shiming on 2018/4/20 15:13
 * mailbox：lamshiming@sina.com
 磁盘缓存
 Glide 使用 DiskLruCacheWrapper 作为默认的 磁盘缓存 。
 DiskLruCacheWrapper 是一个使用 LRU 算法的固定大小的磁盘缓存。默认磁盘大小为 250 MB ，
 位置是在应用的 缓存文件夹 中的一个 特定目录 。
 假如应用程序展示的媒体内容是公开的（从无授权机制的网站上加载，或搜索引擎等），
 那么应用可以将这个缓存位置改到外部存储：
 */

class ExternalDiskCacheFactory implements DiskCache.Factory {
    public ExternalDiskCacheFactory(Context context) {
    }

    @Nullable
    @Override
    public DiskCache build() {
        return null;
    }
}
```

为了维持对 Glide v3 的 GlideModules 的向后兼容性， Glide 仍然会解析应用程序和所有被包含的库中的 AndroidManifest.xml 文件， 并包含在这些清单中列出的旧 GlideModules 模块类。如果你已经迁移到 Glide v4 的 AppGlideModule 和 LibraryGlideModule ，你可以完全禁用清单解析。 这样可以改善 Glide 的初始启动时间，并避免尝试解析元数据时的一些潜在问题。要禁用清单解析， 请在你的 AppGlideModule 实现中复写 isManifestParsingEnabled() 方法：

```
    @Override
    public boolean isManifestParsingEnabled() {
        return false;
    }
```
IImageLoaderClient接口
```

/**
 * Created by shiming on 2016/10/26.
 */

public interface IImageLoaderClient {
    public void init(Context context);

    public void destroy(Context context);

    public File getCacheDir(Context context);

    public void clearMemoryCache(Context context);

    public void clearDiskCache(Context context);

    public Bitmap getBitmapFromCache(Context context, String url);

    public void getBitmapFromCache(Context context, String url, IGetBitmapListener listener);

    public void displayImage(Context context, int resId, ImageView imageView);

    public void displayImage(Context context, String url, ImageView imageView);

    public void displayImage(Context context, String url, ImageView imageView, boolean isCache);

    public void displayImage(Fragment fragment, String url, ImageView imageView);

    public void displayImage(Context context, String url, ImageView imageView, int defRes);

    public void displayImage(Fragment fragment, String url, ImageView imageView, int defRes);

    public void displayImage(Context context, String url, ImageView imageView, int defRes, BitmapTransformation transformations);

    public void displayImage(Fragment fragment, String url, ImageView imageView, int defRes, BitmapTransformation transformations);

    public void displayImage(Context context, String url, ImageView imageView, int defRes, ImageSize size);

    public void displayImage(Fragment fragment, String url, ImageView imageView, int defRes, ImageSize size);

    public void displayImage(Context context, String url, ImageView imageView, int defRes, boolean cacheInMemory);

    public void displayImage(Fragment fragment, String url, ImageView imageView, int defRes, boolean cacheInMemory);


    public void displayImage(Context context, String url, ImageView imageView, IImageLoaderListener listener);

    public void displayImage(Fragment fragment, String url, ImageView imageView, IImageLoaderListener listener);

    public void displayImage(Context context, String url, ImageView imageView, int defRes, IImageLoaderListener listener);

    public void displayImage(Fragment fragment, String url, ImageView imageView, int defRes, IImageLoaderListener listener);


    public void displayCircleImage(Context context, String url, ImageView imageView, int defRes);

    public void displayCircleImage(Fragment fragment, String url, ImageView imageView, int defRes);

    public void displayRoundImage(Context context, String url, ImageView imageView, int defRes, int radius);

    public void displayRoundImage(Fragment fragment, String url, ImageView imageView, int defRes, int radius);

    public void displayBlurImage(Context context, String url, int blurRadius, IGetDrawableListener listener);

    public void displayBlurImage(Context context, String url, ImageView imageView, int defRes, int blurRadius);

    public void displayBlurImage(Context context, int resId, ImageView imageView, int blurRadius);

    public void displayBlurImage(Fragment fragment, String url, ImageView imageView, int defRes, int blurRadius);

    public void displayImageInResource(Context context, int resId, ImageView imageView);

    public void displayImageInResource(Fragment fragment, int resId, ImageView imageView);

    public void displayImageInResource(Context context, int resId, ImageView imageView, BitmapTransformation transformations);

    public void displayImageInResource(Fragment fragment, int resId, ImageView imageView, BitmapTransformation transformations);

    public void displayImageInResource(Context context, int resId, ImageView imageView, int defRes);

    public void displayImageInResource(Fragment fragment, int resId, ImageView imageView, int defRes);

    public void displayImageInResource(Context context, int resId, ImageView imageView, int defRes, BitmapTransformation transformations);

    public void displayImageInResource(Fragment fragment, int resId, ImageView imageView, int defRes, BitmapTransformation transformations);


    //add shiming   2018.4.20 transformation 需要装换的那种图像的风格，错误图片，或者是，正在加载中的错误图
    public void displayImageInResourceTransform(Activity activity, int resId, ImageView imageView, Transformation transformation, int errorResId);
    public void displayImageInResourceTransform(Context context, int resId, ImageView imageView, Transformation transformation, int errorResId);
    public void displayImageInResourceTransform(Fragment fragment, int resId, ImageView imageView, Transformation transformation, int errorResId);

    //这是对网络图片，进行的图片操作，使用的glide中的方法
    public void displayImageByNet(Context context, String url, ImageView imageView, int defRes,Transformation transformation);
    public void displayImageByNet(Fragment fragment, String url, ImageView imageView, int defRes,Transformation transformation);
    public void displayImageByNet(Activity activity, String url, ImageView imageView, int defRes,Transformation transformation);


    /**
     *  停止图片的加载，对某一个的Activity
     * @hide
     */
    public void clear(Activity activity,ImageView imageView);
    /**
     * 停止图片的加载，context
     * {@hide}
     */
    public void clear(Context context,ImageView imageView);
    /**
     * 停止图片的加载，fragment
     * {@hide}
     */
    public void clear(Fragment fragment,ImageView imageView);


    //如果需要的话，需要指定加载中，或者是失败的图片
    public void displayImageByDiskCacheStrategy(Fragment fragment, String url, DiskCacheStrategy diskCacheStrategy,ImageView imageView);
    public void displayImageByDiskCacheStrategy(Activity activity, String url, DiskCacheStrategy diskCacheStrategy,ImageView imageView);
    public void displayImageByDiskCacheStrategy(Context context, String url, DiskCacheStrategy diskCacheStrategy,ImageView imageView);
    //某些情形下，你可能希望只要图片不在缓存中则加载直接失败（比如省流量模式）
    public void disPlayImageOnlyRetrieveFromCache(Fragment fragment,String url,ImageView imageView);
    public void disPlayImageOnlyRetrieveFromCache(Activity activity,String url,ImageView imageView);
    public void disPlayImageOnlyRetrieveFromCache(Context context,String url,ImageView imageView);



    /**
     *如果你想确保一个特定的请求跳过磁盘和/或内存缓存（比如，图片验证码 –）
     * @param fragment
     * @param url
     * @param imageView
     * @param skipflag  是否跳过内存缓存
     * @param diskCacheStratey  是否跳过磁盘缓存
     */
    public void disPlayImageSkipMemoryCache(Fragment fragment,String url,ImageView imageView,boolean skipflag,boolean diskCacheStratey);
    public void disPlayImageSkipMemoryCache(Activity activity,String url,ImageView imageView,boolean skipflag,boolean diskCacheStratey);
    public void disPlayImageSkipMemoryCache(Context context,String url,ImageView imageView,boolean skipflag,boolean diskCacheStratey);

    /**
     * 知道这个图片会加载失败，那么的话，我们可以重新加载
     * @param fragment
     * @param url
     * @param fallbackUrl
     * @param imageView
     */
    //从 Glide 4.3.0 开始，你可以很轻松地使用 .error() 方法。这个方法接受一个任意的 RequestBuilder,它会且只会在主请求失败时开始一个新的请求：
    public void disPlayImageErrorReload(Fragment fragment,String url,String fallbackUrl,ImageView imageView);
    public void disPlayImageErrorReload(Activity activity,String url,String fallbackUrl,ImageView imageView);
    public void disPlayImageErrorReload(Context context,String url,String fallbackUrl,ImageView imageView);


    /**
     未来 Glide 将默认加载硬件位图而不需要额外的启用配置，只保留禁用的选项 现在已经默认开启了这个配置，但是在有些情况下需要关闭
     所以提供了以下的方法，禁用硬件位图 disallowHardwareConfig
     * @param fragment
     * @param url
     * @param imageView
     */
//    哪些情况不能使用硬件位图?
//    在显存中存储像素数据意味着这些数据不容易访问到，在某些情况下可能会发生异常。已知的情形列举如下：
//    在 Java 中读写像素数据，包括：
//    Bitmap#getPixel
//    Bitmap#getPixels
//    Bitmap#copyPixelsToBuffer
//    Bitmap#copyPixelsFromBuffer
//    在本地 (native) 代码中读写像素数据
//    使用软件画布 (software Canvas) 渲染硬件位图:
//    Canvas canvas = new Canvas(normalBitmap)
//canvas.drawBitmap(hardwareBitmap, 0, 0, new Paint());
//    在绘制位图的 View 上使用软件层 (software layer type) （例如，绘制阴影）
//    ImageView imageView = …
//            imageView.setImageBitmap(hardwareBitmap);
//imageView.setLayerType(View.LAYER_TYPE_SOFTWARE, null);
//    打开过多的文件描述符 . 每个硬件位图会消耗一个文件描述符。
// 这里存在一个每个进程的文件描述符限制 ( Android O 及更早版本一般为 1024，在某些 O-MR1 和更高的构建上是 32K)。
// Glide 将尝试限制分配的硬件位图以保持在这个限制以内，但如果你已经分配了大量的文件描述符，这可能是一个问题。
//    需要ARGB_8888 Bitmaps 作为前置条件
//    在代码中触发截屏操作，它会尝试使用 Canvas 来绘制视图层级。
//    作为一个替代方案，在 Android O 以上版本你可以使用 PixelCopy.
//   共享元素过渡 (shared element transition)(OMR1已修复)
    public void disPlayImagedisallowHardwareConfig(Fragment fragment,String url,ImageView imageView);
    public void disPlayImagedisallowHardwareConfig(Activity activity,String url,ImageView imageView);
    public void disPlayImagedisallowHardwareConfig(Context context,String url,ImageView imageView);

    //监听图片的下载进度，是否完成，百分比 也可以加载本地图片，扩张一下
    public void disPlayImageProgress(Context context,String url,ImageView imageView,int placeholderResId,int errorResId,OnGlideImageViewListener listener);
    public void disPlayImageProgress(Activity activity,String url,ImageView imageView,int placeholderResId,int errorResId,OnGlideImageViewListener listener);
    public void disPlayImageProgress(Fragment fragment,String url,ImageView imageView,int placeholderResId,int errorResId,OnGlideImageViewListener listener);

    public void disPlayImageProgressByOnProgressListener(Context context,String url,ImageView imageView,int placeholderResId,int errorResId, OnProgressListener onProgressListener);
    public void disPlayImageProgressByOnProgressListener(Activity activity,String url,ImageView imageView,int placeholderResId,int errorResId, OnProgressListener onProgressListener);
    public void disPlayImageProgressByOnProgressListener(Fragment fragment,String url,ImageView imageView,int placeholderResId,int errorResId, OnProgressListener onProgressListener);



//    TransitionOptions 用于给一个特定的请求指定过渡。
//    每个请求可以使用 RequestBuilder 中的 transition()
//    方法来设定 TransitionOptions 。还可以通过使用
//    BitmapTransitionOptions 或 DrawableTransitionOptions
//    来指定类型特定的过渡动画。对于 Bitmap 和 Drawable
//    之外的资源类型，可以使用 GenericTransitionOptions。 Glide v4 将不会默认应用交叉淡入或任何其他的过渡效果。每个请求必须手动应用过渡。
    public void displayImageByTransition(Context context, String url, TransitionOptions transitionOptions, ImageView imageView);
    public void displayImageByTransition(Activity activity, String url, TransitionOptions transitionOptions, ImageView imageView);
    public void displayImageByTransition(Fragment fragment, String url, TransitionOptions transitionOptions, ImageView imageView);

    //失去焦点，建议实际的项目中少用，取消求情
    public void glidePauseRequests(Context context);
    public void glidePauseRequests(Activity activity);
    public void glidePauseRequests(Fragment fragment);

    //获取焦点，建议实际的项目中少用
    public void glideResumeRequests(Context context);
    public void glideResumeRequests(Activity activity);
    public void glideResumeRequests(Fragment fragment);
    //加载缩图图     int thumbnailSize = 10;//越小，图片越小，低网络的情况，图片越小
    //GlideApp.with(this).load(urlnoData).override(thumbnailSize))// API 来强制 Glide 在缩略图请求中加载一个低分辨率图像
    public void displayImageThumbnail(Context context,String url,String backUrl,int thumbnailSize,ImageView imageView);
    public void displayImageThumbnail(Activity activity,String url,String backUrl,int thumbnailSize,ImageView imageView);
    public void displayImageThumbnail(Fragment fragment,String url,String backUrl,int thumbnailSize,ImageView imageView);
    //如果没有两个url的话，也想，记载一个缩略图
    public void displayImageThumbnail(Fragment fragment,String url,float thumbnailSize,ImageView imageView);
    public void displayImageThumbnail(Activity activity,String url,float thumbnailSize,ImageView imageView);
    public void displayImageThumbnail(Context context,String url,float thumbnailSize,ImageView imageView);
}

```

GlideImageLoaderClient实现类

  ```
package code.shiming.com.imageloader471;

import android.annotation.SuppressLint;
import android.app.Activity;
import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.drawable.Drawable;
import android.os.AsyncTask;
import android.os.Handler;
import android.os.Looper;
import android.os.SystemClock;
import android.support.annotation.NonNull;
import android.support.annotation.Nullable;
import android.support.annotation.UiThread;
import android.support.v4.app.Fragment;
import android.util.Log;
import android.widget.ImageView;

import com.bumptech.glide.Glide;
import com.bumptech.glide.TransitionOptions;
import com.bumptech.glide.load.DataSource;
import com.bumptech.glide.load.Transformation;
import com.bumptech.glide.load.engine.DiskCacheStrategy;
import com.bumptech.glide.load.engine.GlideException;
import com.bumptech.glide.load.resource.bitmap.BitmapTransformation;
import com.bumptech.glide.load.resource.drawable.DrawableTransitionOptions;
import com.bumptech.glide.request.RequestListener;
import com.bumptech.glide.request.RequestOptions;
import com.bumptech.glide.request.target.SimpleTarget;
import com.bumptech.glide.request.target.Target;
import com.bumptech.glide.request.transition.Transition;

import java.io.File;

import code.shiming.com.imageloader471.listener.IGetBitmapListener;
import code.shiming.com.imageloader471.listener.IGetDrawableListener;
import code.shiming.com.imageloader471.listener.IImageLoaderListener;
import code.shiming.com.imageloader471.listener.ImageSize;
import code.shiming.com.imageloader471.okhttp.OnGlideImageViewListener;
import code.shiming.com.imageloader471.okhttp.OnProgressListener;
import code.shiming.com.imageloader471.okhttp.ProgressManager;
import code.shiming.com.imageloader471.tranform.BlurBitmapTranformation;
import code.shiming.com.imageloader471.tranform.GlideCircleTransformation;
import code.shiming.com.imageloader471.tranform.RoundBitmapTranformation;


/**
 * Created by shiming on 2016/10/26.
 * des:
 * with(Context context). 使用Application上下文，Glide请求将不受Activity/Fragment生命周期控制。
   with(Activity activity).使用Activity作为上下文，Glide的请求会受到Activity生命周期控制。
   with(FragmentActivity activity).Glide的请求会受到FragmentActivity生命周期控制。
   with(android.app.Fragment fragment).Glide的请求会受到Fragment 生命周期控制。
   with(android.support.v4.app.Fragment fragment).Glide的请求会受到Fragment生命周期控制。
 */

public class GlideImageLoaderClient implements IImageLoaderClient {

    private static final String TAG ="GlideImageLoaderClient";

    @Override
    public void init(Context context) {
    }
    @Override
    public void destroy(Context context) {
        clearMemoryCache(context);
    }
    @Override
    public File getCacheDir(Context context) {
        return Glide.getPhotoCacheDir(context);
    }

    /**
     * 使用ui线程
     * @param context
     */
    @UiThread
    @Override
    public void clearMemoryCache(Context context) {
        GlideApp.get(context).clearMemory();
    }

    @SuppressLint("StaticFieldLeak")
    @Override
    public void clearDiskCache(final Context context) {
        new AsyncTask<Void, Void, Void> (){
            @Override
            protected Void doInBackground(Void... params) {
                //必须在子线程中  This method must be called on a background thread.
                Glide.get(context).clearDiskCache();
                return null;
            }
        };
    }

    @Override
    public Bitmap getBitmapFromCache(Context context, String url) {
        throw new UnsupportedOperationException("glide 不支持同步 获取缓存中 bitmap");
    }
    /**
     * 获取缓存中的图片
     * @param context
     * @param url
     * @param listener
     */
    @Override
    public void getBitmapFromCache(Context context, String url, final IGetBitmapListener listener) {
       GlideApp.with(context).asBitmap().load(url).into(new SimpleTarget<Bitmap>() {
           @Override
           public void onResourceReady(@NonNull Bitmap resource, @Nullable Transition<? super Bitmap> transition) {
               if (listener != null) {
                   listener.onBitmap(resource);
               }
           }
       });
    }

    /**
     *
     默认的策略是DiskCacheStrategy.AUTOMATIC
     DiskCacheStrategy.ALL 使用DATA和RESOURCE缓存远程数据，仅使用RESOURCE来缓存本地数据。
     DiskCacheStrategy.NONE 不使用磁盘缓存
     DiskCacheStrategy.DATA 在资源解码前就将原始数据写入磁盘缓存
     DiskCacheStrategy.RESOURCE 在资源解码后将数据写入磁盘缓存，即经过缩放等转换后的图片资源。
     DiskCacheStrategy.AUTOMATIC 根据原始图片数据和资源编码策略来自动选择磁盘缓存策略。
     * @param context  上下文
     * @param resId  id
     * @param imageView into
      */
    //DiskCacheStrategy.SOURCE：缓存原始数据 DiskCacheStrategy.DATA对应Glide 3中的DiskCacheStrategy.SOURCE
    @Override
    public void displayImage(Context context, int resId, ImageView imageView) {
        //设置缓存策略缓存原始数据  Saves just the original data to cache
        GlideApp.with(context).load(resId).diskCacheStrategy(DiskCacheStrategy.AUTOMATIC).into(imageView);
    }

    /**
     *
     * @param context
     * @param url url
     * @param imageView in
     */
    @Override
    public void displayImage(Context context, String url, ImageView imageView) {
        GlideApp.with(context).load(url).diskCacheStrategy(DiskCacheStrategy.AUTOMATIC).into(imageView);
    }

    /**
     *
     * @param context
     * @param url
     * @param imageView
     * @param isCache 是否是缓存 如果是：缓存策略缓存原始数据  不是的话 ：缓存策略DiskCacheStrategy.NONE：什么都不缓存
     */
    @Override
    public void displayImage(Context context, String url, ImageView imageView, boolean isCache) {
        GlideApp.with(context).load(url).skipMemoryCache(isCache).diskCacheStrategy(isCache ? DiskCacheStrategy.AUTOMATIC : DiskCacheStrategy.NONE).into(imageView);
    }

    /**
     *
     * @param fragment 绑定生命周期
     * @param url
     * @param imageView
     */
    @Override
    public void displayImage(Fragment fragment, String url, ImageView imageView) {
        GlideApp.with(fragment).load(url).diskCacheStrategy(DiskCacheStrategy.AUTOMATIC).into(imageView);
    }

    /**
     * 使用.placeholder()方法在某些情况下会导致图片显示的时候出现图片变形的情况
     * 这是因为Glide默认开启的crossFade动画导致的TransitionDrawable绘制异常
     * @param context
     * @param url
     * @param imageView
     * @param defRes defRes 可以是个new ColorDrawable(Color.BLACK) 也可以是张图片
     */
    //默认为200  时间有点长  ，工程中要修改下，设置一个加载失败和加载中的动画过渡，V4.0的使用的方法
    @Override
    public void displayImage(Context context, String url, ImageView imageView, int defRes) {
        GlideApp.with(context).load(url).diskCacheStrategy(DiskCacheStrategy.AUTOMATIC).transition(new DrawableTransitionOptions().crossFade(200)).placeholder(defRes).error(defRes).into(imageView);
    }

    /**
     * 默认时间为200 需
     * @param fragment
     * @param url
     * @param imageView
     * @param defRes
     */
    @Override
    public void displayImage(Fragment fragment, String url, ImageView imageView, int defRes) {
        GlideApp.with(fragment).load(url).diskCacheStrategy(DiskCacheStrategy.AUTOMATIC).transition(new DrawableTransitionOptions().crossFade(200)).placeholder(defRes).error(defRes).into(imageView);
    }

    /**
     *
     * @param context
     * @param url
     * @param imageView
     * @param defRes
     * @param transformations bitmapTransform 方法设置图片转换
     */

    @Override
    public void displayImage(Context context, String url, ImageView imageView, int defRes, final BitmapTransformation transformations) {
        GlideApp.with(context).load(url).diskCacheStrategy(DiskCacheStrategy.AUTOMATIC).apply(requestOptionsTransformation(defRes,defRes,transformations)).into(imageView);
    }

    @Override
    public void displayImage(Fragment fragment, String url, ImageView imageView, int defRes, BitmapTransformation transformations) {
        GlideApp.with(fragment).load(url).diskCacheStrategy(DiskCacheStrategy.AUTOMATIC).apply(requestOptionsTransformation(defRes,defRes,transformations)).into(imageView);
    }


    /**
     * 加载原圆形图片
     * @param context
     * @param url
     * @param imageView
     * @param defRes
     */
    public void displayImageCircle(Context context, String url, ImageView imageView, int defRes) {
        GlideApp.with(context).load(url).diskCacheStrategy(DiskCacheStrategy.AUTOMATIC).apply(circleRequestOptions(defRes,defRes)).into(imageView);

    }
    public void displayImageCircle(Fragment context, String url, ImageView imageView, int defRes) {
        GlideApp.with(context).load(url).diskCacheStrategy(DiskCacheStrategy.DATA).apply(circleRequestOptions(defRes,defRes)).into(imageView);
    }
    public RequestOptions requestOptions(int placeholderResId, int errorResId) {
        return new RequestOptions()
                .placeholder(placeholderResId)
                .error(errorResId);
    }
    public RequestOptions requestOptionsTransformation(int placeholderResId, int errorResId,BitmapTransformation bitmapTransformation) {
        return requestOptions(placeholderResId,errorResId)
                .transform(bitmapTransformation);
    }
    /**
     * 加载原图
     * @param placeholderResId
     * @param errorResId
     * @return
     */
    public RequestOptions circleRequestOptions(int placeholderResId, int errorResId) {
        return requestOptions(placeholderResId, errorResId)
                .transform(new GlideCircleTransformation());
    }

    public RequestOptions roundRequestOptions(int placeholderResId, int errorResId,int radius) {
        return requestOptions(placeholderResId, errorResId)
                .transform(new RoundBitmapTranformation(radius));
    }
    /**
     *
     * @param context
     * @param url
     * @param imageView
     * @param defRes placeholder(int resourceId). 设置资源加载过程中的占位Drawable  error(int resourceId).设置load失败时显示的Drawable
     * @param size override(int width, int height). 重新设置Target的宽高值
     */
    @Override
    public void displayImage(Context context, String url, ImageView imageView, int defRes, ImageSize size) {
        GlideApp.with(context).load(url).diskCacheStrategy(DiskCacheStrategy.AUTOMATIC).placeholder(defRes).error(defRes).override(size.getWidth(), size.getHeight()).into(imageView);
    }

    @Override
    public void displayImage(Fragment fragment, String url, ImageView imageView, int defRes, ImageSize size) {
        GlideApp.with(fragment).load(url).diskCacheStrategy(DiskCacheStrategy.AUTOMATIC).placeholder(defRes).error(defRes).override(size.getWidth(), size.getHeight()).into(imageView);
    }

    /**
     * .skipMemoryCache( true )去特意告诉Glide跳过内存缓存  是否跳过内存，还是不跳过
     * @param context
     * @param url
     * @param imageView
     * @param defRes
     * @param cacheInMemory
     */
    @Override
    public void displayImage(Context context, String url, ImageView imageView, int defRes, boolean cacheInMemory) {
        GlideApp.with(context).load(url).diskCacheStrategy(DiskCacheStrategy.AUTOMATIC).placeholder(defRes).error(defRes).skipMemoryCache(cacheInMemory).into(imageView);
    }

    @Override
    public void displayImage(Fragment fragment, String url, ImageView imageView, int defRes, boolean cacheInMemory) {
        GlideApp.with(fragment).load(url).diskCacheStrategy(DiskCacheStrategy.AUTOMATIC).placeholder(defRes).error(defRes).skipMemoryCache(cacheInMemory).into(imageView);
    }

    /**
     * 只在需要的地方进行监听 listener 通过自定义的接口回调参数
     * @param context
     * @param url
     * @param imageView
     * @param listener  监听资源加载的请求状态 但不要每次请求都使用新的监听器，要避免不必要的内存申请，
     *                  可以使用单例进行统一的异常监听和处理
     */
    @Override
    public void displayImage(Context context, final String url, final ImageView imageView, final IImageLoaderListener listener) {
        GlideApp.with(context).load(url).diskCacheStrategy(DiskCacheStrategy.AUTOMATIC).listener(new RequestListener<Drawable>() {
            @Override
            public boolean onLoadFailed(@Nullable GlideException e, Object model, Target<Drawable> target, boolean isFirstResource) {
                listener.onLoadingFailed(url, imageView, e);
                Log.e(TAG, "Load failed", e);//如果关系的话，关系如何失败
                return false;
            }

            @Override
            public boolean onResourceReady(Drawable resource, Object model, Target<Drawable> target, DataSource dataSource, boolean isFirstResource) {
                listener.onLoadingComplete(url, imageView);
                return false;
            }
        }).into(imageView);
    }

    @Override
    public void displayImage(Fragment fragment, final String url, final ImageView imageView, final IImageLoaderListener listener) {
        GlideApp.with(fragment).load(url).diskCacheStrategy(DiskCacheStrategy.AUTOMATIC).listener(new RequestListener<Drawable>() {
            @Override
            public boolean onLoadFailed(@Nullable GlideException e, Object model, Target<Drawable> target, boolean isFirstResource) {
                listener.onLoadingFailed(url, imageView, e);
                return false;
            }

            @Override
            public boolean onResourceReady(Drawable resource, Object model, Target<Drawable> target, DataSource dataSource, boolean isFirstResource) {
                listener.onLoadingComplete(url, imageView);
                return false;
            }
        }).into(imageView);
    }

    @Override
    public void displayImage(Context context, final String url, final ImageView imageView, int defRes, final IImageLoaderListener listener) {
        GlideApp.with(context).load(url).diskCacheStrategy(DiskCacheStrategy.AUTOMATIC).placeholder(defRes).error(defRes).listener(new RequestListener<Drawable>() {
            @Override
            public boolean onLoadFailed(@Nullable GlideException e, Object model, Target<Drawable> target, boolean isFirstResource) {
                listener.onLoadingFailed(url, imageView, e);
                return false;
            }

            @Override
            public boolean onResourceReady(Drawable resource, Object model, Target<Drawable> target, DataSource dataSource, boolean isFirstResource) {
                listener.onLoadingComplete(url, imageView);
                return false;
            }
        }).into(imageView);
    }

    @Override
    public void displayImage(Fragment fragment, final String url, final ImageView imageView, int defRes, final IImageLoaderListener listener) {
        GlideApp.with(fragment).load(url).diskCacheStrategy(DiskCacheStrategy.AUTOMATIC).listener(new RequestListener<Drawable>() {
            @Override
            public boolean onLoadFailed(@Nullable GlideException e, Object model, Target<Drawable> target, boolean isFirstResource) {
                listener.onLoadingFailed(url, imageView, e);
                return false;
            }

            @Override
            public boolean onResourceReady(Drawable resource, Object model, Target<Drawable> target, DataSource dataSource, boolean isFirstResource) {
                listener.onLoadingComplete(url, imageView);
                return false;
            }
        }).into(imageView);
    }

    /**
     * 圆形图片的裁剪
     * @param context
     * @param url
     * @param imageView
     * @param defRes
     */
    @Override
    public void displayCircleImage(Context context, String url, ImageView imageView, int defRes) {
        GlideApp.with(context).load(url).diskCacheStrategy(DiskCacheStrategy.AUTOMATIC).apply(circleRequestOptions(defRes,defRes)).into(imageView);
    }

    @Override
    public void displayCircleImage(Fragment fragment, String url, ImageView imageView, int defRes) {
        GlideApp.with(fragment).load(url).diskCacheStrategy(DiskCacheStrategy.AUTOMATIC).apply(circleRequestOptions(defRes,defRes)).into(imageView);
    }


    public void displayCircleImage(Activity fragment, String url, ImageView imageView, int defRes) {
        GlideApp.with(fragment).load(url).diskCacheStrategy(DiskCacheStrategy.DATA).apply(circleRequestOptions(defRes,defRes)).into(imageView);
    }
    /**
     *
     * @param context
     * @param url
     * @param imageView
     * @param defRes
     * @param radius 倒圆角的图片 需要传入需要radius  越大的话，倒角越明显
     */
    @Override
    public void displayRoundImage(Context context, String url, ImageView imageView, int defRes, int radius) {
        GlideApp.with(context).load(url).diskCacheStrategy(DiskCacheStrategy.AUTOMATIC).apply(roundRequestOptions(defRes,defRes,radius)).into(imageView);
    }

    @Override
    public void displayRoundImage(Fragment fragment, String url, ImageView imageView, int defRes, int radius) {
        GlideApp.with(fragment).load(url).diskCacheStrategy(DiskCacheStrategy.DATA).apply(roundRequestOptions(defRes,defRes,radius)).into(imageView);
    }
    /**
     *
     * @param context
     * @param url
     * @param blurRadius 模糊的程度 ，数字越大越模糊
     * @param listener 接口回调需要拿到drawable
     */
    @Override
    public void displayBlurImage(Context context, String url, int blurRadius, final IGetDrawableListener listener) {
        GlideApp.with(context).load(url).diskCacheStrategy(DiskCacheStrategy.AUTOMATIC).into(new SimpleTarget<Drawable>() {
            @Override
            public void onResourceReady(@NonNull Drawable resource, @Nullable Transition<? super Drawable> transition) {
                if ( listener != null ) {
                    listener.onDrawable( resource );
                }
            }
        });
    }

    /**
     * 不需要关系此模糊图的drawable
     * @param context
     * @param url
     * @param imageView
     * @param defRes
     * @param blurRadius
     */
    @Override
    public void displayBlurImage(Context context, String url, ImageView imageView, int defRes, int blurRadius) {
        GlideApp.with(context).load(url).diskCacheStrategy(DiskCacheStrategy.AUTOMATIC).apply(blurRequestOptions(defRes,defRes,blurRadius)).into(imageView);
    }

    private RequestOptions blurRequestOptions(int defRes, int defRes1, int blurRadius) {
        return requestOptions(defRes, defRes1)
                .transform(new BlurBitmapTranformation(blurRadius));
    }

    @Override
    public void displayBlurImage(Context context, int resId, ImageView imageView, int blurRadius) {
        GlideApp.with(context).load(resId).diskCacheStrategy(DiskCacheStrategy.AUTOMATIC).apply(blurRequestOptions(resId,resId,blurRadius)).into(imageView);
    }

    @Override
    public void displayBlurImage(Fragment fragment, String url, ImageView imageView, int defRes, int blurRadius) {
        GlideApp.with(fragment).load(url).diskCacheStrategy(DiskCacheStrategy.AUTOMATIC).apply(blurRequestOptions(defRes,defRes,blurRadius)).into(imageView);
    }

    /**
     *  加载资源文件
     * @param context
     * @param resId
     * @param imageView
     */
    @Override
    public void displayImageInResource(Context context, int resId, ImageView imageView) {
        GlideApp.with(context).load(resId).diskCacheStrategy(DiskCacheStrategy.NONE).into(imageView);
    }

    @Override
    public void displayImageInResource(Fragment fragment, int resId, ImageView imageView) {
        GlideApp.with(fragment).load(resId).diskCacheStrategy(DiskCacheStrategy.NONE).into(imageView);
    }

    public void displayImageInResource(Activity fragment, int resId, ImageView imageView) {
        GlideApp.with(fragment).load(resId).diskCacheStrategy(DiskCacheStrategy.NONE).into(imageView);
    }

    /**
     *
     * @param fragment
     * @param resId
     * @param imageView
     * @param transformation 需要变换那种图像
     * @param errorResId
     */
    @Override
    public void displayImageInResourceTransform(Activity fragment, int resId, ImageView imageView,Transformation transformation,int errorResId) {
        GlideApp.with(fragment).load(resId).diskCacheStrategy(DiskCacheStrategy.NONE).apply(requestOptionsTransform(errorResId,errorResId,transformation)).into(imageView);
    }

    @Override
    public void displayImageInResourceTransform(Context context, int resId, ImageView imageView, Transformation transformation, int errorResId) {
        GlideApp.with(context).load(resId).diskCacheStrategy(DiskCacheStrategy.NONE).apply(requestOptionsTransform(errorResId,errorResId,transformation)).into(imageView);
    }

    @Override
    public void displayImageInResourceTransform(Fragment fragment, int resId, ImageView imageView, Transformation transformation, int errorResId) {
        GlideApp.with(fragment).load(resId).diskCacheStrategy(DiskCacheStrategy.NONE).apply(requestOptionsTransform(errorResId,errorResId,transformation)).into(imageView);
    }

    @Override
    public void displayImageByNet(Context context, String url, ImageView imageView, int defRes, Transformation transformation) {
        GlideApp.with(context).load(url).diskCacheStrategy(DiskCacheStrategy.AUTOMATIC).apply(requestOptionsTransform(defRes,defRes,transformation)).into(imageView);
    }

    @Override
    public void displayImageByNet(Fragment fragment, String url, ImageView imageView, int defRes, Transformation transformation) {
        GlideApp.with(fragment).load(url).diskCacheStrategy(DiskCacheStrategy.AUTOMATIC).apply(requestOptionsTransform(defRes,defRes,transformation)).into(imageView);
    }

    @Override
    public void displayImageByNet(Activity activity, String url, ImageView imageView, int defRes, Transformation transformation) {
        GlideApp.with(activity).load(url).diskCacheStrategy(DiskCacheStrategy.AUTOMATIC).apply(requestOptionsTransform(defRes,defRes,transformation)).into(imageView);
    }

    @Override
    public void clear(Activity activity, ImageView imageView) {
        GlideApp.with(activity).clear(imageView);
    }

    @Override
    public void clear(Context context, ImageView imageView) {
        GlideApp.with(context).clear(imageView);
    }

    @Override
    public void clear(Fragment fragment, ImageView imageView) {
        GlideApp.with(fragment).clear(imageView);
    }
//    默认的策略是DiskCacheStrategy.AUTOMATIC
//    DiskCacheStrategy.ALL 使用DATA和RESOURCE缓存远程数据，仅使用RESOURCE来缓存本地数据。
//    DiskCacheStrategy.NONE 不使用磁盘缓存
//    DiskCacheStrategy.DATA 在资源解码前就将原始数据写入磁盘缓存
//    DiskCacheStrategy.RESOURCE 在资源解码后将数据写入磁盘缓存，即经过缩放等转换后的图片资源。
//    DiskCacheStrategy.AUTOMATIC 根据原始图片数据和资源编码策略来自动选择磁盘缓存策略。
    @Override
    public void displayImageByDiskCacheStrategy(Fragment fragment, String url, DiskCacheStrategy diskCacheStrategy,ImageView imageView) {
        GlideApp.with(fragment).load(url).diskCacheStrategy(diskCacheStrategy).into(imageView);
    }
//    DiskCacheStrategy.NONE： 表示不缓存任何内容。
//    DiskCacheStrategy.DATA： 表示只缓存原始图片。
//    DiskCacheStrategy.RESOURCE： 表示只缓存转换过后的图片。
//    DiskCacheStrategy.ALL ： 表示既缓存原始图片，也缓存转换过后的图片。
//    DiskCacheStrategy.AUTOMATIC： 表示让Glide根据图片资源智能地选择使用哪一种缓存策略（默认选项）。
    @Override
    public void displayImageByDiskCacheStrategy(Activity activity, String url, DiskCacheStrategy diskCacheStrategy,ImageView imageView) {
        GlideApp.with(activity).load(url).diskCacheStrategy(diskCacheStrategy).into(imageView);
    }

    @Override
    public void displayImageByDiskCacheStrategy(Context context, String url, DiskCacheStrategy diskCacheStrategy,ImageView imageView) {
        GlideApp.with(context).load(url).diskCacheStrategy(diskCacheStrategy).into(imageView);
    }

    @Override
    public void disPlayImageOnlyRetrieveFromCache(Fragment fragment, String url, ImageView imageView) {
        GlideApp.with(fragment)
                .load(url)
                .onlyRetrieveFromCache(true)
                .into(imageView);
    }

    @Override
    public void disPlayImageOnlyRetrieveFromCache(Activity activity, String url, ImageView imageView) {
        GlideApp.with(activity)
                .load(url)
                .onlyRetrieveFromCache(true)
                .into(imageView);
    }

    @Override
    public void disPlayImageOnlyRetrieveFromCache(Context context, String url, ImageView imageView) {
        GlideApp.with(context)
                .load(url)
                .onlyRetrieveFromCache(true)
                .into(imageView);
    }
    /**
     *如果你想确保一个特定的请求跳过磁盘和/或内存缓存（比如，图片验证码 –）
     * @param fragment
     * @param url
     * @param imageView
     * @param skipflag  是否跳过内存缓存
     * @param diskCacheStratey  是否跳过磁盘缓存
     */
    @Override
    public void disPlayImageSkipMemoryCache(Fragment fragment, String url, ImageView imageView, boolean skipflag, boolean diskCacheStratey) {
        GlideApp.with(fragment)
                .load(url)
                .diskCacheStrategy(diskCacheStratey?DiskCacheStrategy.NONE:DiskCacheStrategy.AUTOMATIC)
                .skipMemoryCache(skipflag)
                .into(imageView);
    }

    @Override
    public void disPlayImageSkipMemoryCache(Activity activity, String url, ImageView imageView, boolean skipflag, boolean diskCacheStratey) {
        GlideApp.with(activity)
                .load(url)
                .diskCacheStrategy(diskCacheStratey?DiskCacheStrategy.NONE:DiskCacheStrategy.AUTOMATIC)
                .skipMemoryCache(skipflag)
                .into(imageView);
    }

    @Override
    public void disPlayImageSkipMemoryCache(Context context, String url, ImageView imageView, boolean skipflag, boolean diskCacheStratey) {
        GlideApp.with(context)
                .load(url)
                .diskCacheStrategy(diskCacheStratey?DiskCacheStrategy.NONE:DiskCacheStrategy.AUTOMATIC)
                .skipMemoryCache(skipflag)
                .into(imageView);
    }

    @Override
    public void disPlayImageErrorReload(Fragment fragment, String url, String fallbackUrl, ImageView imageView) {
        GlideApp.with(fragment)
                .load(url)
                .error(GlideApp.with(fragment)
                        .load(fallbackUrl))
                .into(imageView);
    }

    @Override
    public void disPlayImageErrorReload(Activity activity, String url, String fallbackUrl, ImageView imageView) {
        GlideApp.with(activity)
                .load(url)
                .error(GlideApp.with(activity)
                        .load(fallbackUrl))
                .into(imageView);
    }

    @Override
    public void disPlayImageErrorReload(Context context, String url, String fallbackUrl, ImageView imageView) {
        GlideApp.with(context)
                .load(url)
                .error(GlideApp.with(context)
                        .load(fallbackUrl))
                .into(imageView);
    }

    @Override
    public void disPlayImagedisallowHardwareConfig(Fragment fragment, String url, ImageView imageView) {
        GlideApp.with(fragment)
                .load(url)
                .disallowHardwareConfig()
                .into(imageView);
        //第二种方法
//        RequestOptions options = new RequestOptions().disallowHardwareConfig();
//        Glide.with(fragment)
//                .load(url)
//                .apply(options)
//                .into(imageView);
    }

    @Override
    public void disPlayImagedisallowHardwareConfig(Activity activity, String url, ImageView imageView) {
        GlideApp.with(activity)
                .load(url)
                .disallowHardwareConfig()
                .into(imageView);
    }

    @Override
    public void disPlayImagedisallowHardwareConfig(Context context, String url, ImageView imageView) {
        GlideApp.with(context)
                .load(url)
                .disallowHardwareConfig()
                .into(imageView);
    }
    //监听进度
    @Override
    public void disPlayImageProgress(Context context, final String url, ImageView imageView, int placeholderResId, int errorResId, OnGlideImageViewListener listener) {
        GlideApp.with(context)
                .load(url)
                .diskCacheStrategy(DiskCacheStrategy.NONE)//todo 我是为了测试，看到进度条，才把缓存策略设置成这样的，项目中一定不要这样做
                .apply(new RequestOptions()
                        .placeholder(placeholderResId)
                        .error(errorResId))
                .listener(new RequestListener<Drawable>() {
                    @Override
                    public boolean onLoadFailed(@Nullable GlideException e, Object model, Target<Drawable> target, boolean isFirstResource) {
                        mainThreadCallback(url,mLastBytesRead, mTotalBytes, true, e);
                        ProgressManager.removeProgressListener(internalProgressListener);
                        return false;
                    }
                    @Override
                    public boolean onResourceReady(Drawable resource, Object model, Target<Drawable> target, DataSource dataSource, boolean isFirstResource) {
                        mainThreadCallback(url,mLastBytesRead, mTotalBytes, true, null);
                        ProgressManager.removeProgressListener(internalProgressListener);
                        return false;
                    }
                }).into(imageView);

        //赋值 上去
        onGlideImageViewListener=listener;
        mMainThreadHandler = new Handler(Looper.getMainLooper());
        internalProgressListener = new OnProgressListener() {
            @Override
            public void onProgress(String imageUrl, final long bytesRead, final long totalBytes, final boolean isDone, final GlideException exception) {
                if (totalBytes == 0) return;
                if (mLastBytesRead == bytesRead && mLastStatus == isDone) return;

                mLastBytesRead = bytesRead;
                mTotalBytes = totalBytes;
                mLastStatus = isDone;
                mainThreadCallback(imageUrl,bytesRead, totalBytes, isDone, exception);

                if (isDone) {
                    ProgressManager.removeProgressListener(this);
                }
            }
        };
        ProgressManager.addProgressListener(internalProgressListener);



    }

    @Override
    public void disPlayImageProgress(Activity activity, final String url, ImageView imageView, int placeholderResId, int errorResId, OnGlideImageViewListener listener) {
        GlideApp.with(activity)
                .load(url)
                .diskCacheStrategy(DiskCacheStrategy.NONE)//todo 我是为了测试，看到进度条，才把缓存策略设置成这样的，项目中一定不要这样做
                .apply(new RequestOptions()
                        .placeholder(placeholderResId)
                        .error(errorResId))
                .listener(new RequestListener<Drawable>() {
                    @Override
                    public boolean onLoadFailed(@Nullable GlideException e, Object model, Target<Drawable> target, boolean isFirstResource) {
                        mainThreadCallback(url,mLastBytesRead, mTotalBytes, true, e);
                        ProgressManager.removeProgressListener(internalProgressListener);
                        return false;
                    }
                    @Override
                    public boolean onResourceReady(Drawable resource, Object model, Target<Drawable> target, DataSource dataSource, boolean isFirstResource) {
                        mainThreadCallback(url,mLastBytesRead, mTotalBytes, true, null);
                        ProgressManager.removeProgressListener(internalProgressListener);
                        return false;
                    }
                }).into(imageView);

        //赋值 上去
        onGlideImageViewListener=listener;
        mMainThreadHandler = new Handler(Looper.getMainLooper());
        internalProgressListener = new OnProgressListener() {
            @Override
            public void onProgress(String imageUrl, final long bytesRead, final long totalBytes, final boolean isDone, final GlideException exception) {
                if (totalBytes == 0) return;
                if (mLastBytesRead == bytesRead && mLastStatus == isDone) return;

                mLastBytesRead = bytesRead;
                mTotalBytes = totalBytes;
                mLastStatus = isDone;
                mainThreadCallback(imageUrl,bytesRead, totalBytes, isDone, exception);

                if (isDone) {
                    ProgressManager.removeProgressListener(this);
                }
            }
        };
        ProgressManager.addProgressListener(internalProgressListener);
    }

    @Override
    public void disPlayImageProgress(Fragment fragment, final String url, ImageView imageView, int placeholderResId, int errorResId, OnGlideImageViewListener listener) {
        GlideApp.with(fragment)
                .load(url)
                .diskCacheStrategy(DiskCacheStrategy.NONE)//todo 我是为了测试，看到进度条，才把缓存策略设置成这样的，项目中一定不要这样做
                .apply(new RequestOptions()
                        .placeholder(placeholderResId)
                        .error(errorResId))
                .listener(new RequestListener<Drawable>() {
                    @Override
                    public boolean onLoadFailed(@Nullable GlideException e, Object model, Target<Drawable> target, boolean isFirstResource) {
                        mainThreadCallback(url,mLastBytesRead, mTotalBytes, true, e);
                        ProgressManager.removeProgressListener(internalProgressListener);
                        return false;
                    }
                    @Override
                    public boolean onResourceReady(Drawable resource, Object model, Target<Drawable> target, DataSource dataSource, boolean isFirstResource) {
                        mainThreadCallback(url,mLastBytesRead, mTotalBytes, true, null);
                        ProgressManager.removeProgressListener(internalProgressListener);
                        return false;
                    }
                }).into(imageView);

        //赋值 上去
        onGlideImageViewListener=listener;
        mMainThreadHandler = new Handler(Looper.getMainLooper());
        internalProgressListener = new OnProgressListener() {
            @Override
            public void onProgress(String imageUrl, final long bytesRead, final long totalBytes, final boolean isDone, final GlideException exception) {
                if (totalBytes == 0) return;
                if (mLastBytesRead == bytesRead && mLastStatus == isDone) return;

                mLastBytesRead = bytesRead;
                mTotalBytes = totalBytes;
                mLastStatus = isDone;
                mainThreadCallback(imageUrl,bytesRead, totalBytes, isDone, exception);

                if (isDone) {
                    ProgressManager.removeProgressListener(this);
                }
            }
        };
        ProgressManager.addProgressListener(internalProgressListener);
    }



    @Override
    public void disPlayImageProgressByOnProgressListener(Context context, final String url, ImageView imageView, int placeholderResId, int errorResId, OnProgressListener onProgressListener) {
        GlideApp.with(context)
                .load(url)
                .diskCacheStrategy(DiskCacheStrategy.NONE)//todo 我是为了测试，看到进度条，才把缓存策略设置成这样的，项目中一定不要这样做
                .apply(new RequestOptions()
                        .placeholder(placeholderResId)
                        .error(errorResId))
                .listener(new RequestListener<Drawable>() {
                    @Override
                    public boolean onLoadFailed(@Nullable GlideException e, Object model, Target<Drawable> target, boolean isFirstResource) {
                        mainThreadCallback(url,mLastBytesRead, mTotalBytes, true, e);
                        ProgressManager.removeProgressListener(internalProgressListener);
                        return false;
                    }
                    @Override
                    public boolean onResourceReady(Drawable resource, Object model, Target<Drawable> target, DataSource dataSource, boolean isFirstResource) {
                        mainThreadCallback(url,mLastBytesRead, mTotalBytes, true, null);
                        ProgressManager.removeProgressListener(internalProgressListener);
                        return false;
                    }
                }).into(imageView);

        //赋值 上去
        this.onProgressListener = onProgressListener;
        mMainThreadHandler = new Handler(Looper.getMainLooper());
        internalProgressListener = new OnProgressListener() {
            @Override
            public void onProgress(String imageUrl, final long bytesRead, final long totalBytes, final boolean isDone, final GlideException exception) {
                if (totalBytes == 0) return;
                if (mLastBytesRead == bytesRead && mLastStatus == isDone) return;

                mLastBytesRead = bytesRead;
                mTotalBytes = totalBytes;
                mLastStatus = isDone;
                mainThreadCallback(imageUrl,bytesRead, totalBytes, isDone, exception);

                if (isDone) {
                    ProgressManager.removeProgressListener(this);
                }
            }
        };
        ProgressManager.addProgressListener(internalProgressListener);
    }

    @Override
    public void disPlayImageProgressByOnProgressListener(Activity activity, final String url, ImageView imageView, int placeholderResId, int errorResId, OnProgressListener onProgressListener) {
        GlideApp.with(activity)
                .load(url)
                .diskCacheStrategy(DiskCacheStrategy.NONE)//todo 我是为了测试，看到进度条，才把缓存策略设置成这样的，项目中一定不要这样做
                .apply(new RequestOptions()
                        .placeholder(placeholderResId)
                        .error(errorResId))
                .listener(new RequestListener<Drawable>() {
                    @Override
                    public boolean onLoadFailed(@Nullable GlideException e, Object model, Target<Drawable> target, boolean isFirstResource) {
                        mainThreadCallback(url,mLastBytesRead, mTotalBytes, true, e);
                        ProgressManager.removeProgressListener(internalProgressListener);
                        return false;
                    }
                    @Override
                    public boolean onResourceReady(Drawable resource, Object model, Target<Drawable> target, DataSource dataSource, boolean isFirstResource) {
                        mainThreadCallback(url,mLastBytesRead, mTotalBytes, true, null);
                        ProgressManager.removeProgressListener(internalProgressListener);
                        return false;
                    }
                }).into(imageView);

        //赋值 上去
        this.onProgressListener = onProgressListener;
        mMainThreadHandler = new Handler(Looper.getMainLooper());
        internalProgressListener = new OnProgressListener() {
            @Override
            public void onProgress(String imageUrl, final long bytesRead, final long totalBytes, final boolean isDone, final GlideException exception) {
                if (totalBytes == 0) return;
                if (mLastBytesRead == bytesRead && mLastStatus == isDone) return;

                mLastBytesRead = bytesRead;
                mTotalBytes = totalBytes;
                mLastStatus = isDone;
                mainThreadCallback(imageUrl,bytesRead, totalBytes, isDone, exception);

                if (isDone) {
                    ProgressManager.removeProgressListener(this);
                }
            }
        };
        ProgressManager.addProgressListener(internalProgressListener);
    }

    @Override
    public void disPlayImageProgressByOnProgressListener(Fragment fragment, final String url, ImageView imageView, int placeholderResId, int errorResId, OnProgressListener onProgressListener) {
        GlideApp.with(fragment)
                .load(url)
                .diskCacheStrategy(DiskCacheStrategy.NONE)//todo 我是为了测试，看到进度条，才把缓存策略设置成这样的，项目中一定不要这样做
                .apply(new RequestOptions()
                        .placeholder(placeholderResId)
                        .error(errorResId))
                .listener(new RequestListener<Drawable>() {
                    @Override
                    public boolean onLoadFailed(@Nullable GlideException e, Object model, Target<Drawable> target, boolean isFirstResource) {
                        mainThreadCallback(url,mLastBytesRead, mTotalBytes, true, e);
                        ProgressManager.removeProgressListener(internalProgressListener);
                        return false;
                    }
                    @Override
                    public boolean onResourceReady(Drawable resource, Object model, Target<Drawable> target, DataSource dataSource, boolean isFirstResource) {
                        mainThreadCallback(url,mLastBytesRead, mTotalBytes, true, null);
                        ProgressManager.removeProgressListener(internalProgressListener);
                        return false;
                    }
                }).into(imageView);

        //赋值 上去
        this.onProgressListener = onProgressListener;
        mMainThreadHandler = new Handler(Looper.getMainLooper());
        internalProgressListener = new OnProgressListener() {
            @Override
            public void onProgress(String imageUrl, final long bytesRead, final long totalBytes, final boolean isDone, final GlideException exception) {
                if (totalBytes == 0) return;
                if (mLastBytesRead == bytesRead && mLastStatus == isDone) return;

                mLastBytesRead = bytesRead;
                mTotalBytes = totalBytes;
                mLastStatus = isDone;
                mainThreadCallback(imageUrl,bytesRead, totalBytes, isDone, exception);

                if (isDone) {
                    ProgressManager.removeProgressListener(this);
                }
            }
        };
        ProgressManager.addProgressListener(internalProgressListener);
    }

    @Override
    public void displayImageByTransition(Context context, String url, TransitionOptions transitionOptions, ImageView imageView) {

        if (transitionOptions instanceof DrawableTransitionOptions){
            GlideApp.with(context)
                    .load(url)
                    .transition((DrawableTransitionOptions)transitionOptions)
                    .into(imageView);
        }else {
            GlideApp.with(context)
                    .asBitmap()
                    .load(url)
                    .transition(transitionOptions)
                    .into(imageView);
        }

    }

    @Override
    public void displayImageByTransition(Activity activity, String url, TransitionOptions transitionOptions, ImageView imageView) {
        if (transitionOptions instanceof DrawableTransitionOptions){
            GlideApp.with(activity)
                    .load(url)
                    .transition((DrawableTransitionOptions)transitionOptions)
                    .into(imageView);
        }else {
            GlideApp.with(activity)
                    .asBitmap()
                    .load(url)
                    .transition(transitionOptions)
                    .into(imageView);
        }
    }

    @Override
    public void displayImageByTransition(Fragment fragment, String url, TransitionOptions transitionOptions, ImageView imageView) {
        if (transitionOptions instanceof DrawableTransitionOptions){
            GlideApp.with(fragment)
                    .load(url)
                    .transition((DrawableTransitionOptions)transitionOptions)
                    .into(imageView);
        }else {
            GlideApp.with(fragment)
                    .asBitmap()
                    .load(url)
                    .transition(transitionOptions)
                    .into(imageView);
        }
    }

    @Override
    public void glidePauseRequests(Context context) {
        GlideApp.with(context).pauseRequests();
    }

    @Override
    public void glidePauseRequests(Activity activity) {
        GlideApp.with(activity).pauseRequests();
    }

    @Override
    public void glidePauseRequests(Fragment fragment) {
        GlideApp.with(fragment).pauseRequests();
    }

    @Override
    public void glideResumeRequests(Context context) {
        GlideApp.with(context).resumeRequests();
    }

    @Override
    public void glideResumeRequests(Activity activity) {
        GlideApp.with(activity).resumeRequests();
    }

    @Override
    public void glideResumeRequests(Fragment fragment) {
        GlideApp.with(fragment).resumeRequests();
    }

    /**
     *
     *  加载缩略图
     * @param context
     * @param url 图片url
     * @param backUrl 缩略图的url
     * @param thumbnailSize 如果需要放大放小的数值
     * @param imageView
     */
    @Override
    public void displayImageThumbnail(Context context, String url, String backUrl, int thumbnailSize, ImageView imageView) {

        if(thumbnailSize == 0) {
            GlideApp.with(context)
                    .load(url)
                    .thumbnail(Glide.with(context)
                            .load(backUrl))
                    .into(imageView);
        }else {

            //越小，图片越小，低网络的情况，图片越小
            GlideApp.with(context)
                    .load(url)
                    .diskCacheStrategy(DiskCacheStrategy.NONE)//为了测试不缓存
                    .thumbnail(GlideApp.with(context)
                            .load(backUrl)
                            .override(thumbnailSize))// API 来强制 Glide 在缩略图请求中加载一个低分辨率图像
                    .into(imageView);
        }

    }

    @Override
    public void displayImageThumbnail(Activity activity, String url, String backUrl, int thumbnailSize, ImageView imageView) {
        if(thumbnailSize == 0) {
            GlideApp.with(activity)
                    .load(url)
                    .thumbnail(Glide.with(activity)
                            .load(backUrl))
                    .into(imageView);
        }else {

            //越小，图片越小，低网络的情况，图片越小
            GlideApp.with(activity)
                    .load(url)
                    .diskCacheStrategy(DiskCacheStrategy.NONE)//为了测试不缓存
                    .thumbnail(GlideApp.with(activity)
                            .load(backUrl)
                            .override(thumbnailSize))// API 来强制 Glide 在缩略图请求中加载一个低分辨率图像
                    .into(imageView);
        }
    }

    @Override
    public void displayImageThumbnail(Fragment fragment, String url, String backUrl, int thumbnailSize, ImageView imageView) {
        if(thumbnailSize == 0) {
            GlideApp.with(fragment)
                    .load(url)
                    .thumbnail(Glide.with(fragment)
                            .load(backUrl))
                    .into(imageView);
        }else {

            //越小，图片越小，低网络的情况，图片越小
            GlideApp.with(fragment)
                    .load(url)
                    .diskCacheStrategy(DiskCacheStrategy.NONE)//为了测试不缓存
                    .thumbnail(GlideApp.with(fragment)
                            .load(backUrl)
                            .override(thumbnailSize))// API 来强制 Glide 在缩略图请求中加载一个低分辨率图像
                    .into(imageView);
        }
    }

    /**
     *  * thumbnail 方法有一个简化版本，它只需要一个 sizeMultiplier 参数。
     * 如果你只是想为你的加载相同的图片，但尺寸为 View 或 Target 的某个百分比的话特别有用：
     * @param fragment
     * @param url
     * @param thumbnailSize
     * @param imageView
     */
    @Override
    public void displayImageThumbnail(Fragment fragment, String url, float thumbnailSize, ImageView imageView) {
        if(thumbnailSize >= 0.0F && thumbnailSize <= 1.0F) {
            GlideApp.with(fragment)
                    .load(url)
                    .thumbnail(/*sizeMultiplier=*/ thumbnailSize)
                    .into(imageView);
        } else {
            throw new IllegalArgumentException("thumbnailSize 的值必须在0到1之间");
        }

    }

    @Override
    public void displayImageThumbnail(Activity activity, String url, float thumbnailSize, ImageView imageView) {
        if(thumbnailSize >= 0.0F && thumbnailSize <= 1.0F) {
            GlideApp.with(activity)
                    .load(url)
                    .thumbnail(/*sizeMultiplier=*/ thumbnailSize)
                    .into(imageView);
        } else {
            throw new IllegalArgumentException("thumbnailSize 的值必须在0到1之间");
        }
    }

    @Override
    public void displayImageThumbnail(Context context, String url, float thumbnailSize, ImageView imageView) {
        if(thumbnailSize >= 0.0F && thumbnailSize <= 1.0F) {
            GlideApp.with(context)
                    .load(url)
                    .thumbnail(/*sizeMultiplier=*/ thumbnailSize)
                    .into(imageView);
        } else {
            throw new IllegalArgumentException("thumbnailSize 的值必须在0到1之间");
        }
    }

    private Handler mMainThreadHandler;
    private long mTotalBytes = 0;
    private long mLastBytesRead = 0;
    private void mainThreadCallback(final String url, final long bytesRead, final long totalBytes, final boolean isDone, final GlideException exception) {
        mMainThreadHandler.post(new Runnable() {
            @Override
            public void run() {
                // TODO: 2018/4/20 放慢效果  实际的项目中给我去掉
                SystemClock.sleep(100);
                final int percent = (int) ((bytesRead * 1.0f / totalBytes) * 100.0f);
                if (onProgressListener != null) {
                    onProgressListener.onProgress(url, bytesRead, totalBytes, isDone, exception);
                }

                if (onGlideImageViewListener != null) {
                    onGlideImageViewListener.onProgress(percent, isDone, exception);
                }
            }
        });
    }
    private boolean mLastStatus = false;
    private OnProgressListener internalProgressListener;
    private OnGlideImageViewListener onGlideImageViewListener;
    private OnProgressListener onProgressListener;

    /**
     * 指定传入的那种图片的变形
     * @param placeholderResId
     * @param errorResId
     * @param transformation
     * @return
     */
    public RequestOptions requestOptionsTransform(int placeholderResId, int errorResId,Transformation transformation) {
        return new RequestOptions()
                .placeholder(placeholderResId)
                .error(errorResId).transform(transformation);
    }
    /**
     * 加载资源文件的同时，对图片进行处理
     * @param context
     * @param resId
     * @param imageView
     * @param transformations
     */
    @Override
    public void displayImageInResource(Context context, int resId, ImageView imageView, BitmapTransformation transformations) {
        GlideApp.with(context).load(resId).diskCacheStrategy(DiskCacheStrategy.NONE).transform(transformations).into(imageView);
    }

    @Override
    public void displayImageInResource(Fragment fragment, int resId,  ImageView imageView, BitmapTransformation transformations) {
        GlideApp.with(fragment).load(resId).diskCacheStrategy(DiskCacheStrategy.NONE).transform(transformations).into(imageView);
    }

    /**
     * 当传入到其他的东西的时候，我要保证图片不变形
     * @param placeholderResId
     * @param errorResId
     * @param transformation
     * @return
     */
    public RequestOptions requestOptionsNoTransform(int placeholderResId, int errorResId, Transformation<Bitmap> transformation) {
        return new RequestOptions()
                .placeholder(placeholderResId)
                .error(errorResId).transform(transformation);
    }
    /**
     * 加载资源文件失败了，加载中的默认图和失败的图片
     * @param context
     * @param resId
     * @param imageView
     * @param defRes
     */
    @Override
    public void displayImageInResource(Context context, int resId,  ImageView imageView, int defRes) {
        GlideApp.with(context).load(resId).diskCacheStrategy(DiskCacheStrategy.NONE).placeholder(defRes).error(defRes).into(imageView);
    }

    @Override
    public void displayImageInResource(Fragment fragment, int resId, ImageView imageView, int defRes) {
        GlideApp.with(fragment).load(resId).diskCacheStrategy(DiskCacheStrategy.NONE).placeholder(defRes).error(defRes).into(imageView);
    }
    //关心context
    @Override
    public void displayImageInResource(Context context, int resId,  ImageView imageView, int defRes, BitmapTransformation transformations) {
        GlideApp.with(context).load(resId).diskCacheStrategy(DiskCacheStrategy.NONE).placeholder(defRes).error(defRes).transform(transformations).into(imageView);
    }
    //关心fragment
    @Override
    public void displayImageInResource(Fragment fragment, int resId,  ImageView imageView, int defRes, BitmapTransformation transformations) {
        GlideApp.with(fragment).load(resId).diskCacheStrategy(DiskCacheStrategy.NONE).placeholder(defRes).error(defRes).transform(transformations).into(imageView);
    }


}

```

ImageLoaderV4调用类

```
package code.shiming.com.imageloader471;

import android.app.Activity;
import android.content.Context;
import android.graphics.Bitmap;
import android.support.v4.app.Fragment;
import android.widget.ImageView;

import com.bumptech.glide.TransitionOptions;
import com.bumptech.glide.load.Transformation;
import com.bumptech.glide.load.engine.DiskCacheStrategy;
import com.bumptech.glide.load.resource.bitmap.BitmapTransformation;

import java.io.File;

import code.shiming.com.imageloader471.listener.IGetBitmapListener;
import code.shiming.com.imageloader471.listener.IGetDrawableListener;
import code.shiming.com.imageloader471.listener.IImageLoaderListener;
import code.shiming.com.imageloader471.listener.ImageSize;
import code.shiming.com.imageloader471.okhttp.OnGlideImageViewListener;
import code.shiming.com.imageloader471.okhttp.OnProgressListener;


/**
 * Created by shiming on 2016/10/26.
 */
public class ImageLoaderV4 implements IImageLoaderClient {
    /**
     * volatile 关键字：我个人理解的是：使用volatile关键字的程序在并发时能够正确执行。
     * 但是它不能够代替synchronized关键字。在网上找到这么一句话：
     * 观察加入volatile关键字和没有加入volatile关键字时所生成的汇编代码发现，
     * 加入volatile关键字时，会多出一个lock前缀指令lock前缀指令实际上相当于一个内存屏障（也成内存栅栏），
     * 内存屏障会提供3个功能：1）它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，
     * 也不会把前面的指令排到内存屏障的后面；即在执行到内存屏障这句指令时，在它前面的操作已经全部完成；
     * 2）它会强制将对缓存的修改操作立即写入主存；3）如果是写操作，它会导致其他CPU中对应的缓存行无效。
     */
    private volatile static ImageLoaderV4 instance;
    private IImageLoaderClient client;

    private ImageLoaderV4() {
        client = new GlideImageLoaderClient();
    }

    /***
     * 设置 图片加载库客户端
     */

    public void setImageLoaderClient(Context context, IImageLoaderClient client) {
        if (this.client != null) {
            this.client.clearMemoryCache(context);
        }

        if (this.client != client) {
            this.client = client;
            if (this.client != null) {
                this.client.init(context);
            }
        }

    }
    /**
     * 因为每次调用实例都需要判断同步锁，很多项目包括很多人都是用这种的
     * 双重判断校验的方法，这种的方法看似很完美的解决了效率的问题，但是它
     * 在并发量不多，安全性不太高的情况下能完美的运行，但是，
     * 在jvm编译的过程中会出现指令重排的优化过程，这就会导致singleton实际上
     * 没有被初始化，就分配了内存空间，也就是说singleton！=null但是又没有被初始化，
     * 这就会导致返回的singletonthird返回的是不完整的
     * 参考：http://www.360doc.com/content/11/0810/12/1542811_139352888.shtml
     * 建议使用内部类实现单利模式
     */
    public static ImageLoaderV4 getInstance() {
        if (instance == null) {
            synchronized (ImageLoaderV4.class) {
                if (instance == null) {
                    instance = new ImageLoaderV4();
                }
            }
        }
        return instance;
    }

    @Override
    public void init(Context context) {

    }

    @Override
    public void destroy(Context context) {
        if (client != null) {
            client.destroy(context);
            client = null;
        }

        instance = null;
    }

    @Override
    public File getCacheDir(Context context) {
        if (client != null) {
            return client.getCacheDir(context);
        }
        return null;
    }

    @Override
    public void clearMemoryCache(Context context) {
        if (client != null) {
            client.clearMemoryCache(context);
        }
    }

    @Override
    public void clearDiskCache(Context context) {
        if (client != null) {
            client.clearDiskCache(context);
        }
    }

    @Override
    public Bitmap getBitmapFromCache(Context context, String url) {
        if (client != null) {
            return client.getBitmapFromCache(context, url);
        }
        return null;
    }

    /**
     * 不是
     * @param context
     * @param url
     * @param listener
     */
    @Override
    public void getBitmapFromCache(Context context, String url, IGetBitmapListener listener) {
        if (client != null) {
            client.getBitmapFromCache(context, url, listener);
        }
    }

    @Override
    public void displayImage(Context context, int resId, ImageView imageView) {
        if (client != null) {
            client.displayImage(context, resId, imageView);
        }
    }

    @Override
    public void displayImage(Context context, String url, ImageView imageView) {
        if (client != null) {
            client.displayImage(context, url, imageView);
        }
    }

    @Override
    public void displayImage(Context context, String url, ImageView imageView, boolean isCache) {
        if (client != null) {
            client.displayImage(context, url, imageView,isCache);
        }
    }

    @Override
    public void displayImage(Fragment fragment, String url, ImageView imageView) {
        if (client != null) {
            client.displayImage(fragment, url, imageView);
        }
    }

    @Override
    public void displayImage(Context context, String url, ImageView imageView, int defRes) {
        if (client != null) {
            client.displayImage(context, url, imageView, defRes);
        }
    }

    @Override
    public void displayImage(Fragment fragment, String url, ImageView imageView, int defRes) {
        if (client != null) {
            client.displayImage(fragment, url, imageView, defRes);
        }
    }

    @Override
    public void displayImage(Context context, String url, ImageView imageView, int defRes,BitmapTransformation transformations) {
        if (client != null) {
            client.displayImage(context, url, imageView, defRes, transformations);
        }
    }

    @Override
    public void displayImage(Fragment fragment, String url, ImageView imageView, int defRes, BitmapTransformation transformations) {
        if (client != null) {
            client.displayImage(fragment, url, imageView, defRes, transformations);
        }
    }

    @Override
    public void displayImage(Context context, String url, ImageView imageView, int defRes, ImageSize size) {
        if (client != null) {
            client.displayImage(context, url, imageView, defRes, size);
        }
    }

    @Override
    public void displayImage(Fragment fragment, String url, ImageView imageView, int defRes, ImageSize size) {
        if (client != null) {
            client.displayImage(fragment, url, imageView, defRes, size);
        }
    }

    @Override
    public void displayImage(Context context, String url, ImageView imageView, int defRes, boolean cacheInMemory) {
        if (client != null) {
            client.displayImage(context, url, imageView, defRes, cacheInMemory);
        }
    }

    @Override
    public void displayImage(Fragment fragment, String url, ImageView imageView, int defRes, boolean cacheInMemory) {
        if (client != null) {
            client.displayImage(fragment, url, imageView, defRes, cacheInMemory);
        }
    }

    @Override
    public void displayImage(Context context, String url, ImageView imageView, IImageLoaderListener listener) {
        if (client != null) {
            client.displayImage(context, url, imageView, listener);
        }
    }

    @Override
    public void displayImage(Fragment fragment, String url, ImageView imageView, IImageLoaderListener listener) {
        if (client != null) {
            client.displayImage(fragment, url, imageView, listener);
        }
    }

    @Override
    public void displayImage(Context context, String url, ImageView imageView, int defRes, IImageLoaderListener listener) {
        if (client != null) {
            client.displayImage(context, url, imageView, defRes, listener);
        }
    }

    @Override
    public void displayImage(Fragment fragment, String url, ImageView imageView, int defRes, IImageLoaderListener listener) {
        if (client != null) {
            client.displayImage(fragment, url, imageView, defRes, listener);
        }
    }

    @Override
    public void displayCircleImage(Context context, String url, ImageView imageView, int defRes) {
        if (client != null) {
            client.displayCircleImage(context, url, imageView, defRes);
        }
    }

    @Override
    public void displayCircleImage(Fragment fragment, String url, ImageView imageView, int defRes) {
        if (client != null) {
            client.displayCircleImage(fragment, url, imageView, defRes);
        }
    }

    @Override
    public void displayRoundImage(Context context, String url, ImageView imageView, int defRes, int radius) {
        if (client != null) {
            client.displayRoundImage(context, url, imageView, defRes, radius);
        }
    }

    @Override
    public void displayBlurImage(Context context, String url, int blurRadius, final IGetDrawableListener listener) {
        if (client != null) {
            client.displayBlurImage(context, url, blurRadius, listener);
        }
    }

    @Override
    public void displayRoundImage(Fragment fragment, String url, ImageView imageView, int defRes, int radius) {
        if (client != null) {
            client.displayRoundImage(fragment, url, imageView, defRes, radius);
        }
    }

    @Override
    public void displayBlurImage(Context context, String url, ImageView imageView, int defRes, int blurRadius) {
        if (client != null) {
            client.displayBlurImage(context, url, imageView, defRes, blurRadius);
        }
    }

    @Override
    public void displayBlurImage(Context context, int resId, ImageView imageView, int blurRadius) {
        if (client != null) {
            client.displayBlurImage(context, resId, imageView, blurRadius);
        }
    }

    @Override
    public void displayBlurImage(Fragment fragment, String url, ImageView imageView, int defRes, int blurRadius) {
        if (client != null) {
            client.displayBlurImage(fragment, url, imageView, defRes, blurRadius);
        }
    }

    @Override
    public void displayImageInResource(Context context, int resId,  ImageView imageView) {
        if (client != null) {
            client.displayImageInResource(context, resId,  imageView);
        }
    }

    @Override
    public void displayImageInResource(Fragment fragment, int resId,  ImageView imageView) {
        if (client != null) {
            client.displayImageInResource(fragment, resId,  imageView);
        }
    }

    @Override
    public void displayImageInResource(Context context, int resId, ImageView imageView, BitmapTransformation transformations) {
        if (client != null) {
            client.displayImageInResource(context, resId, imageView, transformations);
        }
    }

    @Override
    public void displayImageInResource(Fragment fragment, int resId,  ImageView imageView, BitmapTransformation transformations) {
        if (client != null) {
            client.displayImageInResource(fragment, resId, imageView, transformations);
        }
    }

    @Override
    public void displayImageInResource(Context context, int resId,  ImageView imageView, int defRes) {
        if (client != null) {
            client.displayImageInResource(context, resId, imageView, defRes);
        }
    }

    @Override
    public void displayImageInResource(Fragment fragment, int resId,  ImageView imageView, int defRes) {
        if (client != null) {
            client.displayImageInResource(fragment, resId, imageView, defRes);
        }
    }

    @Override
    public void displayImageInResource(Context context, int resId,  ImageView imageView, int defRes, BitmapTransformation transformations) {
        if (client != null) {
            client.displayImageInResource(context, resId, imageView, defRes, transformations);
        }
    }

    @Override
    public void displayImageInResource(Fragment fragment, int resId,  ImageView imageView, int defRes, BitmapTransformation transformations) {
        if (client != null) {
            client.displayImageInResource(fragment, resId,  imageView, defRes, transformations);
        }
    }

    @Override
    public void displayImageInResourceTransform(Activity activity, int resId, ImageView imageView, Transformation transformation, int errorResId) {
        if (client != null) {
            client.displayImageInResourceTransform(activity, resId,  imageView, transformation, errorResId);
        }
    }

    @Override
    public void displayImageInResourceTransform(Context context, int resId, ImageView imageView, Transformation transformation, int errorResId) {
        if (client != null) {
            client.displayImageInResourceTransform(context, resId,  imageView, transformation, errorResId);
        }
    }

    @Override
    public void displayImageInResourceTransform(Fragment fragment, int resId, ImageView imageView, Transformation transformation, int errorResId) {
        if (client != null) {
            client.displayImageInResourceTransform(fragment, resId,  imageView, transformation, errorResId);
        }
    }

    @Override
    public void displayImageByNet(Context context, String url, ImageView imageView, int defRes, Transformation transformation) {
        if (client != null) {
            client.displayImageByNet(context, url,  imageView, defRes, transformation);
        }
    }

    @Override
    public void displayImageByNet(Fragment fragment, String url, ImageView imageView, int defRes, Transformation transformation) {
        if (client != null) {
            client.displayImageByNet(fragment, url,  imageView, defRes, transformation);
        }
    }

    @Override
    public void displayImageByNet(Activity activity, String url, ImageView imageView, int defRes, Transformation transformation) {
        if (client != null) {
            client.displayImageByNet(activity, url,  imageView, defRes, transformation);
        }
    }
    /**尽管及时取消不必要的加载是很好的实践，但这并不是必须的操作。
     * 实际上，当 Glide.with() 中传入的 Activity 或 Fragment 实例销毁时，
     * Glide 会自动取消加载并回收资源。这里我隐藏了api的调用
     * {@hide}
     */
    @Override
    public void clear(Activity activity, ImageView imageView) {
        if (client != null) {
            client.clear(activity, imageView);
        }
    }
    /**尽管及时取消不必要的加载是很好的实践，但这并不是必须的操作。
     * 实际上，当 Glide.with() 中传入的 Activity 或 Fragment 实例销毁时，
     * Glide 会自动取消加载并回收资源。这里我隐藏了api的调用
     * {@hide}
     */
    @Override
    public void clear(Context context, ImageView imageView) {
        if (client != null) {
            client.clear(context, imageView);
        }
    }
    /**尽管及时取消不必要的加载是很好的实践，但这并不是必须的操作。
     * 实际上，当 Glide.with() 中传入的 Activity 或 Fragment 实例销毁时，
     * Glide 会自动取消加载并回收资源。这里我隐藏了api的调用
     * {@hide}
     */
    @Override
    public void clear(Fragment fragment, ImageView imageView) {
        if (client != null) {
            client.clear(fragment, imageView);
        }
    }

    /**
     * 指定选择哪种缓存的策略
     * @param fragment
     * @param url
     * @param diskCacheStrategy
     * @param imageView
     */
    @Override
    public void displayImageByDiskCacheStrategy(Fragment fragment, String url, DiskCacheStrategy diskCacheStrategy, ImageView imageView) {
        if (client != null) {
            client.displayImageByDiskCacheStrategy(fragment, url,  diskCacheStrategy, imageView);
        }
    }

    @Override
    public void displayImageByDiskCacheStrategy(Activity activity, String url, DiskCacheStrategy diskCacheStrategy, ImageView imageView) {
        if (client != null) {
            client.displayImageByDiskCacheStrategy(activity, url,  diskCacheStrategy, imageView);
        }
    }

    @Override
    public void displayImageByDiskCacheStrategy(Context context, String url, DiskCacheStrategy diskCacheStrategy, ImageView imageView) {
        if (client != null) {
            client.displayImageByDiskCacheStrategy(context, url,  diskCacheStrategy, imageView);
        }
    }

    @Override
    public void disPlayImageOnlyRetrieveFromCache(Fragment fragment, String url, ImageView imageView) {
        if (client != null) {
            client.disPlayImageOnlyRetrieveFromCache(fragment, url, imageView);
        }
    }

    @Override
    public void disPlayImageOnlyRetrieveFromCache(Activity activity, String url, ImageView imageView) {
        if (client != null) {
            client.disPlayImageOnlyRetrieveFromCache(activity, url, imageView);
        }
    }

    @Override
    public void disPlayImageOnlyRetrieveFromCache(Context context, String url, ImageView imageView) {
        if (client != null) {
            client.disPlayImageOnlyRetrieveFromCache(context, url, imageView);
        }
    }

    @Override
    public void disPlayImageSkipMemoryCache(Fragment fragment, String url, ImageView imageView, boolean skipflag, boolean diskCacheStratey) {
        if (client != null) {
            client.disPlayImageSkipMemoryCache(fragment, url, imageView,skipflag,diskCacheStratey);
        }
    }

    @Override
    public void disPlayImageSkipMemoryCache(Activity activity, String url, ImageView imageView, boolean skipflag, boolean diskCacheStratey) {
        if (client != null) {
            client.disPlayImageSkipMemoryCache(activity, url, imageView,skipflag,diskCacheStratey);
        }
    }

    @Override
    public void disPlayImageSkipMemoryCache(Context context, String url, ImageView imageView, boolean skipflag, boolean diskCacheStratey) {
        if (client != null) {
            client.disPlayImageSkipMemoryCache(context, url, imageView,skipflag,diskCacheStratey);
        }
    }

    @Override
    public void disPlayImageErrorReload(Fragment fragment, String url, String fallbackUrl, ImageView imageView) {
        if (client != null) {
            client.disPlayImageErrorReload(fragment, url,fallbackUrl, imageView);
        }
    }

    @Override
    public void disPlayImageErrorReload(Activity activity, String url, String fallbackUrl, ImageView imageView) {
        if (client != null) {
            client.disPlayImageErrorReload(activity, url,fallbackUrl, imageView);
        }
    }

    @Override
    public void disPlayImageErrorReload(Context context, String url, String fallbackUrl, ImageView imageView) {
        if (client != null) {
            client.disPlayImageErrorReload(context, url,fallbackUrl, imageView);
        }
    }

    @Override
    public void disPlayImagedisallowHardwareConfig(Fragment fragment, String url, ImageView imageView) {
        if (client != null) {
            client.disPlayImagedisallowHardwareConfig(fragment, url,imageView);
        }
    }

    @Override
    public void disPlayImagedisallowHardwareConfig(Activity activity, String url, ImageView imageView) {
        if (client != null) {
            client.disPlayImagedisallowHardwareConfig(activity, url,imageView);
        }
    }

    @Override
    public void disPlayImagedisallowHardwareConfig(Context context, String url, ImageView imageView) {
        if (client != null) {
            client.disPlayImagedisallowHardwareConfig(context, url,imageView);
        }
    }

    @Override
    public void disPlayImageProgress(Context context, String url, ImageView imageView, int placeholderResId, int errorResId, OnGlideImageViewListener listener) {
        if (client != null) {
            client.disPlayImageProgress(context, url,imageView,placeholderResId,errorResId,listener);
        }
    }

    @Override
    public void disPlayImageProgress(Activity activity, String url, ImageView imageView, int placeholderResId, int errorResId, OnGlideImageViewListener listener) {
        if (client != null) {
            client.disPlayImageProgress(activity, url,imageView,placeholderResId,errorResId,listener);
        }
    }

    @Override
    public void disPlayImageProgress(Fragment fragment, String url, ImageView imageView, int placeholderResId, int errorResId, OnGlideImageViewListener listener) {
        if (client != null) {
            client.disPlayImageProgress(fragment, url,imageView,placeholderResId,errorResId,listener);
        }
    }

    @Override
    public void disPlayImageProgressByOnProgressListener(Context context, String url, ImageView imageView, int placeholderResId, int errorResId, OnProgressListener onProgressListener) {
        if (client != null) {
            client.disPlayImageProgressByOnProgressListener(context, url,imageView,placeholderResId,errorResId,onProgressListener);
        }
    }

    @Override
    public void disPlayImageProgressByOnProgressListener(Activity activity, String url, ImageView imageView, int placeholderResId, int errorResId, OnProgressListener onProgressListener) {
        if (client != null) {
            client.disPlayImageProgressByOnProgressListener(activity, url,imageView,placeholderResId,errorResId,onProgressListener);
        }
    }

    /**
     * 需要监听 总的字节数，和文件的大小，同时也可以扩展为，加载本地图片
     * @param fragment
     * @param url
     * @param imageView
     * @param placeholderResId
     * @param errorResId
     * @param onProgressListener
     */
    @Override
    public void disPlayImageProgressByOnProgressListener(Fragment fragment, String url, ImageView imageView, int placeholderResId, int errorResId, OnProgressListener onProgressListener) {
        if (client != null) {
            client.disPlayImageProgressByOnProgressListener(fragment, url,imageView,placeholderResId,errorResId,onProgressListener);
        }
    }
    /**
     * 过渡选项
     TransitionOptions 用于决定你的加载完成时会发生什么。
     使用 TransitionOption 可以应用以下变换：
     View淡入
     与占位符交叉淡入
     或者什么都不发生
     从白色 慢慢变透明 5s的间隔
     GlideApp.with(this)
     .load(ur11)
     .transition(withCrossFade(5000)) 可以传入过度的时间
     .into(mImageView_7);
     */

    @Override
    public void displayImageByTransition(Context context, String url, TransitionOptions transitionOptions, ImageView imageView) {
        if (client != null) {
            client.displayImageByTransition(context, url,transitionOptions,imageView);
        }

    }

    @Override
    public void displayImageByTransition(Activity activity, String url, TransitionOptions transitionOptions, ImageView imageView) {
        if (client != null) {
            client.displayImageByTransition(activity, url,transitionOptions,imageView);
        }
    }

    @Override
    public void displayImageByTransition(Fragment fragment, String url, TransitionOptions transitionOptions, ImageView imageView) {
        if (client != null) {
            client.displayImageByTransition(fragment, url,transitionOptions,imageView);
        }
    }

    @Override
    public void glidePauseRequests(Context context) {
        if (client != null) {
            client.glidePauseRequests(context);
        }
    }

    @Override
    public void glidePauseRequests(Activity activity) {
        if (client != null) {
            client.glidePauseRequests(activity);
        }
    }

    @Override
    public void glidePauseRequests(Fragment fragment) {
        if (client != null) {
            client.glidePauseRequests(fragment);
        }
    }

    @Override
    public void glideResumeRequests(Context context) {
        if (client != null) {
            client.glideResumeRequests(context);
        }
    }

    @Override
    public void glideResumeRequests(Activity activity) {
        if (client != null) {
            client.glideResumeRequests(activity);
        }
    }

    @Override
    public void glideResumeRequests(Fragment fragment) {
        if (client != null) {
            client.glideResumeRequests(fragment);
        }
    }

    @Override
    public void displayImageThumbnail(Context context, String url, String backUrl, int thumbnailSize, ImageView imageView) {
        if (client != null) {
            client.displayImageThumbnail(context,url,backUrl,thumbnailSize,imageView);
        }
    }

    @Override
    public void displayImageThumbnail(Activity activity, String url, String backUrl, int thumbnailSize, ImageView imageView) {
        if (client != null) {
            client.displayImageThumbnail(activity,url,backUrl,thumbnailSize,imageView);
        }
    }

    @Override
    public void displayImageThumbnail(Fragment fragment, String url, String backUrl, int thumbnailSize, ImageView imageView) {
        if (client != null) {
            client.displayImageThumbnail(fragment,url,backUrl,thumbnailSize,imageView);
        }
    }

    /**
     * 没有地址也需要指定缩略图
     * @param fragment
     * @param url
     * @param thumbnailSize
     * @param imageView
     */
    @Override
    public void displayImageThumbnail(Fragment fragment, String url, float thumbnailSize, ImageView imageView) {
        if (client != null) {
            client.displayImageThumbnail(fragment,url,thumbnailSize,imageView);
        }
    }

    @Override
    public void displayImageThumbnail(Activity activity, String url, float thumbnailSize, ImageView imageView) {
        if (client != null) {
            client.displayImageThumbnail(activity,url,thumbnailSize,imageView);
        }
    }

    @Override
    public void displayImageThumbnail(Context context, String url, float thumbnailSize, ImageView imageView) {
        if (client != null) {
            client.displayImageThumbnail(context,url,thumbnailSize,imageView);
        }
    }
}

```

关于图片下载进度的监听,在AppGlideModuleProgress 需要初始化ProgressManager
```
@GlideModule
public class AppGlideModuleProgress extends AppGlideModule {
    /**
     OkHttp 是一个底层网络库(相较于 Cronet 或 Volley 而言)，尽管它也包含了 SPDY 的支持。
     OkHttp 与 Glide 一起使用可以提供可靠的性能，并且在加载图片时通常比 Volley 产生的垃圾要少。
     对于那些想要使用比 Android 提供的 HttpUrlConnection 更 nice 的 API，
      或者想确保网络层代码不依赖于 app 安装的设备上 Android OS 版本的应用，OkHttp 是一个合理的选择。
      如果你已经在 app 中某个地方使用了 OkHttp ，这也是选择继续为 Glide 使用 OkHttp 的一个很好的理由，就像选择其他网络库一样。
     添加 OkHttp 集成库的 Gradle 依赖将使 Glide 自动开始使用 OkHttp 来加载所有来自 http 和 https URL 的图片
     * @param context
     * @param glide
     * @param registry
     */
    @Override
    public void registerComponents(Context context, Glide glide, Registry registry) {
        super.registerComponents(context, glide, registry);
        registry.replace(GlideUrl.class, InputStream.class, new OkHttpUrlLoader.Factory(ProgressManager.getOkHttpClient()));
    }
}
```

ProgressManager

```
public class ProgressManager {

    private static List<WeakReference<OnProgressListener>> listeners = Collections.synchronizedList(new ArrayList<WeakReference<OnProgressListener>>());
    private static OkHttpClient okHttpClient;

    private ProgressManager() {
    }

    public static OkHttpClient getOkHttpClient() {
        if (okHttpClient == null) {
            okHttpClient = new OkHttpClient.Builder()
                    .addNetworkInterceptor(new Interceptor() {
                        @Override
                        public Response intercept(@NonNull Chain chain) throws IOException {
                            Request request = chain.request();
                            Response response = chain.proceed(request);
                            return response.newBuilder()
                                    .body(new ProgressResponseBody(request.url().toString(), response.body(), LISTENER))
                                    .build();
                        }
                    })
                    .build();
        }
        return okHttpClient;
    }

    private static final OnProgressListener LISTENER = new OnProgressListener() {
        @Override
        public void onProgress(String imageUrl, long bytesRead, long totalBytes, boolean isDone, GlideException exception) {
            if (listeners == null || listeners.size() == 0) return;

            for (int i = 0; i < listeners.size(); i++) {
                WeakReference<OnProgressListener> listener = listeners.get(i);
                OnProgressListener progressListener = listener.get();
                if (progressListener == null) {
                    listeners.remove(i);
                } else {
                    progressListener.onProgress(imageUrl, bytesRead, totalBytes, isDone, exception);
                }
            }
        }
    };

    public static void addProgressListener(OnProgressListener progressListener) {
        if (progressListener == null) return;

        if (findProgressListener(progressListener) == null) {
            listeners.add(new WeakReference<>(progressListener));
        }
    }

    public static void removeProgressListener(OnProgressListener progressListener) {
        if (progressListener == null) return;

        WeakReference<OnProgressListener> listener = findProgressListener(progressListener);
        if (listener != null) {
            listeners.remove(listener);
        }
    }

    private static WeakReference<OnProgressListener> findProgressListener(OnProgressListener listener) {
        if (listener == null) return null;
        if (listeners == null || listeners.size() == 0) return null;

        for (int i = 0; i < listeners.size(); i++) {
            WeakReference<OnProgressListener> progressListener = listeners.get(i);
            if (progressListener.get() == listener) {
                return progressListener;
            }
        }
        return null;
    }
}

```

ProgressResponseBody 
```
public class ProgressResponseBody extends ResponseBody {

    private String imageUrl;
    private ResponseBody responseBody;
    private OnProgressListener progressListener;
    private BufferedSource bufferedSource;

    public ProgressResponseBody(String url, ResponseBody responseBody, OnProgressListener progressListener) {
        this.imageUrl = url;
        this.responseBody = responseBody;
        this.progressListener = progressListener;
    }

    @Override
    public MediaType contentType() {
        return responseBody.contentType();
    }

    @Override
    public long contentLength() {
        return responseBody.contentLength();
    }

    @Override
    public BufferedSource source() {
        if (bufferedSource == null) {
            bufferedSource = Okio.buffer(source(responseBody.source()));
        }
        return bufferedSource;
    }

    private Source source(Source source) {
        return new ForwardingSource(source) {
            long totalBytesRead = 0;

            @Override
            public long read(@NonNull Buffer sink, long byteCount) throws IOException {
                long bytesRead = super.read(sink, byteCount);
                totalBytesRead += (bytesRead == -1) ? 0 : bytesRead;

                if (progressListener != null) {
                    progressListener.onProgress(imageUrl, totalBytesRead, contentLength(), (bytesRead == -1), null);
                }
                return bytesRead;
            }
        };
    }
}

```
OnProgressListener  
```
public interface OnProgressListener {
    /**
     * 
      * @param imageUrl 图片地址
     * @param bytesRead 下载了多少字节
     * @param totalBytes 总共的大小
     * @param isDone 是否完成
     * @param exception 异常
     */
    void onProgress(String imageUrl, long bytesRead, long totalBytes, boolean isDone, GlideException exception);
}

```
OnGlideImageViewListener 
```
public interface OnGlideImageViewListener {
    /**
     * 
     * @param percent 下载进度的百分比，不关心，大小
     * @param isDone 是否完成
     * @param exception 异常
     */
    void onProgress(int percent, boolean isDone, GlideException exception);
}
```

glide4.X优点
 1、新的文档，用户可以通过提交请求到Glide’s gh-pages分支贡献。
2、用户可以添加新类型或自定义选项集来轻松地自定义Glide流畅的API。大量简化个人请求类型，确保选项始终如一，易于使用，即使您正在加载不同类型的资源。
3、各种性能改进，包括在下载采样图像时大量减少垃圾，更加智能的默认磁盘缓存策略，以及加载GIF时性能提升。
4、改进了视图大小和布局的处理，特别是在RecyclerView中。
官方文档：https://muyangmin.github.io/glide-docs-cn/

Glide V4变化较大的是库处理选项的方式。在Glide v3中，选项是由一系列复杂的多类型构建器单独处理的。在Glide v4中，这些已被具有单一类型的单个构建器和可以提供给构建器的一系列选项的对象所替代。Glide 生成的API通过将选项对象和任何包含的集成库与构建器的选项合并，来创建单个流畅的API。 可以简单理解为，Glide v4 将Glide V3中Glide.with()实现的一系列复杂功能拆分成一些独立的API。

建议看官方文档，虽然需要点时间！

#####最后总结下Glide4.X对比Glide3.X的区别：
######1、选项(Options)
Glide v4 中的一个比较大的改动是Glide库处理选项(`centerCrop()`, `placeholder()` 等)的方式。在 v3 版本中，选项由一系列复杂的异构建造者(multityped builders)单独处理。在新版本中，由一个单一类型的唯一一个建造者接管一系列选项对象。Glide 的[generated API](https://muyangmin.github.io/glide-docs-cn/doc/generatedapi.html)进一步简化了这个操作：它会合并传入建造者的选项对象和任何已包含的集成库里的选项，以生成一个流畅的 API。
######2、RequestBuilder
对于这类方法：

```
listener()
thumbnail()
load()
into()

```
在 Glide v4 版本中，只存在一个 [`RequestBuilder`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html) 对应一个你正在试图加载的类型(`Bitmap`, `Drawable`, `GifDrawable` 等)。 `RequestBuilder` 可以直接访问对这个加载过程有影响的选项，包括你想加载的数据模型（url, uri等），可能存在的[`缩略图`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html#thumbnail-com.bumptech.glide.RequestBuilder-)请求，以及任何的[`监听器`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html#listener-com.bumptech.glide.request.RequestListener-)。`RequestBuilder`也是你使用 [`into()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html#into-Y-) 或者 [`preload()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html#preload-int-int-) 方法开始加载的地方：
```
RequestBuilder<Drawable> requestBuilder = Glide.with(fragment)
    .load(url);

requestBuilder
    .thumbnail(Glide.with(fragment)
        .load(thumbnailUrl))
    .listener(requestListener)
    .load(url)
    .into(imageView);
```
######3、请求选项
对于这类方法：

```
centerCrop()
placeholder()
error()
priority()
diskCacheStrategy()

```

大部分选项被移动到了一个单独的称为 [`RequestOptions`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestOptions.html) 的对象中，

```
RequestOptions options = new RequestOptions()
    .centerCrop()
    .placeholder(R.drawable.placeholder)
    .error(R.drawable.error)
    .priority(Priority.HIGH);

```

`RequestOptions` 允许你一次指定一系列的选项，然后对多个加载重用它们：

```
RequestOptions myOptions = new RequestOptions()
    .fitCenter()
    .override(100, 100);

Glide.with(fragment)
    .load(url)
    .apply(myOptions)
    .into(drawableView);

Glide.with(fragment)
    .asBitmap()
    .apply(myOptions)
    .load(url)
    .into(bitmapView);
```
######4、变换
Glide v4 里的 [`Transformations`](https://muyangmin.github.io/glide-docs-cn/javadocs/410/com/bumptech/glide/load/Transformation.html) 现在会替换之前设置的任何变换。在 Glide v4 中，如果你想应用超过一个的 [`Transformation`](https://muyangmin.github.io/glide-docs-cn/javadocs/410/com/bumptech/glide/load/Transformation.html)，你需要使用 [`transforms()`](https://muyangmin.github.io/glide-docs-cn/javadocs/410/com/bumptech/glide/request/RequestOptions.html#transforms-com.bumptech.glide.load.Transformation...-) 方法：

```
Glide.with(fragment)
  .load(url)
  .apply(new RequestOptions().transforms(new CenterCrop(), new RoundedCorners(20)))
  .into(target);

```

或使用 [generated API](https://muyangmin.github.io/glide-docs-cn/doc/generatedapi.html):

```
GlideApp.with(fragment)
  .load(url)
  .transforms(new CenterCrop(), new RoundedCorners(20))
  .into(target);
```
######5、解码格式
在 Glide v3， 默认的 [`DecodeFormat`](https://muyangmin.github.io/glide-docs-cn/javadocs/410/com/bumptech/glide/load/DecodeFormat.html) 是 [`DecodeFormat.PREFER_RGB_565`](https://muyangmin.github.io/glide-docs-cn/javadocs/410/com/bumptech/glide/load/DecodeFormat.html#PREFER_RGB_565)，它将使用 [`Bitmap.Config.RGB_565`]，除非图片包含或可能包含透明像素。对于给定的图片尺寸，`RGB_565` 只使用 [`Bitmap.Config.ARGB_8888`] 一半的内存，但对于特定的图片有明显的画质问题，包括条纹(banding)和着色(tinting)。为了避免`RGB_565`的画质问题，Glide 现在默认使用 `ARGB_8888`。结果是，图片质量变高了，但内存使用也增加了。

要将 Glide v4 默认的 [`DecodeFormat`](https://muyangmin.github.io/glide-docs-cn/javadocs/410/com/bumptech/glide/load/DecodeFormat.html) 改回 [`DecodeFormat.PREFER_RGB_565`](https://muyangmin.github.io/glide-docs-cn/javadocs/410/com/bumptech/glide/load/DecodeFormat.html#PREFER_RGB_565)，请在 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 中应用一个 `RequestOption`：

```
@GlideModule
public final class YourAppGlideModule extends GlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    builder.setDefaultRequestOptions(new RequestOptions().format(DecodeFormat.PREFER_RGB_565));
  }
}

```

关于使用 [`AppGlideModules`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 的更多信息，请查阅 [配置](https://muyangmin.github.io/glide-docs-cn/doc/configuration.html) 页面。请注意，为了让 Glide 发现你的 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 实现，你必须确保添加了对 Glide 的注解解析器的依赖。关于如何设置这个库的更多信息，请查看 [下载和设置](https://muyangmin.github.io/glide-docs-cn/doc/download-setup.html)。
######6、过渡选项
对于这类方法：

```
crossFade()
animate()

```

控制从占位符到图片和/或缩略图到全图的交叉淡入和其他类型变换的选项，被移动到了 [`TransitionOptions`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/resource/bitmap/BitmapTransitionOptions.html) 中。

要应用过渡（之前的动画），请使用下列选项中符合你请求的资源类型的一个：

*   [`GenericTransitionOptions`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/GenericTransitionOptions.html)
*   [`DrawableTransitionOptions`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/resource/drawable/DrawableTransitionOptions.html)
*   [`BitmapTransitionOptions`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/resource/bitmap/BitmapTransitionOptions.html)

如果你想移除任何默认的过渡，可以使用 `TransitionOptions.dontTransition()`][17](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/TransitionOptions.html#dontTransition--) 。

过渡动画通过 [`RequestBuilder`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html) 应用到请求上：

```
Glide.with(fragment)
    .load(url)
    .transition(withCrossFade(R.anim.fade_in, 300));
```
######7、交叉淡入 (Cross fade)
不同于 Glide v3，Glide v4 将不会默认应用交叉淡入或任何其他的过渡效果。每个请求必须手动应用过渡。

要为一个特定的加载应用一个交叉淡入变换效果，你可以使用：
```
import static com.bumptech.glide.load.resource.drawable.DrawableTransitionOptions.withCrossFade;

Glide.with(fragment)
  .load(url)
  .transition(withCrossFade())
  .into(imageView);
```

或:
```
Glide.with(fragment)
  .load(url)
  .transition(
      new DrawableTransitionOptions
        .crossFade())
  .into(imageView);
```
######8、Generated API
为了让使用 Glide v4 更简单轻松，Glide 现在也提供了一套可以为应用定制化生成的 API。应用可以通过包含一个标记了 [`AppGlideModule`][[2](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 的实现来访问生成的 API。如果你不了解这是怎么工作的，可以查看 [Generated API](https://muyangmin.github.io/glide-docs-cn/doc/generatedapi.html) 。

Generated API添加了一个 `GlideApp` 类，该类提供了对 `RequestBuilder` 和 `RequestOptions` 子类的访问。`RequestOptions` 的子类包含了所有 `RequestOptions` 中的方法，以及 [`GlideExtensions`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/annotation/GlideExtension.html) 中定义的方法。`RequestBuilder` 的子类则提供了生成的 `RequestOptions` 中所有方法的访问，而不需要你再手动调用 `apply` 。举个例子：

在没有使用 Generated API 时，请求大概长这样：

```
Glide.with(fragment)
    .load(url)
    .apply(centerCropTransform()
        .placeholder(R.drawable.placeholder)
        .error(R.drawable.error)
        .priority(Priority.HIGH))
    .into(imageView);

```

使用 Generated API，`RequestOptions` 的调用可以被内联：

```
GlideApp.with(fragment)
    .load(url)
    .centerCrop()
    .placeholder(R.drawable.placeholder)
    .error(R.drawable.error)
    .priority(Priority.HIGH)
    .into(imageView);

```

你仍然可以使用生成的 `RequestOptions` 子类来应用相同的选项到多次加载中；但生成的 `RequestBuilder` 子类可能在多数情况下更为方便。
######9、类型(Type)与目标(Target)
### 选择资源类型[](https://muyangmin.github.io/glide-docs-cn/doc/migrating.html#%E9%80%89%E6%8B%A9%E8%B5%84%E6%BA%90%E7%B1%BB%E5%9E%8B)

Glide 允许你指定你想加载的资源类型。如果你指定了一个超类型，Glide 会尝试加载任何可用的子类型。比如，如果你请求的是 Drawable ，Glide 可能会加载一个 BitmapDrawable 或一个 GifDrawable 。而如果你请求的是一个 GifDrawable ，要么会加载出一个 GifDrawable，要么报错–只要图片不是 GIF 的话（即使它凑巧是一个完全有效的图片也是如此）。

默认请求的类型是 Drawable：

```
Glide.with(fragment).load(url)

```

如果要明确指定请求 Bitmap：

```
Glide.with(fragment).asBitmap()

```

如果要创建一个文件路径（本地图片的最佳选项）：

```
Glide.with(fragment).asFile()

```

如果要下载一个远程文件到缓存然后创建文件路径：

```
Glide.with(fragment).downloadOnly()
// or if you have the url already:
Glide.with(fragment).download(url);
```
######10、Drawables
Glide v3 版本中的 `GlideDrawable` 类已经被移除，支持标准的Android [`Drawable`](https://developer.android.com/reference/android/graphics/drawable/Drawable.html)。 `GlideBitmapDrawable` 也已经被删除，由 [`BitmapDrawable`](https://developer.android.com/reference/android/graphics/drawable/BitmapDrawable.html) 代替之。

如果你想知道某个 Drawable 是否是动画(animated)，可以检查它是否为 [`Animatable`](https://developer.android.com/reference/android/graphics/drawable/Animatable.html) 的实例。

```
boolean isAnimated = drawable instanceof Animatable;
```
######11、Targets
`onResourceReady` 方法的签名做了一些修改。例如，对于 `Drawables`:

```
onResourceReady(GlideDrawable drawable, GlideAnimation<? super GlideDrawable> anim) 

```

现在改为:

```
onResourceReady(Drawable drawable, Transition<? super Drawable> transition);

```

类似地, `onLoadFailed` 的签名也有一些变动：

```
onLoadFailed(Exception e, Drawable errorDrawable)

```

改为：

```
onLoadFailed(Drawable errorDrawable)

```

如果你想要获得更多导致加载失败的错误信息，你可以使用 [`RequestListener`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestListener.html) 。
######12、取消请求
`Glide.clear(Target)` 方法被移动到了 [`RequestManager`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestManager.html) 中:

```
Glide.with(fragment).clear(target)

```

使用 `RequestManager` 清除之前由它启动的加载过程，通常能提高性能，虽然这并不是强制要求的。Glide v4 会为每一个 Activity 和 Fragment 跟踪请求，所以你需要在合适的层级去清除请求。

######13、配置
在 Glide v3 中，配置使用一个或多个 [`GlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/360/com/bumptech/glide/module/GlideModule.html) 来完成。而在 Glide v4 中，配置改为使用一个类似但稍微复杂的系统来完成。

关于这个新系统的细节，可以查看[配置](https://muyangmin.github.io/glide-docs-cn/doc/configuration.html)页面。


######14、应用程序
在早期版本中使用了一个 [`GlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/360/com/bumptech/glide/module/GlideModule.html) 的应用，可以将它转换为一个 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 。

在 Glide v3 中，你可能会有一个像这样的 `GlideModule` ：

```
public class GiphyGlideModule implements GlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    builder.setMemoryCache(new LruResourceCache(10 * 1024 * 1024));
  }

  @Override
  public void registerComponents(Context context, Registry registry) {
    registry.append(Api.GifResult.class, InputStream.class, new GiphyModelLoader.Factory());
  }
}

```

在 Glide v4 中，你需要将其转换成一个 `AppGlideModule` ，它看起来像这样：

```
@GlideModule
public class GiphyGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    builder.setMemoryCache(new LruResourceCache(10 * 1024 * 1024));
  }

  @Override
  public void registerComponents(Context context, Registry registry) {
    registry.append(Api.GifResult.class, InputStream.class, new GiphyModelLoader.Factory());
  }
}

```

请注意，`@GlideModule` 注解不能省略。

如果你的应用拥有多个 `GlideModule`，你需要把其中一个转换成 `AppGlideModule`，剩下的转换成 [`LibraryGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html) 。除非存在`AppGlideModule` ，否则程序不会发现 `LibraryGlideModule` ，因此您不能仅使用 `LibraryGlideModule` 。


######15、程序库
拥有一个或多个 `GlideModule` 的程序库应该使用 [`LibraryGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html) 。程序库不应该使用 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) ，因为它在一个应用里只能有一个。因此，如果你试图在程序库里使用它，将不仅会妨碍这个库的用户设置自己的选项，还会在多个程序库都这么做时造成冲突。

例如，v3 版本中 Volley 集成库的 `GlideModule` ：

```
public class VolleyGlideModule implements GlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    // Do nothing.
  }

  @Override
  public void registerComponents(Context context, Registry registry) {
    registry.replace(GlideUrl.class, InputStream.class, new VolleyUrlLoader.Factory(context));
  }
}

```

在 v4 版本中可以转换成为一个 `LibraryGlideModule` ：

```
@GlideModule
public class VolleyLibraryGlideModule extends LibraryGlideModule {
  @Override
  public void registerComponents(Context context, Registry registry) {
    registry.replace(GlideUrl.class, InputStream.class, new VolleyUrlLoader.Factory(context));
  }
}
```

######16、清单解析

为了简化迁移过程，尽管清单解析和旧的 [`GlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/360/com/bumptech/glide/module/GlideModule.html) 接口已被废弃，但它们在 v4 版本中仍被支持。`AppGlideModule`，`LibraryGlideModule`，与已废弃的 `GlideModule` 可以在一个应用中共存。

然而，为了避免检查元数据的性能天花板（以及相关的 bugs ），你可以在迁移完成后禁用掉清单解析，在你的 `AppGlideModule` 中复写一个方法：

```
@GlideModule
public class GiphyGlideModule extends AppGlideModule {
  @Override
  public boolean isManifestParsingEnabled() {
    return false;
  }

  ...
}
```

######17、using(), ModelLoader, StreamModelLoader.
#### ModelLoader[](https://muyangmin.github.io/glide-docs-cn/doc/migrating.html#modelloader)

[`ModelLoader`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/model/ModelLoader.html) API 在 v4 版本中仍然存在，并且它的设计目标仍然和它在 v3 中一样，但有一些细节变化。

第一个细节，`ModelLoader` 的子类型如 `StreamModelLoader` ，现在已没有存在的必要，用户可以直接实现 `ModelLoader` 。例如，一个`StreamModelLoader<File>` 类现在可以通过 `ModelLoader<File, InputStream>` 的方式来实现和引用。

第二， `ModelLoader` 现在并不直接返回 `DataFetcher`，而是返回 [`LoadData`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/model/ModelLoader.LoadData.html) 。[`LoadData`] 是一个非常简单的封装，包含一个磁盘缓存键和一个 `DataFetcher`。

第三， `ModelLoaders` 有一个 `handles()` 方法，这使你可以为同一个类型参数注册超过一个的 ModelLoader 。

将一个 `ModelLoader` 从 v3 API转换到 v4 API ，通常是很简单直接的。如果你在你的 v3 `ModelLoader` 中只是简单滴返回一个 `DataFetcher` ：

```
public final class MyModelLoader implements StreamModelLoader<File> {

  @Override
  public DataFetcher<InputStream> getResourceFetcher(File model, int width, int height) {
    return new MyDataFetcher(model);
  }
}

```

那么你在 v4 替代类上需要做的仅仅只是封装一下这个 data fetcher ：

```
public final class MyModelLoader implements ModelLoader<File, InputStream> {

  @Override
  public LoadData<InputStream> buildLoadData(File model, int width, int height,
      Options options) {
    return new LoadData<>(model, new MyDataFetcher(model));
  }

  @Override
  public void handles(File model) {
    return true;
  }
}

```

请注意，除了 `DataFetcher` 之外，模型也被传递给 `LoadData` 作为缓存键的一部分。这个规则为某些特殊场景提供了更多对磁盘缓存键的控制。大部分实现可以直接将 model 传入 `LoadData` ，就像上面这样。

如果你仅仅是想为某些 model（而不是所有）使用你的 ModelLoader，你可以在你尝试加载 model 之前使用 `handles()` 方法来检查它。如果你从 `handles` 方法中返回了 `false` ，那么你的 `ModelLoader` 将不能加载指定的 model ，即使你的 `ModelLoader` 类型 (在这个例子里是 `File` 和 `InputStream`) 与之匹配。

举个例子，如果你在某个指定文件夹下写入了加密的图片，你可以使用 `handles` 方法来实现一个 `ModelLoader` 以从那个特定的文件夹下解密图片，但是并不用于加载其他文件夹下的 `File` ：

```
public final class MyModelLoader implements ModelLoader<File, InputStream> {
  private static final String ENCRYPTED_PATH = "/my/encrypted/folder";

  @Override
  public LoadData<InputStream> buildLoadData(File model, int width, int height,
      Options options) {
    return new LoadData<>(model, new MyDataFetcher(model));
  }

  @Override
  public void handles(File model) {
    return model.getAbsolutePath().startsWith(ENCRYPTED_PATH);
  }
}

```

#### `using()`[](https://muyangmin.github.io/glide-docs-cn/doc/migrating.html#using)

`using` API在 Glide v4 中被删除了，这是为了鼓励用户使用 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 一次性地 [注册](https://muyangmin.github.io/glide-docs-cn/doc/generatedapi.html) 所有组件，避免对象重用(re-use, 原文如此 –译者注)。你无需每次加载图片时都创建一个新的 `ModelLoader` ；你应该在 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 中注册一次，然后交给 Glide 在每次加载时检查 model (即你传入 [`load()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html#load-java.lang.Object-) 方法的对象)来决定什么时候使用你注册的 ``ModelLoader` 。

为了确保你仅为特定的 model 使用你的 `ModelLoader` ，请像上面展示的那样实现 `handles` 方法：检查每个 model ，但仅在应当使用你的 `ModelLoader` 时才返回 true 。

##以上17点就是Glide4.X和Glide3.X的区别，还是建议看官方文档
