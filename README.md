# H5网页音频遇到的问题

### 1.关于ios不能自动播放视频、音频的解决方案

```javascript
  if ("wView" in window) {
    window.wView.allowsInlineMediaPlayback = "YES";
    window.wView.mediaPlaybackRequiresUserAction = "NO";
  }
```

在head中加入此段代码，ios音视频不能自动播放的问题迎刃而解。
当然，在video标签中，需要先设定autoplay和preload属性，如下：


<video src="xxxxxx" autoplay preload></video>

### 2.通过直接调用video.play()方法
在一些情况下我们想加入一些判断逻辑，如判断用户网络环境，在wifi下自动播放，在4g环境下给出提示，这中情况下就适合直接选中video并调用video.play来播放视频

但是这种情况下也需要webview的支持，如在手Q下可以做到直接调用，在微信下因为不允许视频直接播放，则必须通过用户的真实操作来触发调用video.play(),这就是各种微信的h5活动页面需要引导用户进行一下点击操作才开始的原因。

同时发现真实点击必须使用触发 touchend、click、doubleclick 或 keydown 事件等标准的事件才能触发，使用Zepto封装过的tap事件并不能触发播放器的播放

### 3.页面内联播放问题

在iOS Safari和一些安卓的一些浏览器下播放视频的时候，不能在h5页面中播放视频，系统会自动接管视频

如果需要在h5页面内播放视频，需要在视频标签上加上 webkit-playsinline，在iOS10以后，需要加上playsinline,建议同时加上这两个属性，同时需要app支持这种模式，手Q和微信都支持这种模式

```html
 //在html
    <video id="player" webkit-playsinline playsinline >    //在app内设置webview属性
    webview.allowsInlineMediaPlayback = YES;
```

### 4.视频的高度问题

在安卓下，一些浏览器如QQ浏览器和UC浏览器，系统会把视频的层级调到最高，所以如果想在页面上显示dom元素，都会被视频盖住，单纯的设置该dom的z-index是无效的，
![image](http://mmbiz.qpic.cn/mmbiz_png/70ke266reibiaSvZl1oNrfGpaJI82fibdVOATKF4Al4s46LicVkINo1lnUU14GGtQ4OZrkCFbUicaWW8QOB4HcR3CPA/0?)

解决方案：
1.在弹出会显示在视频上方dom的时候暂停视频播放

2.将视频所在的dom的父元素的高度设为1

3.处理完弹出的事件后将视频所在的父元素高度还原

### 5.视频的默认播放图标

![image](http://mmbiz.qpic.cn/mmbiz_png/70ke266reibiaSvZl1oNrfGpaJI82fibdVO9AmlnPb8iaBv5hvJwSia28O5jV0GJbYOmC2xwUtobdxhxoKu2C9KfHew/0?)

在iOS都会默认显示，不能通过js控制，但是可以通过css样式将其隐藏

```css
 video::-webkit-media-controls-start-playback-button {
            display: none;
    }
```

### 5.视频的控制栏

在h5播放的时候，如果在video标签上设置了controls属性，则会在视频里显示控制栏

![image]("http://mmbiz.qpic.cn/mmbiz_png/70ke266reibiaSvZl1oNrfGpaJI82fibdVOcs7MSswmPVoC1MM0kvwHElVpG5dHXopyEjQxiaKsyj1BGSuUqhJMMLw/0?)

需要注意的是这个控制栏是系统webview自带的，无法通过css控制其样式，建议不要使用这个属性而是自己通过dom自己制作一套控制条


### 6.视频的刷新
我们知道video暴露了play和pause方法来提供视频的播放和暂停，但是h5没有标准的刷新方法，如果我们想实现视频的刷新，则需要通过js实现

```javascript
var player = $('#player')[0];
player.load();
setTimeout(function () {
     player.play();
})
```

### 7.视频的全屏问题

1）全屏api

h5暴露了一个webkitRequestFullScreen方法，可以让每个dom都请求全屏，当然video标签也可以使用。

但是在测试中发现，一些安卓机不支持该属性，如小米手机，所以需要在调用的时候进行一下判断

```javascript
var player = $('#player')[0];if (player.webkitSupportsFullscreen) {
    player.webkitEnterFullscreen();
} else {
   player.webkitRequestFullScreen();
}
```

2）系统接管播放

我们上边说道调用h5的webkitRequestFullScreen方法来进入视频的全屏，那么这个方法会使浏览器完全接管视频播放，如图所示

![image](http://mmbiz.qpic.cn/mmbiz_png/70ke266reibiaSvZl1oNrfGpaJI82fibdVO6Ix77ee8DozBUkPcwnsfIIRiclRhl2icwNDhvuwKaiayyIXdpZicUcc52g/0?)

这种接管的后果是这时的我们是没有办法控制视频的播放，也没有办法在上面浮动我们的dom元素，如弹幕，礼物这些，会完全被视频盖在下面，所以我们的目标即是解决这种系统接管的问题

3）使用伪全屏（样式全屏）

样式全屏的核心是设置video标签的宽高，使其撑满整个webview，看上去像全屏一样

但是因为视频一般都是16:9的宽高比，所以在竖屏情况下不能很好的做到铺满整个屏幕

![image](http://mmbiz.qpic.cn/mmbiz_png/70ke266reibiaSvZl1oNrfGpaJI82fibdVOLmb8G92HFdo83JvtIv0UfbicKEjXM8F5gAdL9ibjXgDR8gPp1HpV4kBA/0?)

而一般用户进入页面基本都是竖屏，所以我们就要考虑怎么让用户在竖屏点击全屏按钮时，能体验到像终端app一样自动进入横屏全屏的体验，下面有两种方案

1.在用户点击全屏时候，通过css3属性旋转屏幕

通过css的transform，我们可以把dom元素旋转显示

通过-webkit-transform: rotate(90deg)并设置video的高度为当前webview的宽度，video的宽度为当前webview的高度来实现旋转全屏。

这种模式的显示没有太大问题，但因为是通过css控制的页面dom显示，对于原生的空间不能很好的控制，如系统的键盘

![image](http://mmbiz.qpic.cn/mmbiz_png/70ke266reibiaSvZl1oNrfGpaJI82fibdVOY7ibMSyTLHc6UBeBhZnRtFc6CXI39icqFzytnmxBX51Yle6uVMghZ8mQ/0?)

在拉起键盘输入弹幕的时候，键盘不受控制还是竖屏显示了

如果页面不涉及与原生组件的交互，那么这种方案是一种很可行且兼容性比较好的方案

2.用户在点击全屏时，通过js api来控制webview旋转横屏

在手Q里，我们和终端的同学合作添加了控制webview横竖屏的接口

在用户点击全屏的时候，先判断当前是否横屏

```javascript
  /**
     * 是否横屏
     */
    function isHorizontal() {       
        if (window.orientation != undefined) {        
            return (window.orientation == 90 || window.orientation == -90);
        } else {         
            return window.innerWidth > window.innerHeight;
        }
    }
//设置webview为横竖屏
 mqq.ui.setWebViewBehavior({         
      orientation: 0 //0是竖屏，1是横屏
 });
```

如果是竖屏则强制webview旋转进入横屏，同时监听页面的resize方法，页面横竖屏变化的时候会触发这个方法，在这个方法里再动态的调整video的宽高来铺满整个屏幕

![image](http://mmbiz.qpic.cn/mmbiz_png/70ke266reibiaSvZl1oNrfGpaJI82fibdVOyvOebaiahXQISeCSl63XvFF3dqRg2gnmXUP2T0tYxe4swQ1ydp29iaOg/0?wx_fmt=png)
