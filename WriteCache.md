# APP bar 的使用

最近在自己的项目[MDVideo](https://github.com/AndroidTips/MDVideo)中添加了半透明的 StatusBar 效果，索性把官方文档中关于这一部分的讲解总结一下，
加强一下这一块的记忆。之前做了一年半的 TV 应用开发，由于交互上只处理 OnKey 的事件，所以应用基本都是采用 FullScreen 样式。并且由于当时使用的 Eclipse 
对 support V7 包的支持完全令人无语，导致这一块细节的了解还是比较陌生的。

Google 在 Android 5.0 引入 Material Design，同时很多 ActionBar 的方法被弃用了，通过在 Appcombat V7 包中添加 ToolBar 来替代原有的 ActionBar。由于是引用 Library, 这在版本兼容上避免了很多问题。ToolBar 之前，在 Android 3.0开始，ActionBar包含于 Theme.Holo 主题中，本篇会引用 ActionBar 的一些介绍,以便于了解 ActionBar 在 APP 中的作用，ToolBar 的设计理念与它是一样的，但不赘述怎么使用，因为已经过时了。

## 添加ToolBar

> Action bar 最基本的形式，就是为 Activity 显示标题，并且在标题左边显示一个 app icon。即使在这样简单的形式下，action bar对于所有的 activity 来说是十分有用的。它告知用户他们当前所处的位置，并为你的 app 维护了持续的同一标识。

在xml文件中添加ToolBar:
```java
<android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:fitsSystemWindows="true"
            android:background="?attr/colorPrimary"
            app:popupTheme="@style/AppTheme.PopupOverlay"/>
```
然后在AppCompatActivity里设置：
```java
Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
```
tips：这里使用的style都是Theme.AppCompat主题族。

## 在 ToolBar 中添加item
> Action bar 允许我们为当前环境下最重要的操作添加按钮。那些直接出现在 action bar 中的 icon 和/或文本被称作action buttons(操作按钮)。安排不下的或不足够重要的操作被隐藏在 action overflow 中。

【此处差一张图】
一个有search操作按钮和 action overflow 的 action bar，在 action overflow 里能展现额外的操作。
所有的操作按钮和 action overflow 中其他可用的条目都被定义在 menu资源 的 XML 文件中。通过在项目的 res/menu 目录中新增一个 XML 文件来为 ToolBar 添加操作。

```java
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:app="http://schemas.android.com/apk/res-auto">

    <item
        android:id="@+id/player_scale_4_3"
        android:title="@string/player_scale_4_3"/>
    <item
        android:id="@+id/player_scale_16_9"
        android:title="@string/player_scale_16_9"/>
    <item
        android:id="@+id/player_scale_default"
        android:title="@string/player_scale_default"/>
    <item
        android:id="@+id/player_Rotation"
        android:icon="@drawable/ic_rotate_right"
        android:title="@string/player_Rotation"
        app:showAsAction="always"/>
</menu>
```
tips：这里的 showAsAction 属性决定了该 item 是否在 ToolBar 上单独显示，或是只在 action overflow 中显示。因为使用的ToolBar来自 V7 包，这里需要添加自定义命名空间。

```java
xmlns:app="http://schemas.android.com/apk/res-auto"

app:showAsAction="always"
```
在 Activity 的 onCreateOptionsMenu() 方法回调中 inflate 菜单资源从而获取 Menu 对象：
```java
@Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.main, menu);
        return true;
    }
```
## 添加ToolBar的事件响应
> 当用户按下某一个操作按钮或者 action overflow 中的其他条目，系统将调用 activity 中onOptionsItemSelected()的回调方法。在该方法的实现里面调用MenuItem的getItemId()来判断哪个条目被按下

```java
@Override
    public boolean onOptionsItemSelected(MenuItem item) {

        switch (item.getItemId()){
            case R.id.player_scale_4_3:
                mVideoView.setDisplayAspectRatio(PLVideoTextureView.ASPECT_RATIO_4_3);
                break;
            case R.id.player_scale_16_9:
                mVideoView.setDisplayAspectRatio(PLVideoTextureView.ASPECT_RATIO_16_9);
                break;
            case R.id.player_scale_default:
                mVideoView.setDisplayAspectRatio(PLVideoTextureView.ASPECT_RATIO_PAVED_PARENT);
                break;

            case R.id.player_Rotation:
                mRotation = (mRotation + 90) % 360;
                mVideoView.setDisplayOrientation(mRotation);
                break;

            default:
                mVideoView.setDisplayAspectRatio(PLVideoTextureView.ASPECT_RATIO_PAVED_PARENT);
                break;
        }

        //必须super,否则manifest中设置的actionBar返回无效
        return super.onOptionsItemSelected(item);
    }
```
## 为下级 Activity 添加向上按钮
在不是程序入口的其他所有屏中（activity 不位于主屏时），需要在 action bar 中为用户提供一个导航到逻辑父屏的up button(向上按钮)。
【缺一张图】
在 manifest 文件中声明父 activity，并在activity中设置：
```java
 <activity
            android:name=".PlayerModule.PlayerTextureActivity"
            android:configChanges="orientation|keyboardHidden|screenSize"
            android:parentActivityName="com.artharyoung.mdvideo.MainActivity"
            android:screenOrientation="landscape"
            android:theme="@style/MyTheme.FullScreen">
        </activity>
        
        getSupportActionBar().setDisplayHomeAsUpEnabled(true);
```
tips:在onOptionsItemSelected中不能return true,否则这个返回键会不响应。

## 添加搜索与分享

在Menu资源文件中添加item:
```java
<item android:id="@+id/action_search"
          android:title="@string/menu_search"
          android:icon="@android:drawable/ic_menu_search"
          app:showAsAction="ifRoom|collapseActionView"
          app:actionViewClass="android.support.v7.widget.SearchView" />
```
在 onCreateOptionsMenu 获取SearchView的对象并设置监听事件：
```java
MenuItem searchItem = menu.findItem(R.id.action_search);
SearchView searchView = (SearchView) MenuItemCompat.getActionView(searchItem);

 // Configure the search info and add any event listeners...
```
关于分享的设置，请参考另一个帖子《XXX》
## 沉浸式状态栏的实现
APP的状态栏随主题变化，在Android 4.4开始引进，在4.4之前是不能自定义状态栏颜色的。实现主要有三个要点：

- ToolBar高度设置为wrap_content
- ToolBar添加属性android:fitsSystemWindows="true"
- 父布局添加属性android:fitsSystemWindows="true"
这样可以达到在android 5.0以上系统中状态栏的半透明，如下：
【图片】
为了兼容API 19的效果，需要在values-v19添加属性：
```java
 <!-- API 19 兼容半透明状态栏 -->
    <style name="AppTheme" parent="@style/BaseAppTheme">
        <item name="android:windowTranslucentStatus">true</item>
    </style>
```
这样，在API 19的机器上状态栏将变为全透明。但是比较坑爹的是，国产手机对系统的更改，使得这样设置并不能达到理想的效果。比如我在小米4 MIUI 7.3的系统上，Android 6.0的机器上状态栏居然全透明了。
这时候就不能在style里做兼容了，可以在Activity中设置：
```java
  private void initStatusBar() {
        if (Build.VERSION.SDK_INT == Build.VERSION_CODES.KITKAT) {
            getWindow().addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);
        }
    }
```


## 参考
[《Adding the App Bar》](https://developer.android.com/training/appbar/index.html)
[《添加Action Bar》](http://hukai.me/android-training-course-in-chinese/basics/actionbar/index.html)
