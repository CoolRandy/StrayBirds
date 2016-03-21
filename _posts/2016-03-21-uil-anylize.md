---
layout: post
title: Universal-Image-Loader源码解析
category: 技术
comments: false
---

# 使用介绍

## 开源库特征

1. 多线程下载图片，图片来源：网络、文件系统或项目文件夹
2. 可配置度高：支持任务线程池、下载器、解码器、内存及磁盘缓存等
3. 采用内存和磁盘缓存的两级缓存
4. 支持多线程以及异步、同步加载
5. 支持多种缓存算法实现
6. 可以根据控件（ImageView）大小对Bitmap对象进行裁剪，减少Bitmap对象内存占用
7. 较好的控制图片的加载，一般应用于Listview的滑动加载时机的处理
8. 提供网络类型判断，支持较慢网络下加载

## 基本使用

1. 初始化
    1.1 添加完依赖后在Application或Activity中初始化ImageLoader

    ```java
    public class YourApplication extends Application {

        @Override
        public void onCreate() {
            super.onCreate();
            //创建默认的ImageLoader配置参数  
            ImageLoaderConfiguration configuration = ImageLoaderConfiguration  
                .createDefault(this); 
            //Initialize ImageLoader with configuration.  
            ImageLoader.getInstance().init(configuration);     
        }
    }
    ```
    configuration表示ImageLoader的配置信息，这里使用了建造者模式，创建了一个默认的配置，此外我们可以手动配置，可以按照如下代码设置：
    
    ```java
    //个性化配置，建造者模式
     File cacheDir = StorageUtils.getCacheDirectory(context); 
     ImageLoaderConfiguration config = new ImageLoaderConfiguration.Builder(context) 
     .memoryCacheExtraOptions(480, 800) // 默认屏幕大小
     .diskCacheExtraOptions(480, 800, CompressFormat.JPEG, 75, null) 
     .taskExecutor(…) //线程池
     .taskExecutorForCachedImages(…) 
     .threadPoolSize(3) //
     .threadPriority(Thread.NORM_PRIORITY - 1) // 线程优先级
     .tasksProcessingOrder(QueueProcessingType.FIFO) //处理顺序，先进先出等 
     .denyCacheImageMultipleSizesInMemory() 
     .memoryCache(new LruMemoryCache(2  1024  1024)) //内存缓存 
     .memoryCacheSize(2  1024  1024) 
     .memoryCacheSizePercentage(13) // default 
     .diskCache(new UnlimitedDiscCache(cacheDir)) // 磁盘缓存，自定义地址
     .diskCacheSize(50  1024  1024) 
     .diskCacheFileCount(100) 
     .diskCacheFileNameGenerator(new HashCodeFileNameGenerator()) // default 
     .imageDownloader(new BaseImageDownloader(context)) // 下载器 
     .imageDecoder(new BaseImageDecoder()) // 解析器
     .defaultDisplayImageOptions(DisplayImageOptions.createSimple()) 
     .build(); 
     ImageLoader.getInstance().init(configuration);
     ```
     上面的配置选项可以根据我们实际需要进行添加
    1.2 加载配置项
    对于每次要加载的显示项在加载时也可以进行设置
    
    ```java
    DisplayImageOptions options = new DisplayImageOptions.Builder() 
     .showImageOnLoading(R.drawable.ic_stub) //设置加载前默认图片
     .showImageForEmptyUri(R.drawable.ic_empty) 
     .showImageOnFail(R.drawable.ic_error) 
     .resetViewBeforeLoading(false) //加载前会清空图片
     .delayBeforeLoading(1000) 
     .cacheInMemory(true) //开始缓存
     .cacheOnDisk(true) 
     .preProcessor(…) 
     .postProcessor(…) 
     .extraForDownloader(…) 
     .considerExifParams(false) // default 
     .imageScaleType(ImageScaleType.IN_SAMPLE_POWER_OF_2) //类似imageview scaleType
     .bitmapConfig(Bitmap.Config.ARGB_8888) // 在要求不高时可以用555缩小内存占用 
     .decodingOptions(…) 
     .displayer(new SimpleBitmapDisplayer()) // 渲染器，可以加一些特殊效果，例如矩形圆角等 
     .handler(new Handler()) // default 
     .build();
     ```
    1.3 Manifest配置
    添加网络权限，提供读写磁盘权限
    1.4 下载显示图片
    两种方式：一种直接展示在显示控件中，一种下载图片进行回调
    
    ```java
    //直接展示
    imageLoader.displayImage(imageUri, imageView);
    //下载回调
    imageLoader.loadImage(imageUri, new SimpleImageLoadingListener() {
        @Override
        public void onLoadingComplete(String imageUri, View view, Bitmap loadedImage) {
            // 图片处理
        }
    });
    ```
    至此就是UIL的简单使用情况，更为详细的使用可以参见夏安明的博文：[Android开源框架Universal-Image-Loader完全解析（一）—- 基本介绍及使用](http://blog.csdn.net/xiaanming/article/details/26810303)
    接下来将从源码的角度来分析UIL的使用
## 源码分析
UIL整个框架比较复杂，首先来看一下总体设计图![总体设计图](https://raw.githubusercontent.com/android-cn/android-open-project-analysis/master/tool-lib/image-cache/universal-image-loader/image/overall-design.png):
    可以看到整个库分为五大模块，整体的工作流程就是：ImageLoader收到显示或加载图片的任务之后，将它交给ImageLoaderEngine，ImageLoaderEngine负责分发任务到具体的线程池中，任务通过缓存Cache和网络请求ImageDownloader来获取图片，在这期间有可能会对Bitmap对象进行处理以及解码，最终转换为Bitmap交给BitmapDisplayer显示在ImageAware控件中
    下面的流程图更为直观的表示了整个过程：![流程图](https://raw.githubusercontent.com/android-cn/android-open-project-analysis/master/tool-lib/image-cache/universal-image-loader/image/uil-flow.png)
    
### 下面将从几个最为重要的模块进行详细分析
1.图片缓存
![缓存结构](http://images.cnblogs.com/cnblogs_com/CoolRandy/672231/o_cache.PNG)

上图显示了库中缓存这块的结构，disc是磁盘缓存实现，memory是内存缓存实现
UIL采用三层cache机制来缓存图片(这里网络请求算不上cache)，分级缓存的思想就是每次根据url请求图片时，都会首先访问内存缓存，内存中没有时，再从磁盘文件中查找，如果磁盘缓存文件中仍没有，这时再通过网络请求加载
