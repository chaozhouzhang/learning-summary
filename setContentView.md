
# Activity
1、在自定义Activity的onCreate中调用setContentView()方法：
`ViewDrawActivity.java`
```java
public class ViewDrawActivity extends AppCompatActivity {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_view_draw);
    }
}
```


2、进入AppCompatActivity的setContentView()方法：
`AppCompatActivity.java`
```java
@Override
public void setContentView(@LayoutRes int layoutResID) {
    getDelegate().setContentView(layoutResID);
}
```

2.1、先通过getDelegate()方法，获取继承自抽象类AppCompatDelegate的实现类AppCompatDelegateImpl的实例：
`AppCompatActivity.java`
```java
public AppCompatDelegate getDelegate() {
    if (mDelegate == null) {
        mDelegate = AppCompatDelegate.create(this, this);
    }
    return mDelegate;
}
```
`AppCompatDelegate.java`
```java
@NonNull
public static AppCompatDelegate create(@NonNull Activity activity,
        @Nullable AppCompatCallback callback) {
    return new AppCompatDelegateImpl(activity, callback);
}
```

2.2、获取到AppCompatDelegateImpl的实例后，回到步骤2，继续调用实例的setContentView()实现方法：
`AppCompatDelegateImpl.java`
```java
@Override
public void setContentView(int resId) {
	//安装子装饰视图
    ensureSubDecor();
    //获取内容视图的添加位置
    ViewGroup contentParent = mSubDecor.findViewById(android.R.id.content);
    contentParent.removeAllViews();
    //解析内容视图
    LayoutInflater.from(mContext).inflate(resId, contentParent);
    //告知内容视图已经变化
    mAppCompatWindowCallback.getWrapped().onContentChanged();
}
```

# DecorView
3.1、安装子装饰视图
`AppCompatDelegateImpl.java`
```java
    private void ensureSubDecor() {
    		//如果子装饰视图未安装
        if (!mSubDecorInstalled) {
        	//创建子装饰视图
            mSubDecor = createSubDecor();
            //获取标题
            CharSequence title = getTitle();
            //标题不空
            if (!TextUtils.isEmpty(title)) {
            	//装饰内容父亲
                if (mDecorContentParent != null) {
                	//通过窗口的顶级装饰布局设置窗口标题
                    mDecorContentParent.setWindowTitle(title);
                } else if (peekSupportActionBar() != null) {
                	//通过ActionBar设置窗口标题
                    peekSupportActionBar().setWindowTitle(title);
                } else if (mTitleView != null) {
                	//直接通过标题文本空间设置标题
                    mTitleView.setText(title);
                }
            }
            //窗口大小设置
            applyFixedSizeWindow();
            //子装饰视图已安装
            onSubDecorInstalled(mSubDecor);
            mSubDecorInstalled = true;
            PanelFeatureState st = getPanelState(FEATURE_OPTIONS_PANEL, false);
            if (!mIsDestroyed && (st == null || st.menu == null)) {
            	//刷新面板菜单
                invalidatePanelMenu(FEATURE_SUPPORT_ACTION_BAR);
            }
        }
    }
```

"You need to use a Theme.AppCompat theme (or descendant) with this activity."
也就是经常说的使用了AppCompatActivity却没有指定Theme.AppCompat主题


