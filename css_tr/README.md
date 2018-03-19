>前往博客查看更直观的效果[CSS 动画 Cheat Sheet](https://ns96.com/2018/03/19/css-ss-an/)
Github 不支持JsFiddle 推荐前往博客查看直观效果

# CSS 动画
CSS 动画一共有如下三种：
- CSS Transition 平滑的改变CSS的值
- CSS Animation  适用于CSS2，CSS3
- CSS Transforms 变换主要实现（拉伸，压缩，旋转，偏移）

下面依次介绍一下：

# CSS Transition

## 基本运用
在CSS 3引入Transition（过渡）这个概念之前，CSS是没有时间轴的。也就是说，所有的状态变化，都是即时完成。

例如：
```css
.img {
  background-color: pink;
  height: 15px;
  width: 15px;
  transition: 1s;
}

.img:hover {
  height: 200px;
  width: 200px;
}
```

当鼠标放置在粉色的div上时，原本瞬间放大的div会在1s内慢慢放大。

[前往jsfiddle查看](https://jsfiddle.net/raphaelli96/e5bozenj/)