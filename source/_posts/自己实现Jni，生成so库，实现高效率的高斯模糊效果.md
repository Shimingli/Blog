---
title: 自己实现Jni，生成so库，实现高效率的高斯模糊效果
date: 2017-09-26 16:36:01
tags: [Jni,,So库]
---
## 看效果

![微信图片_20170927142832.jpg](http://upload-images.jianshu.io/upload_images/5363507-ac94f6a250b7ab80.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_20170927142827.jpg](http://upload-images.jianshu.io/upload_images/5363507-c1a17ae6f1d16d03.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_20170927142824.jpg](http://upload-images.jianshu.io/upload_images/5363507-297cec9fdf894877.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_20170927142820.jpg](http://upload-images.jianshu.io/upload_images/5363507-e81127e1886efb12.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!--  more  -->

##为什么要做，因为在实现模糊图上，当radios过大的话不同手机设备上可能会导致OutOfMemoryError,高斯模糊在安卓上实现的算法，一般的手机还不能够完成，所以在想能不能把实现模糊图的过程让jni来玩成，通过自己找些开源的算法，然后生成自己需要的so库，这就是写这个Demo的原因
####个人封装的Glide框架，在实现很模糊的效果上，在锤子手机渲染不出来，因为计算的次数太多了，这是代码的地址：http://www.jianshu.com/p/9080483bac91
```
//本地图片，模糊处理
        ImageLoader.getInstance().displayImageInResource(this, R.mipmap.test, mImageView_5, new BlurBitmapTransformation(this, 200));

 /**
     * 当设置过大的radius 容易内存溢出
     * at iamgeloader.client.tranform.BlurBitmapTransformation.blur(BlurBitmapTransformation.java:80)
     * @param source
     * @param radius
     * @return
     */
    private Bitmap blur(Bitmap source, int radius) {
        Bitmap bitmap = source.copy(source.getConfig(), true);
        int w = source.getWidth();
        int h = source.getHeight();
        int[] pix = new int[w * h];
//        像素颜色写入位图
//        要跳过的像素[ [] ]中的条目数
//*行（必须是=位图的宽度）。
//        从每行读取的像素数
//        要读取的行数。
        bitmap.getPixels(pix, 0, w, 0, 0, w, h);

        int wm = w - 1;
        int hm = h - 1;
        int wh = w * h;
        int div = radius + radius + 1;

        int r[] = new int[wh];
        int g[] = new int[wh];
        int b[] = new int[wh];
        int rsum, gsum, bsum, x, y, i, p, yp, yi, yw;
        int vmin[] = new int[Math.max(w, h)];

        int divsum = (div + 1) >> 1;
        divsum *= divsum;
        int dv[] = new int[256 * divsum];
        for (i = 0; i < 256 * divsum; i++) {
            dv[i] = (i / divsum);
        }

        yw = yi = 0;

        int[][] stack = new int[div][3];
        int stackPointer;
        int stackStart;
        int[] sir;
        int rbs;
        int r1 = radius + 1;
        int routSum, goutSum, boutSum;
        int rinSum, ginSum, binSum;

        for (y = 0; y < h; y++) {
            rinSum = ginSum = binSum = routSum = goutSum = boutSum = rsum = gsum = bsum = 0;
            for (i = -radius; i <= radius; i++) {
                p = pix[yi + Math.min(wm, Math.max(i, 0))];
                sir = stack[i + radius];
//    12&5 的值是多少？答：12转成二进制数是1100（前四位省略了），
// 5转成二进制数是0101，则运算后的结果为0100即4  这是两侧为数值时；
                sir[0] = (p & 0xff0000) >> 16;
                sir[1] = (p & 0x00ff00) >> 8;
                sir[2] = (p & 0x0000ff);
                rbs = r1 - Math.abs(i);
                rsum += sir[0] * rbs;
                gsum += sir[1] * rbs;
                bsum += sir[2] * rbs;
                if (i > 0) {
                    rinSum += sir[0];
                    ginSum += sir[1];
                    binSum += sir[2];
                } else {
                    routSum += sir[0];
                    goutSum += sir[1];
                    boutSum += sir[2];
                }
            }
            stackPointer = radius;

            for (x = 0; x < w; x++) {

                r[yi] = dv[rsum];
                g[yi] = dv[gsum];
                b[yi] = dv[bsum];

                rsum -= routSum;
                gsum -= goutSum;
                bsum -= boutSum;

                stackStart = stackPointer - radius + div;
                sir = stack[stackStart % div];

                routSum -= sir[0];
                goutSum -= sir[1];
                boutSum -= sir[2];

                if (y == 0) {
                    vmin[x] = Math.min(x + radius + 1, wm);
                }
                p = pix[yw + vmin[x]];

                sir[0] = (p & 0xff0000) >> 16;
                sir[1] = (p & 0x00ff00) >> 8;
                sir[2] = (p & 0x0000ff);

                rinSum += sir[0];
                ginSum += sir[1];
                binSum += sir[2];

                rsum += rinSum;
                gsum += ginSum;
                bsum += binSum;

                stackPointer = (stackPointer + 1) % div;
                sir = stack[(stackPointer) % div];

                routSum += sir[0];
                goutSum += sir[1];
                boutSum += sir[2];

                rinSum -= sir[0];
                ginSum -= sir[1];
                binSum -= sir[2];

                yi++;
            }
            yw += w;
        }
        for (x = 0; x < w; x++) {
            rinSum = ginSum = binSum = routSum = goutSum = boutSum = rsum = gsum = bsum = 0;
            yp = -radius * w;
            for (i = -radius; i <= radius; i++) {
                yi = Math.max(0, yp) + x;

                sir = stack[i + radius];

                sir[0] = r[yi];
                sir[1] = g[yi];
                sir[2] = b[yi];

                rbs = r1 - Math.abs(i);

                rsum += r[yi] * rbs;
                gsum += g[yi] * rbs;
                bsum += b[yi] * rbs;

                if (i > 0) {
                    rinSum += sir[0];
                    ginSum += sir[1];
                    binSum += sir[2];
                } else {
                    routSum += sir[0];
                    goutSum += sir[1];
                    boutSum += sir[2];
                }

                if (i < hm) {
                    yp += w;
                }
            }
            yi = x;
            stackPointer = radius;
            for (y = 0; y < h; y++) {
                // Preserve alpha channel: ( 0xff000000 & pix[yi] )
                //在该位置点出现一个萌白的点
                pix[yi] = (0xff000000 & pix[yi]) | (dv[rsum] << 16) | (dv[gsum] << 8) | dv[bsum];

                rsum -= routSum;
                gsum -= goutSum;
                bsum -= boutSum;

                stackStart = stackPointer - radius + div;
                sir = stack[stackStart % div];

                routSum -= sir[0];
                goutSum -= sir[1];
                boutSum -= sir[2];

                if (x == 0) {
                    vmin[y] = Math.min(y + r1, hm) * w;
                }
                p = x + vmin[y];

                sir[0] = r[p];
                sir[1] = g[p];
                sir[2] = b[p];

                rinSum += sir[0];
                ginSum += sir[1];
                binSum += sir[2];

                rsum += rinSum;
                gsum += ginSum;
                bsum += binSum;

                stackPointer = (stackPointer + 1) % div;
                sir = stack[stackPointer];

                routSum += sir[0];
                goutSum += sir[1];
                boutSum += sir[2];

                rinSum -= sir[0];
                ginSum -= sir[1];
                binSum -= sir[2];

                yi += w;
            }
        }

        bitmap.setPixels(pix, 0, w, 0, 0, w, h);

        return bitmap;
    }
```
####创建.c文件，创建jni文件夹，这里我们是没有so库的，所以我们要通过androidStudio生成so库

![image.png](http://upload-images.jianshu.io/upload_images/5363507-74e5d05d6ef22bc9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
####配置ndk正确的方法
![image.png](http://upload-images.jianshu.io/upload_images/5363507-67165df6eb690eec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里是正确的配置地址

![image.png](http://upload-images.jianshu.io/upload_images/5363507-8aba48543758ea36.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####Android.mk
```
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)
#编译生成的文件的类库叫什么名字
LOCAL_MODULE    := ShimingImageBlur
#要编译的c文件
LOCAL_SRC_FILES := com_shiming_imageloader_jnitest_JniUtils.cpp

include $(BUILD_SHARED_LIBRARY)

```
####Application.mk
```
#jni打包的C语言类库默认仅支持 arm架构，需要在jni目录下创建 Android.mk 文件添加如下代码可以支持x86架构
#或者是 ：=all
APP_ABI :=armeabi armeabi-v7a x86
```
####com_shiming_imageloader_jnitest_JniUtils.cpp 这里必须要注意一个问题JniUtils使我们的java类全地址，一定要加上Java_在前面，要不然生成的so库,会找不到方法，直接报错
```
#include "com_shiming_imageloader_jnitest_JniUtils.h"
#include "ShimingImageBlur.c"
#include <android/log.h>

//用c++实现的，方法名必须为本地方法的全类名改为下划线
//第一个参数为java虚拟机的内存地址的二级指正，用于本地方法与java虚拟机在内存中的交互
//第二个参数为一个java对象，即是那个对象调用了这个c的方法 ，
//后面的参数就是我们java的方法参数
JNIEXPORT void JNICALL Java_com_shiming_imageloader_jnitest_JniUtils_blurIntArray
(JNIEnv *env, jclass obj, jintArray arrIn, jint w, jint h, jint r)
{
	jint *pix;
	pix = env->GetIntArrayElements(arrIn, 0);
	if (pix == NULL)
		return;
	//Start
	pix = StackBlur(pix, w, h, r);
	//End
	//int size = w * h;
	//jintArray result = env->NewIntArray(size);
	//env->SetIntArrayRegion(result, 0, size, pix);
	env->ReleaseIntArrayElements(arrIn, pix, 0);
	//return result;
}

```

####com_shiming_imageloader_jnitest_JniUtils.h 在这里也是
```
/* DO NOT EDIT THIS FILE - it is machine generated */
#include <jni.h>
/* Header for class com_shiming_imageloader_jni_JniUtils */

#ifndef _Included_com_shiming_imageloader_jnitest_JniUtils
#define _Included_com_shiming_imageloader_jnitest_JniUtils
#ifdef __cplusplus
extern "C" {
#endif
/*
 * Class:     Java_com_shiming_imageloader_jnitest_JniUtils
 * Method:    blurIntArray
 * Signature: ([IIII)V
 * 可千万要加上 Java_的前缀 ，要不然找不到啊
 */

JNIEXPORT void JNICALL Java_com_shiming_imageloader_jnitest_JniUtils_blurIntArray
  (JNIEnv *, jclass, jintArray, jint, jint, jint);

#ifdef __cplusplus
}
#endif
#endif

```
####ShimingImageBlur.c 这里面的算法，是我参考了git上一个开源的项目的算法在git上搜索：ImageBlur即可，但是我用不到那么的方法，所以只需要这个，实现高斯模糊的，其实这个算法和上面的java实现的模糊效果有相同之处，但是我还不能说的太明白，见谅

```
#include<malloc.h>

#define ABS(a) ((a)<(0)?(-a):(a))
#define MAX(a,b) ((a)>(b)?(a):(b))
#define MIN(a,b) ((a)<(b)?(a):(b))


static int* StackBlur(int* pix, int w, int h, int radius) {
	int wm = w - 1;
	int hm = h - 1;
	int wh = w * h;
	int div = radius + radius + 1;
    // 指针
	int *r = (int *)malloc(wh * sizeof(int));
	int *g = (int *)malloc(wh * sizeof(int));
	int *b = (int *)malloc(wh * sizeof(int));
	int rsum, gsum, bsum, x, y, i, p, yp, yi, yw;

	int *vmin = (int *)malloc(MAX(w,h) * sizeof(int));

	int divsum = (div + 1) >> 1;
	divsum *= divsum;
	int *dv = (int *)malloc(256 * divsum * sizeof(int));
	for (i = 0; i < 256 * divsum; i++) {
		dv[i] = (i / divsum);
	}

	yw = yi = 0;

	int(*stack)[3] = (int(*)[3])malloc(div * 3 * sizeof(int));
	int stackpointer;
	int stackstart;
	int *sir;
	int rbs;
	int r1 = radius + 1;
	int routsum, goutsum, boutsum;
	int rinsum, ginsum, binsum;

	for (y = 0; y < h; y++) {
		rinsum = ginsum = binsum = routsum = goutsum = boutsum = rsum = gsum = bsum = 0;
		for (i = -radius; i <= radius; i++) {
			p = pix[yi + (MIN(wm, MAX(i, 0)))];
			sir = stack[i + radius];
			sir[0] = (p & 0xff0000) >> 16;
			sir[1] = (p & 0x00ff00) >> 8;
			sir[2] = (p & 0x0000ff);

			rbs = r1 - ABS(i);
			rsum += sir[0] * rbs;
			gsum += sir[1] * rbs;
			bsum += sir[2] * rbs;
			if (i > 0) {
				rinsum += sir[0];
				ginsum += sir[1];
				binsum += sir[2];
			}
			else {
				routsum += sir[0];
				goutsum += sir[1];
				boutsum += sir[2];
			}
		}
		stackpointer = radius;

		for (x = 0; x < w; x++) {

			r[yi] = dv[rsum];
			g[yi] = dv[gsum];
			b[yi] = dv[bsum];

			rsum -= routsum;
			gsum -= goutsum;
			bsum -= boutsum;

			stackstart = stackpointer - radius + div;
			sir = stack[stackstart % div];

			routsum -= sir[0];
			goutsum -= sir[1];
			boutsum -= sir[2];

			if (y == 0) {
				vmin[x] = MIN(x + radius + 1, wm);
			}
			p = pix[yw + vmin[x]];

			sir[0] = (p & 0xff0000) >> 16;
			sir[1] = (p & 0x00ff00) >> 8;
			sir[2] = (p & 0x0000ff);

			rinsum += sir[0];
			ginsum += sir[1];
			binsum += sir[2];

			rsum += rinsum;
			gsum += ginsum;
			bsum += binsum;

			stackpointer = (stackpointer + 1) % div;
			sir = stack[(stackpointer) % div];

			routsum += sir[0];
			goutsum += sir[1];
			boutsum += sir[2];

			rinsum -= sir[0];
			ginsum -= sir[1];
			binsum -= sir[2];

			yi++;
		}
		yw += w;
	}
	for (x = 0; x < w; x++) {
		rinsum = ginsum = binsum = routsum = goutsum = boutsum = rsum = gsum = bsum = 0;
		yp = -radius * w;
		for (i = -radius; i <= radius; i++) {
			yi = MAX(0, yp) + x;

			sir = stack[i + radius];

			sir[0] = r[yi];
			sir[1] = g[yi];
			sir[2] = b[yi];

			rbs = r1 - ABS(i);

			rsum += r[yi] * rbs;
			gsum += g[yi] * rbs;
			bsum += b[yi] * rbs;

			if (i > 0) {
				rinsum += sir[0];
				ginsum += sir[1];
				binsum += sir[2];
			}
			else {
				routsum += sir[0];
				goutsum += sir[1];
				boutsum += sir[2];
			}

			if (i < hm) {
				yp += w;
			}
		}
		yi = x;
		stackpointer = radius;
		for (y = 0; y < h; y++) {
			// Preserve alpha channel: ( 0xff000000 & pix[yi] )
			pix[yi] = (0xff000000 & pix[yi]) | (dv[rsum] << 16) | (dv[gsum] << 8) | dv[bsum];

			rsum -= routsum;
			gsum -= goutsum;
			bsum -= boutsum;

			stackstart = stackpointer - radius + div;
			sir = stack[stackstart % div];

			routsum -= sir[0];
			goutsum -= sir[1];
			boutsum -= sir[2];

			if (x == 0) {
				vmin[y] = MIN(y + r1, hm) * w;
			}
			p = x + vmin[y];

			sir[0] = r[p];
			sir[1] = g[p];
			sir[2] = b[p];

			rinsum += sir[0];
			ginsum += sir[1];
			binsum += sir[2];

			rsum += rinsum;
			gsum += ginsum;
			bsum += binsum;

			stackpointer = (stackpointer + 1) % div;
			sir = stack[stackpointer];

			routsum += sir[0];
			goutsum += sir[1];
			boutsum += sir[2];

			rinsum -= sir[0];
			ginsum -= sir[1];
			binsum -= sir[2];

			yi += w;
		}
	}
    //记得要释放掉
	free(r);
	free(g);
	free(b);
	free(vmin);
	free(dv);
	free(stack);
	return(pix);
}

```
还有个算法和这个有异曲同工之妙，作用是把bitmap扫描，把Color.TRANSPARENT排除掉生成了一张位于正中的bitmap
```
 private int mBackColor = Color.TRANSPARENT;

    /**
     * 逐行扫描 清楚边界空白。
     *
     * @param blank 边距留多少个像素
     * @return tks github E-signature
     */

    public Bitmap clearBlank(int blank) {
        if (mBitmap != null) {
            int HEIGHT = mBitmap.getHeight();
            int WIDTH = mBitmap.getWidth();
            int top = 0, left = 0, right = 0, bottom = 0;
            int[] pixs = new int[WIDTH];
            boolean isStop;
            for (int y = 0; y < HEIGHT; y++) {
                mBitmap.getPixels(pixs, 0, WIDTH, 0, y, WIDTH, 1);
                isStop = false;
                for (int pix : pixs) {
                    if (pix != mBackColor) {

                        top = y;
                        isStop = true;
                        break;
                    }
                }
                if (isStop) {
                    break;
                }
            }
            for (int y = HEIGHT - 1; y >= 0; y--) {
                mBitmap.getPixels(pixs, 0, WIDTH, 0, y, WIDTH, 1);
                isStop = false;
                for (int pix : pixs) {
                    if (pix != mBackColor) {
                        bottom = y;
                        isStop = true;
                        break;
                    }
                }
                if (isStop) {
                    break;
                }
            }
            pixs = new int[HEIGHT];
            for (int x = 0; x < WIDTH; x++) {
                mBitmap.getPixels(pixs, 0, 1, x, 0, 1, HEIGHT);
                isStop = false;
                for (int pix : pixs) {
                    if (pix != mBackColor) {
                        left = x;
                        isStop = true;
                        break;
                    }
                }
                if (isStop) {
                    break;
                }
            }
            for (int x = WIDTH - 1; x > 0; x--) {
                mBitmap.getPixels(pixs, 0, 1, x, 0, 1, HEIGHT);
                isStop = false;
                for (int pix : pixs) {
                    if (pix != mBackColor) {
                        right = x;
                        isStop = true;
                        break;
                    }
                }
                if (isStop) {
                    break;
                }
            }
            if (blank < 0) {
                blank = 0;
            }
            left = left - blank > 0 ? left - blank : 0;
            top = top - blank > 0 ? top - blank : 0;
            right = right + blank > WIDTH - 1 ? WIDTH - 1 : right + blank;
            bottom = bottom + blank > HEIGHT - 1 ? HEIGHT - 1 : bottom + blank;
            return Bitmap.createBitmap(mBitmap, left, top, right - left, bottom - top);
        } else {
            return null;
        }
    }
```


![image.png](http://upload-images.jianshu.io/upload_images/5363507-fd1f56e0d351912f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####JniUtils类,加载本地方法，本地方法的方法名要和jni中一样,引用so库
```
/**
     * 参照：MediaPlayer的写法
      static {
      System.loadLibrary("media_jni");
      native_init();
       }
     */
    static {
        /**
         * #编译生成的文件的类库叫什么名字
         LOCAL_MODULE    := ShimingImageBlur
         必须和Android.mk中的一样，生成的so库虽然会加上libShimingImageBlur.so 但是调用得到
         */
        System.loadLibrary("ShimingImageBlur");

    }

```
####设置图片的view模糊 ，使用getViewTreeObserver(）监听在绘画完成前，获取缓存的bitmap
```
 // Register a callback to be invoked when the view tree is about to be drawn
        //翻译注册一个回调，view将要drawn，意思还没有drawn上去
        view.getViewTreeObserver().addOnPreDrawListener(new ViewTreeObserver.OnPreDrawListener() {
            /**
             * 再绘图前加上
             * @return
             */
            @Override
            public boolean onPreDraw() {
                view.getViewTreeObserver().removeOnPreDrawListener(this);
                //buildDrawingCache(false);
                //如果绘制无效，则强制构建绘图缓存，
                view.buildDrawingCache();

                Bitmap bmp = view.getDrawingCache();
                blur(bmp, view,flag);
                //通常cache会占用一定量的内存，所以必须销毁
                view.destroyDrawingCache();
                //返回 true 继续绘制，返回false取消。 如果返回为false的，页面就不会绘制了
                return true;
            }
```
####下一步进行的方法吗，在这里有个问题，      ((ImageView)view).setImageDrawable(blurDrawable); 这里有个问题，当我们调用次数过多的时候，这个方法显示不出来图片， 我还不知道为什么？todo 源于知乎上的一句话：简单地理解为 Bitmap 储存的是 像素信息，Drawable 储存的是 对 Canvas 的一系列操作。而 BitmapDrawable 储存的是「把 Bitmap 渲染到 Canvas 上」这个操作，
```
  /**
     * 模糊图片
     *  @param blurFactor 模糊因子
     * @param blurRadius 模糊半径
     * @param bitmap     图片
     * @param view       view
     * @param flag 是否需要jni里面的算法 
     */
    public static void blur(float blurFactor, float blurRadius, Bitmap bitmap, final View view, boolean flag) {

        Bitmap overlay = Bitmap.createBitmap((int) (view.getMeasuredWidth() / blurFactor),
                (int) (view.getMeasuredHeight() / blurFactor), Bitmap.Config.ARGB_8888);
         overlay.getHeight();
        overlay.getWidth();
        Canvas canvas = new Canvas(overlay);
        //如果我们需要放大1倍，即 scale(2, 2);缩放的中心点默认也是canvas的左上角，所以先要进行坐标平移，才能去缩放
        //平移，将画布的坐标原点向左右方向移动x，向上下方向移动y.canvas的默认位置是在（0,0）
        canvas.translate(-view.getLeft() / blurFactor, -view.getTop() / blurFactor);
        canvas.scale(1 / blurFactor, 1 / blurFactor);
        Paint paint = new Paint();
        //抗锯齿
        paint.setFlags(Paint.FILTER_BITMAP_FLAG);
        canvas.drawBitmap(bitmap, 0, 0, paint);
        //这里的overlay已经包含了信息
        //是否需要jni的计算
        if (flag) {
            overlay = doBlurJniArray(overlay, (int) blurRadius, true);
        }
        BitmapDrawable blurDrawable = new BitmapDrawable(MyApp.getInstance().getApplicationContext()
                .getResources(), overlay);
        if (view instanceof ImageView) {
            ((ImageView)view).setImageDrawable(blurDrawable);
            System.out.println("shiming  blurDrawable" +blurDrawable);
        } else {
            view.setBackgroundDrawable(blurDrawable);
        }
    }

```

####核心的方法对pix进行操作，JniUtils.blurIntArray(pix, w, h, radius); 在通过   bitmap.setPixels(pix, 0, w, 0, 0, w, h); 和java实现所用的方法一样，但是性能上提升的不是一点两点，很流畅,而且在显示的效果上有很明显的不一样，如上图
```
   /**
     *
     * @param sentBitmap
     * @param radius
     * @param canReuseInBitmap
     * @return
     *
    参数 :http://ranlic.iteye.com/blog/1313735
    pixels      接收位图颜色值的数组
    offset      写入到pixels[]中的第一个像素索引值
    stride      pixels[]中的行间距个数值(必须大于等于位图宽度)。可以为负数
    x          　从位图中读取的第一个像素的x坐标值。
    y           从位图中读取的第一个像素的y坐标值
    width    　　从每一行中读取的像素宽度
    height 　　　读取的行数
            异常
    IilegalArgumentExcepiton       如果x，y，width，height越界或stride的绝对值小于位图宽度时将被抛出。
    ArrayIndexOutOfBoundsException          如果像素数组太小而无法接收指定书目的像素值时将被抛出。
    */
    private static Bitmap doBlurJniArray(Bitmap sentBitmap, int radius, boolean canReuseInBitmap) {
        Bitmap bitmap;
        if (canReuseInBitmap) {
            bitmap = sentBitmap;
        } else {
            bitmap = sentBitmap.copy(sentBitmap.getConfig(), true);
        }

        if (radius < 1) {
            return (null);
        }
        int w = bitmap.getWidth();
        int h = bitmap.getHeight();
        int[] pix = new int[w * h];
        bitmap.getPixels(pix, 0, w, 0, 0, w, h);
        //pix数组，所有的关键的逻辑都是这个pix的操作，这里我们去交给了so.库去处理了，所以这里才是关键
        JniUtils.blurIntArray(pix, w, h, radius);
        bitmap.setPixels(pix, 0, w, 0, 0, w, h);
        return (bitmap);
    }
```
####如何生成so库,在build.gradle

![image.png](http://upload-images.jianshu.io/upload_images/5363507-5dde711d043f48ec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
        //ndk编译生成.so文件
        // 只要是第一次生成了so库的文件，那么以后就不用生成这个文件，记住一定要记住，如果需要生成so文件
        //需要把外面的目录的jni移动到和mian目录下和Java同级的目录下
//        ndk {
//            moduleName "ShimingImageBlur"         //生成的so名字
//            abiFilters "armeabi", "armeabi-v7a", "x86"  //输出指定三种abi体系结构下的so库。
//        }
```

![image.png](http://upload-images.jianshu.io/upload_images/5363507-6e0e9527b313631a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
####这里才是so库生成正确的目录地址，一定是生成完成了，然后复制出来放在libs目录下

![image.png](http://upload-images.jianshu.io/upload_images/5363507-a524e10e9812c107.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](http://upload-images.jianshu.io/upload_images/5363507-45513de0f48af443.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##这个就生成了自己需要的so文件了，然后就可以使用了，哈哈，建议可以实际操作下，然后学点c，就可以实现jni了，哈哈，我吹牛了！

##git：https://github.com/Shimingli/ImageLoader