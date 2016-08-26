# ExoPlayer 中 MediaController 的定制

在播放器的组件中，MediaController 主要负责用户在播放中的界面交互，比如进度条的显示与操作、快进、快退、上一个、下一个等。也可以根据需求定制一些自己的操作需求，比如播放器中输入的是直播（或者HLS）的 m3u8 的视频流。那么可能需要添加刷新界面的操作按钮。这一篇主要梳理一下自己在开发 [MDVideo](https://github.com/AndroidTips/MDVideo)过程中遇到的问题，然后以自己所理解的方式阐述一下  MediaController 到底是什么东西。如有错误，还请指正。

## ExoPlayer Demo 中的 MediaController 
在[demo](https://github.com/google/ExoPlayer/tree/release-v1/demo)中，并没有哪个 class 跟 MediaController 名字相关联，因为 demo 中根本没有去写一个 [MediaController](https://developer.android.com/reference/android/widget/MediaController.html) 而是使用系统自带的。那么我们就抓着系统的源码来看看他究竟干了什么，然后再把系统自带的这个干掉，照着样子写一个来实现自己的需求。

我们在[PlayerActivity](https://github.com/google/ExoPlayer/blob/release-v1/demo/src/main/java/com/google/android/exoplayer/demo/PlayerActivity.java)中找到 MediaController 然后查找到调用的方法如下：
```java
  abstract void setAnchorView(View rootView);
  abstract void setMediaPlayer(MediaPlayerControl player);
  abstract void setEnabled(boolean enabled);
  abstract boolean isShowing();
  abstract void hide();
  abstract void show(int timeout);
```
全部写到一个抽象类中，方便下一步定制。接下来再看看 MediaController 中都是怎么实现的。

MediaController 继承自 FrameLayout 。在构造方法中，调用 initFloatingWindow 进行 View 初始化构造：
```
  private void initFloatingWindow() {
        mWindowManager = (WindowManager)mContext.getSystemService(Context.WINDOW_SERVICE);
        mWindow = new PhoneWindow(mContext);
        mWindow.setWindowManager(mWindowManager, null, null);
        mWindow.requestFeature(Window.FEATURE_NO_TITLE);
        mDecor = mWindow.getDecorView();
        mDecor.setOnTouchListener(mTouchListener);
        mWindow.setContentView(this);
        mWindow.setBackgroundDrawableResource(android.R.color.transparent);

        // While the media controller is up, the volume control keys should
        // affect the media stream type
        mWindow.setVolumeControlStream(AudioManager.STREAM_MUSIC);

        setFocusable(true);
        setFocusableInTouchMode(true);
        setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
        requestFocus();
    }
```
在这一步，我们发现。MediaController 首先获取了系统的 WindowManager 对象。然后 new 了一个 PhoneWindow 并将系统的 WindowManager 传给它。然后通过 getDecorView 获取当前界面最底层的 View 然后进行一些add remove等操作来实现播放控制布局的移除与添加。将 MediaController 中的代码复制出来新建一个类，会发现一些错误都好解决，唯独 PhoneWindow 这个对象是无法获取到的。所以起初偷懒打算直接代码 copy 过来，改改layout的想法行不通。（这里是否直接替换了Activity的PhoneWindow我其实没搞清楚，我只知道Activity会在onCreate中去生成window,这里在获取windowManager后已经可以操作悬浮窗了为何还要 new 一个 phoneWindow）

## MediaController 的自定义
重新认识一下 MediaController ，其实就是悬浮的 View 然后响应了播放控制的一些操作。所以我们把之前提取到抽象类的方法实现就可以了。然后在自定义的 MediaController 里构造自己的布局，使用PopupWindow也是可以的。先理一下每个方法的作用：
```
abstract void setAnchorView(View rootView);
```
在 PlayerActivity 中调用，将父布局传入，主要的作用：
- 为了添加事件监听，处理交互操作。
- 确定显示位置（使用 PopupWindow 的话）

```
 abstract void setMediaPlayer(MediaPlayerControl player);
```
播放控制回调，这里我们继续使用系统 MediaController 中的 MediaPlayerControl 代码如下：
```
    public interface MediaPlayerControl {
        void    start();
        void    pause();
        int     getDuration();
        int     getCurrentPosition();
        void    seekTo(int pos);
        boolean isPlaying();
        int     getBufferPercentage();
        boolean canPause();
        boolean canSeekBackward();
        boolean canSeekForward();

        /**
         * Get the audio session id for the player used by this VideoView. This can be used to
         * apply audio effects to the audio track of a video.
         * @return The audio session, or 0 if there was an error.
         */
        int     getAudioSessionId();
    }
```
说实话，这部分代码的调用看着有点别扭。因为 ExoPlayer 中使用的就是系统的 MediaPlayerControl ，可能是因为 demo 中使用 MediaController 缘故。我觉得这个接口应该由 ExoPlayer 自己去定义，这样才不会跟系统的接口搅到一起去，也方便用户扩展。可惜 demo 中没有自定义MediaController,所以导致只能用系统的。话又说回来，这个接口已经能满足需求了，没必要重复造轮子，没准哪天 ExoPlayer 直接整合到系统源代码里了，这个问题也就不存在了。

```
  abstract void setEnabled(boolean enabled);
  abstract boolean isShowing();
  abstract void hide();
  abstract void show(int timeout);
```
这四个方法用来控制悬浮窗的显示与隐藏，判断显示状态，设置是否可操作等。

理清以上关系以后，具体的实现就清晰了，我写了一个本地视频播放控制的 MediaController ,具体可以看[这里](https://github.com/AndroidTips/MDVideo/blob/master/app/src/main/java/com/studyjams/mdvideo/PlayerModule/MediaController/ExtractorMediaController.java)
