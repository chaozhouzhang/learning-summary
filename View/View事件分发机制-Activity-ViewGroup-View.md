

```java
本文源码基于Android 10.0
```

# 事件

|事件|解释|
|----|----|
|ACTION_DOWN|手指初次接触到屏幕时触发|
|ACTION_MOVE|手指在屏幕上滑动时触发，会多次触发|
|ACTION_UP|手指离开屏幕时触发|
|ACTION_CANCEL|事件被上层拦截时触发|

# Activity对触摸事件的分发流程



首先，用户触摸屏幕之后，会调用Activity的dispatchTouchEvent()方法：

`ViewDispatchActivity.java`
```java
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
    Log.e("ViewDispatchActivity", "dispatchTouchEvent");
    return super.dispatchTouchEvent(ev);
}
```
也就是：
`Activity.java`
```java
public boolean dispatchTouchEvent(MotionEvent ev) {
    if (ev.getAction() == MotionEvent.ACTION_DOWN) {
        onUserInteraction();
    }
    if (getWindow().superDispatchTouchEvent(ev)) {
        return true;
    }
    return onTouchEvent(ev);
}
```
以上可知：

## Activity.onUserInteraction()

1、如果事件类型为ACTION_DOWN按下操作，则会回调onUserInteraction()方法，表示用户正在与应用进行交互，可以在此实现与用户的交互功能。


`ViewDispatchActivity.java`
```java
@Override
public void onUserInteraction() {
    Log.e("ViewDispatchActivity", "onUserInteraction");
    super.onUserInteraction();
}
```
`Activity.java`
也就是：
```java
public void onUserInteraction() {
}
```
## ViewGroup.superDispatchTouchEvent()
2、如果Window抽象类的实现类PhoneWindow的superDispatchTouchEvent()方法返回true，则当前方法也返回true。

继续看下PhoneWindow，可知调用的是DecorView的superDispatchTouchEvent()方法：

`PhoneWindow.java`
```java
@Override
public boolean superDispatchTouchEvent(MotionEvent event) {
    return mDecor.superDispatchTouchEvent(event);
}
```
继续看下DecorView，可知调用的是ViewGroup的dispatchTouchEvent()方法：
`DecorView.java`
```java
public boolean superDispatchTouchEvent(MotionEvent event) {
    return super.dispatchTouchEvent(event);
}
```
继续看下ViewGroup：
`ViewGroup.java`
```java
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
	......
	return handled;
}
```
由此，我们知道了Activity是如何将事件传递给ViewGroup的，并且知道了ViewGroup的dispatchTouchEvent()方法返回结果决定了Activity的dispatchTouchEvent()方法的返回结果。
## Activity.onTouchEvent()
3、如果Window抽象类的实现类PhoneWindow的superDispatchTouchEvent()方法返回false，则调用onTouchEvent()方法。
`ViewDispatchActivity.java`
```java
@Override
public boolean onTouchEvent(MotionEvent event) {
    Log.e("ViewDispatchActivity","onTouchEvent");
    return super.onTouchEvent(event);
}
```
也就是：
`Activity.java`
```java
public boolean onTouchEvent(MotionEvent event) {
    if (mWindow.shouldCloseOnTouch(this, event)) {
        finish();
        return true;
    }
    return false;
}
```
以上可知，如果Window的shouldCloseOnTouch()方法返回true，则结束当前Activity，并且返回true；否则返回false。

继续看下Window的shouldCloseOnTouch()方法：
`Window.java`
```java
public boolean shouldCloseOnTouch(Context context, MotionEvent event) {
    final boolean isOutside =
            event.getAction() == MotionEvent.ACTION_UP && isOutOfBounds(context, event)
            || event.getAction() == MotionEvent.ACTION_OUTSIDE;
    if (mCloseOnTouchOutside && peekDecorView() != null && isOutside) {
        return true;
    }
    return false;
}
```
以上可知，如果触摸事件在边界外，将会结束当前Activity，并且返回true，否则返回false。


## 总结

![](https://github.com/chaozhouzhang/learning-summary/blob/master/View/Activity%E5%AF%B9%E8%A7%A6%E6%91%B8%E4%BA%8B%E4%BB%B6%E7%9A%84%E5%88%86%E5%8F%91%E6%B5%81%E7%A8%8B.png?raw=true)

## 后续
ViewGroup和View对触摸事件的分发流程，笔者将在接下来的文章进行讲解，敬请关注。

欢迎关注Android技术堆栈，专注于Android技术学习的公众号，致力于提高Android开发者们的专业技能！

![Android技术堆栈](https://mmbiz.qpic.cn/mmbiz_jpg/MADc6NnIysDjTRbKsg6y2G5eqqQkPDiak4V8jqKLmntDgAfFE8LOibxnSdfJESLJEM8ibrN9RGiamib4rYCt3cU08aQ/0?wx_fmt=jpeg)



