# someNote

### 关于ios不能自动播放视频、音频的解决方案

```javascript
  if ("wView" in window) {
    window.wView.allowsInlineMediaPlayback = "YES";
    window.wView.mediaPlaybackRequiresUserAction = "NO";
  }
```

在head中加入此段代码，ios音视频不能自动播放的问题迎刃而解。
当然，在video标签中，需要先设定autoplay和preload属性，如下：


<video src="xxxxxx" autoplay preload></video>

### 通过直接调用video.play()方法
在一些情况下我们想加入一些判断逻辑，如判断用户网络环境，在wifi下自动播放，在4g环境下给出提示，这中情况下就适合直接选中video并调用video.play来播放视频

但是这种情况下也需要webview的支持，如在手Q下可以做到直接调用，在微信下因为不允许视频直接播放，则必须通过用户的真实操作来触发调用video.play(),这就是各种微信的h5活动页面需要引导用户进行一下点击操作才开始的原因。

同时发现真实点击必须使用触发 touchend、click、doubleclick 或 keydown 事件等标准的事件才能触发，使用Zepto封装过的tap事件并不能触发播放器的播放
