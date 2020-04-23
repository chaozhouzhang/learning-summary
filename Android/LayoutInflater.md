# 1、LayoutInfalter的作用

查看源码注释，可以知道其作用为将布局XML文件实例化为其相应的View对象。
```
Instantiates a layout XML file into its corresponding View objects.
```

并且可以使用以下三种方式获取布局解析服务，其实最后都是使用的第三种：
```
Activity.getLayoutInflater();
```
```
LayoutInflater.from(context);
```
```
Context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
```
# 2、LayoutInfalter的使用

`ViewDrawActivity.java`
```java
public class ViewDrawActivity extends AppCompatActivity {
    private LinearLayout mLinearLayoutMain;
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_view_draw);
        mLinearLayoutMain = findViewById(R.id.ll_main);
        //获取XML对应的View对象
        View viewRootNull = LayoutInflater.from(this).inflate(R.layout.layout_button, null);
        //将View对象添加到其他布局中
        mLinearLayoutMain.addView(viewRootNull);
    }
}
```

`activity_view_draw.xml`
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/ll_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
</LinearLayout>
```

`layout_button.xml`
```
<?xml version="1.0" encoding="utf-8"?>
<Button xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="@string/app_name">
</Button>
```

运行结果：
![](https://github.com/chaozhouzhang/learning-summary/blob/master/Android/viewRootNull.png?raw=true)

# 3、LayoutInfalter的源码解析

## 3.1、尝试获取已经提前编译好的View对象

`LayoutInfalter.java`
```
public View inflate(@LayoutRes int resource, @Nullable ViewGroup root, boolean attachToRoot) {
    final Resources res = getContext().getResources();
	//使用反射尝试获取已经提前编译好的View对象
    View view = tryInflatePrecompiled(resource, res, root, attachToRoot);
    if (view != null) {
        return view;
    }
    //根据资源ID获取布局的XML资源解析器
    XmlResourceParser parser = res.getLayout(resource);
    try {
    	//使用XML资源解析器对XML进行解析
        return inflate(parser, root, attachToRoot);
    } finally {
        parser.close();
    }
}
```
先直接使用反射尝试获取已经提前编译好的View对象，如果获取到直接返回View对象，如果获取不到再根据资源ID获取布局的XML资源解析器，然使用XML资源解析器对XML进行解析。

## 3.2、将XML解析并实例化后的View对象进行添加返回操作
```java
    public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) {
        synchronized (mConstructorArgs) {
            final Context inflaterContext = mContext;
            final AttributeSet attrs = Xml.asAttributeSet(parser);
            Context lastContext = (Context) mConstructorArgs[0];
            mConstructorArgs[0] = inflaterContext;
            View result = root;
            try {
                advanceToRootNode(parser);
                final String name = parser.getName();
                if (TAG_MERGE.equals(name)) {
                //如果布局根标签为merge，那么要求传入的root不能为空，且attachToRoot需为true
                    if (root == null || !attachToRoot) {
                        throw new InflateException("<merge /> can be used only with a valid ViewGroup root and attachToRoot=true");
                    }
                    //继续解析XML
                    rInflate(parser, root, inflaterContext, attrs, false);
                } else {
                	//根据XML中的根标签生成根View对象
                    // Temp is the root view that was found in the xml
                    final View temp = createViewFromTag(root, name, inflaterContext, attrs);
                    ViewGroup.LayoutParams params = null;
                    if (root != null) {
                    //如果布局提供参数，则创建匹配root的布局参数
                        // Create layout params that match root, if supplied
                        params = root.generateLayoutParams(attrs);
                        if (!attachToRoot) {
                        //如果attachToRoot==false，则当使用addView的时候再设置布局参数
                            // Set the layout params for temp if we are not
                            // attaching. (If we are, we use addView, below)
                            temp.setLayoutParams(params);
                        }
                    }
                    //解析子布局
                    // Inflate all children under temp against its context.
                    rInflateChildren(parser, temp, attrs, true);
                    //如果root!=null，且attachToRoot==true，则自动将布局添加到root中，并设置布局参数
                    // We are supposed to attach all the views we found (int temp)
                    // to root. Do that now.
                    if (root != null && attachToRoot) {
                        root.addView(temp, params);
                    }
                    // Decide whether to return the root that was passed in or the
                    // top view found in xml.
                    //当root==null或者attachToRoot==false，则直接返回View对象
                    if (root == null || !attachToRoot) {
                        result = temp;
                    }
                }
            } catch (XmlPullParserException e) {
                final InflateException ie = new InflateException(e.getMessage(), e);
                ie.setStackTrace(EMPTY_STACK_TRACE);
                throw ie;
            } catch (Exception e) {
                final InflateException ie = new InflateException(
                        getParserStateDescription(inflaterContext, attrs)
                        + ": " + e.getMessage(), e);
                ie.setStackTrace(EMPTY_STACK_TRACE);
                throw ie;
            } finally {
                // Don't retain static reference on context.
                mConstructorArgs[0] = lastContext;
                mConstructorArgs[1] = null;
            }
            return result;
        }
    }
```


### 1、布局根标签为merge
首先判断根标签是否是merge标签，如果布局根标签为merge，那么要求传入的root不能为空，且attachToRoot需为true，然后继续解析XML，也就是rInflate()方法，否则会报以下异常：
```
Caused by: android.view.InflateException: Binary XML file line #2: <merge /> can be used only with a valid ViewGroup root and attachToRoot=true
```

如果布局根标签不是merge，首先根据XML中的根标签生成根View对象，也就是createViewFromTag()方法，然后解析子布局，也就是rInflateChildren()方法。然后分以下几种情况进行View对象的添加操作：

### 2、root==null
直接返回生成的View对象，需要使用addView添加到其他布局中，添加的时候会忽略布局最外层的参数。

也就是说，修改了Button布局为固定大小，最后的加载结果还是不变：
`layout_button.xml`
```
<?xml version="1.0" encoding="utf-8"?>
<Button xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="360dp"
    android:layout_height="360dp"
    android:text="@string/app_name">
</Button>
```
![](https://github.com/chaozhouzhang/learning-summary/blob/master/Android/viewRootNull.png?raw=true)

### 3、attachToRoot==true
如果root!=null，默认情况下attachToRoot==true，那么给加载的布局文件指定root为父布局，并且直接添加到root布局中，添加时不忽略布局最外层的参数，然后返回View对象。

layout_width和layout_height是用于设置View在布局中的大小的，但如果还没有依附在父布局中，也就是在Root为null的情况下，这两个的设置将会无效。平时在Activity中指定布局文件的时候，最外层的那个布局是可以指定大小的，layout_width和layout_height都是有作用的，这主要是因为，在setContentView()方法中，Android是使用root!=null和attachToRoot==true的方式进行添加布局的，自动将布局添加到FrameLayout布局contentParent中，所以layout_width和layout_height属性才会有效果。

`AppCompatDelegateImpl.java`
```
@Override
public void setContentView(int resId) {
    ensureSubDecor();
    ViewGroup contentParent = mSubDecor.findViewById(android.R.id.content);
    contentParent.removeAllViews();
    LayoutInflater.from(mContext).inflate(resId, contentParent);
    mAppCompatWindowCallback.getWrapped().onContentChanged();
}
```
我们修改添加Button的方式为root!=null，attachToRoot==true，就可以使得修改Button布局为固定大小产生效果：
`ViewDrawActivity.java`
```java
public class ViewDrawActivity extends AppCompatActivity {
    private LinearLayout mLinearLayoutMain;
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_view_draw);
        mLinearLayoutMain = findViewById(R.id.ll_main);
        //解析获取View对象的同时，添加View到root布局中
        LayoutInflater.from(this).inflate(R.layout.layout_button, mLinearLayoutMain);
    }
}
```
![](https://github.com/chaozhouzhang/learning-summary/blob/master/Android/viewRootNotNullAttachToRoot.png?raw=true)
### 4、attachToRoot==false
如果root!=null，且如果attachToRoot==false，那么给加载的布局文件指定root为父布局，但无法直接添加到root布局中，需要使用addView添加到其他布局中，但不一定非要添加到所传的root布局中，添加时不忽略布局最外层的参数，然后返回View对象。
 
## 3.3、使用XML资源解析器对XML进行解析

在3.2中想要获取XML解析并实例化的View对象，就必须先进行XML解析得到AttributeSet，
也就是rInflate()、rInflateChildren()等方法；


```
    void rInflate(XmlPullParser parser, View parent, Context context,
            AttributeSet attrs, boolean finishInflate) throws XmlPullParserException, IOException {

        final int depth = parser.getDepth();
        int type;
        boolean pendingRequestFocus = false;
		//遍历节点
        while (((type = parser.next()) != XmlPullParser.END_TAG ||
                parser.getDepth() > depth) && type != XmlPullParser.END_DOCUMENT) {

            if (type != XmlPullParser.START_TAG) {
                continue;
            }
            final String name = parser.getName();
            if (TAG_REQUEST_FOCUS.equals(name)) {
				//request_foucs
                pendingRequestFocus = true;
                consumeChildElements(parser);
            } else if (TAG_TAG.equals(name)) {
           		//tag
                parseViewTag(parser, parent, attrs);
            } else if (TAG_INCLUDE.equals(name)) {
            	//include
                if (parser.getDepth() == 0) {
                    throw new InflateException("<include /> cannot be the root element");
                }
                parseInclude(parser, context, parent, attrs);
            } else if (TAG_MERGE.equals(name)) {
            	//merge
                throw new InflateException("<merge /> must be the root element");
            } else {
            	//生成View对象，然后继续循环解析子布局
                final View view = createViewFromTag(parent, name, context, attrs);
                final ViewGroup viewGroup = (ViewGroup) parent;
                final ViewGroup.LayoutParams params = viewGroup.generateLayoutParams(attrs);
                rInflateChildren(parser, view, attrs, true);
                viewGroup.addView(view, params);
            }
        }

        if (pendingRequestFocus) {
            parent.restoreDefaultFocus();
        }

        if (finishInflate) {
            parent.onFinishInflate();
        }
    }
```

然后使用反射根据tag对解析后的AttributeSet进行实例化得到View对象，也就是createViewFromTag()、createView()等方法。
```
@Nullable
public final View createView(@NonNull Context viewContext, @NonNull String name,
        @Nullable String prefix, @Nullable AttributeSet attrs)
        throws ClassNotFoundException, InflateException {
    Objects.requireNonNull(viewContext);
    Objects.requireNonNull(name);
    Constructor<? extends View> constructor = sConstructorMap.get(name);
    if (constructor != null && !verifyClassLoader(constructor)) {
        constructor = null;
        sConstructorMap.remove(name);
    }
    Class<? extends View> clazz = null;
    try {
    	//创建构造器
        if (constructor == null) {
            // Class not found in the cache, see if it's real, and try to add it
            clazz = Class.forName(prefix != null ? (prefix + name) : name, false,
                    mContext.getClassLoader()).asSubclass(View.class);

            if (mFilter != null && clazz != null) {
                boolean allowed = mFilter.onLoadClass(clazz);
                if (!allowed) {
                    failNotAllowed(name, prefix, viewContext, attrs);
                }
            }
            constructor = clazz.getConstructor(mConstructorSignature);
            constructor.setAccessible(true);
            sConstructorMap.put(name, constructor);
        } else {
            // If we have a filter, apply it to cached constructor
            if (mFilter != null) {
                // Have we seen this name before?
                Boolean allowedState = mFilterMap.get(name);
                if (allowedState == null) {
                    // New class -- remember whether it is allowed
                    clazz = Class.forName(prefix != null ? (prefix + name) : name, false, mContext.getClassLoader()).asSubclass(View.class);
                    boolean allowed = clazz != null && mFilter.onLoadClass(clazz);
                    mFilterMap.put(name, allowed);
                    if (!allowed) {
                        failNotAllowed(name, prefix, viewContext, attrs);
                    }
                } else if (allowedState.equals(Boolean.FALSE)) {
                    failNotAllowed(name, prefix, viewContext, attrs);
                }
            }
        }
        Object lastContext = mConstructorArgs[0];
        mConstructorArgs[0] = viewContext;
        Object[] args = mConstructorArgs;
        args[1] = attrs;
        try {
        	//使用反射创建View的实例对象
            final View view = constructor.newInstance(args);
            if (view instanceof ViewStub) {
            	//如果是ViewStub类型，当稍后解析ViewStub时使用同个上下文解析
                // Use the same context when inflating ViewStub later.
                final ViewStub viewStub = (ViewStub) view;
                viewStub.setLayoutInflater(cloneInContext((Context) args[0]));
            }
            return view;
        } finally {
            mConstructorArgs[0] = lastContext;
        }
    } catch (NoSuchMethodException e) {
        final InflateException ie = new InflateException(
                getParserStateDescription(viewContext, attrs)
                + ": Error inflating class " + (prefix != null ? (prefix + name) : name), e);
        ie.setStackTrace(EMPTY_STACK_TRACE);
        throw ie;

    } catch (ClassCastException e) {
        // If loaded class is not a View subclass
        final InflateException ie = new InflateException(
                getParserStateDescription(viewContext, attrs)
                + ": Class is not a View " + (prefix != null ? (prefix + name) : name), e);
        ie.setStackTrace(EMPTY_STACK_TRACE);
        throw ie;
    } catch (ClassNotFoundException e) {
        // If loadClass fails, we should propagate the exception.
        throw e;
    } catch (Exception e) {
        final InflateException ie = new InflateException(
                getParserStateDescription(viewContext, attrs) + ": Error inflating class "
                        + (clazz == null ? "<unknown>" : clazz.getName()), e);
        ie.setStackTrace(EMPTY_STACK_TRACE);
        throw ie;
    } finally {
        Trace.traceEnd(Trace.TRACE_TAG_VIEW);
    }
}
```


附上LayoutInflater脑图：
![](https://github.com/chaozhouzhang/learning-summary/blob/master/Android/LayoutInflater.png?raw=true)


欢迎关注Android技术堆栈，专注于Android技术学习的公众号，致力于提高Android开发者们的专业技能！

![Android技术堆栈](https://mmbiz.qpic.cn/mmbiz_jpg/MADc6NnIysDjTRbKsg6y2G5eqqQkPDiak4V8jqKLmntDgAfFE8LOibxnSdfJESLJEM8ibrN9RGiamib4rYCt3cU08aQ/0?wx_fmt=jpeg)




