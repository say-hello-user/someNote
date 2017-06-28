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
