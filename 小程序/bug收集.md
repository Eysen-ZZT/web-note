## wx.redirectTo() 跳转闪屏
暂时没有解决方案，替换为 `wx.navigateTo()` ，或者将页面做成组件，放在一页，通过`wx:if` 进行切换，添加动画过渡。

## 小程序animation只执行一次

可以通过复原动画实现动画反复执行。

例如:

```javascript
const animation = wx.createAnimation({
  duration: 3000,
  timingFunction: 'ease',
});
this.animation = animation;
animation.rotate(angle + 360 * 8).step();
this.setData({
  animationData: animation.export()
});
```

以上动画效果，可以通过在创建一组动画将旋转角度调整为 0 就可以了。

```javascript
animation1.rotate(0).step();
this.setData({
  animationData: animation1.export()
});
```

## swiper高度自适应问题

由于小程序`Swiper`的高度是固定150的，并且对其设置`height： auto`；是不生效的，只能通过固定值px来改变其高度，这样一来高度就被写死，所以，我们只能通过js动态改变其高度。一般情况下，在接口返回值渲染完成时进行dom高度获取，然后将其高度赋值给`Swiper`。

## input,textarea点击遮罩层依旧弹出键盘

解决方案： 

1. 使用cover-view(不推荐)，对css支持不够
2. 使用view遮住input，点击view的时候手动触发input的focus事件

## 小程序原生组件的坑

1. textarea层级过高导致文字盖过弹窗
2. text组件设置line-height无效
3. textarea在ios中存在内边距
