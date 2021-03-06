# PhotoFactory

[![](https://jitpack.io/v/AnliaLee/PhotoFactory.svg)](https://jitpack.io/#AnliaLee/PhotoFactory)
[![GitHub release](https://img.shields.io/github/release/AnliaLee/PhotoFactory.svg)](https://github.com/AnliaLee/PhotoFactory/releases)
[![GitHub license](https://img.shields.io/github/license/AnliaLee/PhotoFactory.svg)](https://github.com/AnliaLee/PhotoFactory/blob/master/LICENSE)

**PhotoFactory**封装了调用相机拍照，从相册选取照片，压缩选取的照片，图片搜索等功能

### 添加依赖
**Gradle** 

```
repositories {
	...
	maven { url 'https://jitpack.io' }
}

dependencies {
	compile 'com.github.AnliaLee:PhotoFactory:1.1.8'
}

```

### 配置权限

```xml
<uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```
***
### 如何使用

**PhotoFactory**兼容了**Android 7.0 FileProvider**获取相片**uri**的问题，相关的配置在 **1.1.2** 版本以上已为大家配好，在项目中使用时只需考虑**Android 6.0的动态权限管理**即可

#### 初始化及相关API

```java
//两种初始化方法
PhotoFactory photoFactory = new PhotoFactory(context);//自动配置临时图片的路径
PhotoFactory photoFactory = new PhotoFactory(context, photoDir, photoName)
```
设置选择照片的方式

```java
//从相册中选图
photoFactory.FromGallery()
            .StartForResult(new PhotoFactory.OnResultListener() {
                @Override
                public void onCancel() {
                    //取消选择的回调
                }

                @Override
                public void onSuccess(ResultData resultData) {
                    //选择成功的回调
                }

                @Override
                public void onError(String error) {
                    //操作过程中发生异常的回调
                }
            });
            
//从拍照中选图
photoFactory.FromCamera() 

//裁剪图片
photoFactory.FromCrop(uri)
```

对返回的数据的处理

```java
resultData.GetBitmap()
resultData.GetUri()
```

当然，你还可以对图片进行**压缩**处理

```java
resultData.addScaleCompress(w,h).GetUri()//按目标宽高缩放
resultData.addQualityCompress(targetSize).GetUri()//质量压缩，targetSize为目标大小（低端机不建议使用，暂未优化内存）
```

以及抓取处理结果时的**异常**

```java
Uri uri = resultData
        .setExceptionListener(new ResultData.OnExceptionListener() {
            @Override
            public void onCatch(String error, Exception e) {
                //抓取异常的回调
            }
        })
        .GetUri();
```

相关异常解析（适用于 OnResultListener.onError 或 OnExceptionListener.onCatch 回调）

```java
"ERROR_CROP_DATA": 获取裁剪图片uri时出现异常，通常发生在手机存储空间不足、存储空间被占用或oom的情况
"ERROR_RESULT_DATA": 从ActivityResult的intent中获取数据出现异常
"ERROR_CAMERA_NOT_FOUND": 寻找照相设备异常，通常出现在一些没有相机的android设备上
"ERROR_MEDIA_INSERT_IMAGE": 插入图片异常，通常发生在手机存储空间不足或存储空间被占用的情况
"ERROR_MEDIA_GET_BITMAP": 获取bitmap异常，通常发生在通过uri查找不到对应照片的情况
"ERROR_COMPRESS": 压缩图片异常，通常发生在某些配置较低的机型内存不足的情况
```


具体使用流程可参照demo

***
#### 搜索图片

**PhotoFactory**封装了**搜索图片**的功能，你可以根据**图片的路径、名称、图片格式等条件**搜索图片，执行搜索后返回符合条件图片的**list**。这里以查询所有gif图片为例

```java
//加载数据的映射（MediaStore.Images.Media.DATA等）
String[] projection = new String[]{
	MediaStore.Images.Media.DATA,//图片路径
	MediaStore.Images.Media.DISPLAY_NAME,//图片文件名，包括后缀名
	MediaStore.Images.Media.TITLE//图片文件名，不包含后缀
};

photoFactory = new PhotoFactory(this);
photoFactory.FromSearch(getSupportLoaderManager(),getApplicationContext(),projection)
			.setSelectionByFormat(new String[]{".gif"})//设置查询条件（通过图片格式查找，非必选）
			//.setSelection(new String[]{"图片收藏","weixin"}) （或模糊匹配搜索指定图片，非必选）
			.setLoadingEvent(new InterfaceManager.LoadingCallBack() {//设置异步加载时loading操作（非必选）
				@Override
				public void showLoading() {
					myProgressDialog.show();
				}

				@Override
				public void hideLoading() {
					if(myProgressDialog.isShowing()){
						myProgressDialog.dismiss();
					}
				}
			})
			.setErrorEvent(new InterfaceManager.ErrorCallBack() {//设置搜索出错时的操作（非必选）
				@Override
				public void dealError(String s) {
					Toast.makeText(SearchGifActivity.this, s, Toast.LENGTH_SHORT).show();
				}
			})
			.execute(new InterfaceManager.SearchDataCallBack() {//执行搜索并获取回调数据
				@Override
				public void onFinish(final List<Map<String, Object>> list) {
					searchGifAdapter = new SearchGifAdapter(SearchGifActivity.this,list);
					recyclerView.setAdapter(searchGifAdapter);
					
					//通过你之前设置的映射获取数据，例如：list.get(position).get(MediaStore.Images.Media.DATA)
				}
			});
```
