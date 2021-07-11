# View的初始化



![image-20210701030551081](C:\Users\ZouYongpeng\AppData\Roaming\Typora\typora-user-images\image-20210701030551081.png)



## ActivityThread

### `performLaunchActivity`

1. 实例化 activity
2. 对activity 执行绑定操作

```java
	private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
        //......
        ContextImpl appContext = createBaseContextForActivity(r);
        // 1.实例化 activity
        Activity activity = mInstrumentation.newActivity(cl, component.getClassName(), r.intent);
        try {
            // ......
            if (activity != null) {
                //......
                // 2、对activity 执行绑定操作
                activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstances, config,
                        r.referrer, r.voiceInteractor, window, r.configCallback,
                        r.assistToken);
                //......
            }
        } 
        return activity;
    }
```



## Activity

### `attatch`

1. 实例化一个PhoneWindow对象，并进行初始化，设置各种监听回调
2. 把ActivityThread，ActivityInfo，Application等重要的信息绑定到当前的Activity

```java
	final void attach(Context context, ActivityThread aThread,
            Instrumentation instr, IBinder token, int ident,
            Application application, Intent intent, ActivityInfo info,
            CharSequence title, Activity parent, String id,
            NonConfigurationInstances lastNonConfigurationInstances,
            Configuration config, String referrer, IVoiceInteractor voiceInteractor,
            Window window, ActivityConfigCallback activityConfigCallback, IBinder assistToken) {
        attachBaseContext(context);

        mFragments.attachHost(null /*parent*/);

    	// 1、实例化一个PhoneWindow对象，并且设置监听，如点击事件的回调，窗体消失的回调等等
        mWindow = new PhoneWindow(this, window, activityConfigCallback);
        mWindow.setWindowControllerCallback(mWindowControllerCallback);
        mWindow.setCallback(this);
        mWindow.setOnWindowDismissedCallback(this);
        mWindow.getLayoutInflater().setPrivateFactory(this);
        if (info.softInputMode != WindowManager.LayoutParams.SOFT_INPUT_STATE_UNSPECIFIED) {
            mWindow.setSoftInputMode(info.softInputMode);
        }
        if (info.uiOptions != 0) {
            mWindow.setUiOptions(info.uiOptions);
        }
    
    	// 2、把ActivityThread，ActivityInfo，Application等重要的信息绑定到当前的Activity
        mUiThread = Thread.currentThread();
        mMainThread = aThread;
        mInstrumentation = instr;
        mToken = token;
        mAssistToken = assistToken;
        mIdent = ident;
        mApplication = application;
        mIntent = intent;
        mReferrer = referrer;
        mComponent = intent.getComponent();
        mActivityInfo = info;
        mTitle = title;
        mParent = parent;
        mEmbeddedID = id;
        mLastNonConfigurationInstances = lastNonConfigurationInstances;
        if (voiceInteractor != null) {
            if (lastNonConfigurationInstances != null) {
                mVoiceInteractor = lastNonConfigurationInstances.voiceInteractor;
            } else {
                mVoiceInteractor = new VoiceInteractor(voiceInteractor, this, this,
                        Looper.myLooper());
            }
        }

        mWindow.setWindowManager(
                (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
                mToken, mComponent.flattenToString(),
                (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);
        if (mParent != null) {
            mWindow.setContainer(mParent.getWindow());
        }
        mWindowManager = mWindow.getWindowManager();
        mCurrentConfig = config;

        mWindow.setColorMode(info.colorMode);
        mWindow.setPreferMinimalPostProcessing(
                (info.flags & ActivityInfo.FLAG_PREFER_MINIMAL_POST_PROCESSING) != 0);

        setAutofillOptions(application.getAutofillOptions());
        setContentCaptureOptions(application.getContentCaptureOptions());
    }
```



然后 `activity`会在 `onCreate()`时通过我们传入的布局文件 `R.layout.xxx` 调用 `setContentView` 去加载布局

### `setContentView`

我们会重写 `Activity.onCreate()` ，并调用 `setContentView(R.layout.activity_xxx)` 

```java
	public void setContentView(@LayoutRes int layoutResID) {
        // 实际上是调用performLaunchActivity()中实例化的PhoneWindow去setContentView()
        getWindow().setContentView(layoutResID);
        initWindowDecorActionBar();
    }
```



## PhoneWindow

### `setContentView`

1. 通过 `installDecor()` 创建 DecorView 作为最顶层的view，并根据窗口style先加载一个初始布局，得到一个可以真正存放内容view的容器 mContentParent (`android.R.id.content`)

   > <DecorView>
   >
   > ​	<LinearLayout>
   >
   > ​		<ActionBar>
   >
   > ​		<FrameLayout>

2. 让 `LayoutInflater` 去 inflate 布局，然后放在 mContentParent 里

```java
public void setContentView(int layoutResID) {
        if (mContentParent == null) {
            // 当mContentParent为空的时候，会调用installDecor()生成DecorView作为顶层View
            installDecor();
        } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            mContentParent.removeAllViews();
        }

        if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                    getContext());
            transitionTo(newScene);
        } else {
            // 实例化传进来的布局
            mLayoutInflater.inflate(layoutResID, mContentParent);
        }
        mContentParent.requestApplyInsets();
        final Callback cb = getCallback();
        if (cb != null && !isDestroyed()) {
            cb.onContentChanged();
        }
        mContentParentExplicitlySet = true;
    }
```



### `installDecor`

1. `generateDecor()` 生成DecorView 
2. `generateLayout(mDecor)` 获取DecorView的内容区域 android.R.id.content

```java
private void installDecor() {
        mForceDecorInstall = false;
    	// 生成DecorView
        if (mDecor == null) {
            mDecor = generateDecor(-1);
            mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
            mDecor.setIsRootNamespace(true);
            if (!mInvalidatePanelMenuPosted && mInvalidatePanelMenuFeatures != 0) {
                mDecor.postOnAnimation(mInvalidatePanelMenuRunnable);
            }
        } else {
            mDecor.setWindow(this);
        }
    
        if (mContentParent == null) {
            // 获取DecorView中的内容区域
            mContentParent = generateLayout(mDecor);

            // 寻找DecorView中的DecorContentParent处理PanelMenu等系统内置的挂在view
            final DecorContentParent decorContentParent = (DecorContentParent) mDecor.findViewById(
                    R.id.decor_content_parent);
            if (decorContentParent != null) {
                mDecorContentParent = decorContentParent;
                ......
                PanelFeatureState st = getPanelState(FEATURE_OPTIONS_PANEL, false);
                if (!isDestroyed() && (st == null || st.menu == null) && !mIsStartingWindow) {
                    invalidatePanelMenu(FEATURE_ACTION_BAR);
                }
            } else {
                ......
            }

            if (mDecor.getBackground() == null && mBackgroundFallbackDrawable != null) {
                mDecor.setBackgroundFallback(mBackgroundFallbackDrawable);
            }

            // 处理转场动画
            if (hasFeature(FEATURE_ACTIVITY_TRANSITIONS)) {
                ......
            }
        }
    }
```



#### `generateDecor`

```java
protected DecorView generateDecor(int featureId) {
        Context context;
        if (mUseDecorContext) {
            Context applicationContext = getContext().getApplicationContext();
            if (applicationContext == null) {
                context = getContext();
            } else {
                // 一般是一个DecorContext
                context = new DecorContext(applicationContext, this);
                if (mTheme != -1) {
                    context.setTheme(mTheme);
                }
            }
        } else {
            context = getContext();
        }
        return new DecorView(context, featureId, this, getAttributes());
    }
```



#### `generateLayout`

```java
    protected ViewGroup generateLayout(DecorView decor) {
        
        // 获取窗口style
        TypedArray a = getWindowStyle();
        
        // 根据style获取标志位
        ......
            
        // 根据标志位设置资源
        ......

        // 根据标志位获取布局资源文件
        int layoutResource;
        int features = getLocalFeatures();
        if ((features & ((1 << FEATURE_LEFT_ICON) | (1 << FEATURE_RIGHT_ICON))) != 0) {
            if (mIsFloating) {
                layoutResource = res.resourceId;
            } else {
                layoutResource = R.layout.screen_title_icons;
            }
        } else if ((features & ((1 << FEATURE_PROGRESS) | (1 << FEATURE_INDETERMINATE_PROGRESS))) != 0
                && (features & (1 << FEATURE_ACTION_BAR)) == 0) {
            layoutResource = R.layout.screen_progress;
        } else if ((features & (1 << FEATURE_CUSTOM_TITLE)) != 0) {
            if (mIsFloating) {
                layoutResource = res.resourceId;
            } else {
                layoutResource = R.layout.screen_custom_title;
            }
        } else if ((features & (1 << FEATURE_NO_TITLE)) == 0) {
            if (mIsFloating) {
                layoutResource = res.resourceId;
            } else if ((features & (1 << FEATURE_ACTION_BAR)) != 0) {
                layoutResource = a.getResourceId(
                        R.styleable.Window_windowActionBarFullscreenDecorLayout,
                        R.layout.screen_action_bar);
            } else {
                layoutResource = R.layout.screen_title;
            }
        } else if ((features & (1 << FEATURE_ACTION_MODE_OVERLAY)) != 0) {
            layoutResource = R.layout.screen_simple_overlay_action_mode;
        } else {
            layoutResource = R.layout.screen_simple;
        }

        mDecor.startChanging();
        // DecorView加载布局资源，每一种资源都要有一个id为android.R.id.content的layout作为存放布局的容器
        mDecor.onResourcesLoaded(mLayoutInflater, layoutResource);
        ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);
        ......
        return contentParent;
    }

```



## LayoutInflater

### `inflate`

1. 先通过 `Resources` 获取解析 XML 的解析器 `XmlResourceParser`
2. 使用  `XmlResourceParser`去解析 XML布局文件，根据节点实例化view

```java
	public View inflate(@LayoutRes int resource, @Nullable ViewGroup root) {
        // PhoneWindow传进来的root是id为android.R.id.content的布局容器layout
        return inflate(resource, root, root != null);
    }

	public View inflate(@LayoutRes int resource, @Nullable ViewGroup root, boolean attachToRoot) {
        final Resources res = getContext().getResources();
		......
        
        // 通过Resource获取XmlResourceParser的解析器
        XmlResourceParser parser = res.getLayout(resource);
        try {
            // inflate按照解析器实例化View
            return inflate(parser, root, attachToRoot);
        } finally {
            parser.close();
        }
    }
```



## Resources

### `getLayout`

通过 `loadXmlResourceParser` 告诉底层

```java
	public XmlResourceParser getLayout(@LayoutRes int id) throws NotFoundException {
        // 告诉底层现在解析的资源类型是layout
        return loadXmlResourceParser(id, "layout");
    }

	XmlResourceParser loadXmlResourceParser(@AnyRes int id, @NonNull String type)
            throws NotFoundException {
        // 1 获取TypedValue
        final TypedValue value = obtainTempTypedValue();
        try {
            final ResourcesImpl impl = mResourcesImpl;
            impl.getValue(id, value, true);
            if (value.type == TypedValue.TYPE_STRING) {
                // 让ResourcesImpl和AssetManager去完成解析、加载资源
                return loadXmlResourceParser(value.string.toString(), id,
                        value.assetCookie, type);
            }
            throw new NotFoundException("Resource ID #0x" + Integer.toHexString(id)
                    + " type #0x" + Integer.toHexString(value.type) + " is not valid");
        } finally {
            // 3 释放TypedValue
            releaseTempTypedValue(value);
        }
    }

	XmlResourceParser loadXmlResourceParser(String file, int id, int assetCookie,
                                            String type) throws NotFoundException {
        return mResourcesImpl.loadXmlResourceParser(file, id, assetCookie, type);
    }
```



## LayoutInflater

### `inflate`

```java
	public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) {
        synchronized (mConstructorArgs) {
            Trace.traceBegin(Trace.TRACE_TAG_VIEW, "inflate");

            final Context inflaterContext = mContext;
            final AttributeSet attrs = Xml.asAttributeSet(parser);
            Context lastContext = (Context) mConstructorArgs[0];
            mConstructorArgs[0] = inflaterContext;
            View result = root;

            try {
                // 1、跳过空行、注释等，直至找到 START_TAG : “<” 
                advanceToRootNode(parser);
                final String name = parser.getName();
                
                // 2、开始inflate
                if (TAG_MERGE.equals(name)) {
                    // 2.1 如果xml布局使用了merge优化，那么会检查merge有没有声明一个root,然后调用rInflate
                    if (root == null || !attachToRoot) {
                        throw new InflateException("<merge /> can be used only with a valid "
                                + "ViewGroup root and attachToRoot=true");
                    }
                    rInflate(parser, root, inflaterContext, attrs, false);
                } else {
                    // 2.2 xml是普通的布局
                    // 2.2.1 先通过createViewFromTag实例化对应的view
                    final View temp = createViewFromTag(root, name, inflaterContext, attrs);

                    ViewGroup.LayoutParams params = null;
				
                    // 2.2.2 如果root不为空，那么temp直接获取根布局的LayoutParams
                    if (root != null) {
                        params = root.generateLayoutParams(attrs);
                        if (!attachToRoot) {
                            temp.setLayoutParams(params);
                        }
                    }

                    // 2.2.3、解析生成view之后，继续递归解析子布局
                    rInflateChildren(parser, temp, attrs, true);

                    // We are supposed to attach all the views we found (int temp)
                    // to root. Do that now.
                    if (root != null && attachToRoot) {
                        root.addView(temp, params);
                    }

                    // Decide whether to return the root that was passed in or the
                    // top view found in xml.
                    if (root == null || !attachToRoot) {
                        result = temp;
                    }
                }
            } 
            return result;
        }
    }
```



### `createViewFromTag`

```java
	private View createViewFromTag(View parent, String name, Context context, AttributeSet attrs) {
        return createViewFromTag(parent, name, context, attrs, false);
    }

	View createViewFromTag(View parent, String name, Context context, AttributeSet attrs,
            boolean ignoreThemeAttr) {
        // 如果标签直接是view，则直接获取xml标签中系统中class的属性，这样就能找到view对应的类名
        if (name.equals("view")) {
            name = attrs.getAttributeValue(null, "class");
        }

        // 如果允许并指定一个themeWrapper，那么就包装
        if (!ignoreThemeAttr) {
            final TypedArray ta = context.obtainStyledAttributes(attrs, ATTRS_THEME);
            final int themeResId = ta.getResourceId(0, 0);
            if (themeResId != 0) {
                context = new ContextThemeWrapper(context, themeResId);
            }
            ta.recycle();
        }

        try {
            View view = tryCreateView(parent, name, context, attrs);
            return view;
        }
    }
```



#### `tryCreateView`

```java
public final View tryCreateView(@Nullable View parent, @NonNull String name,Context context,AttributeSet attrs) {
    	// 如果name == "blink"，则创建一个深度链接的布局
        if (name.equals(TAG_1995)) {
            return new BlinkLayout(context, attrs);
        }

        View view;
    	// 让用户或者系统定义的Factory去拦截创建view，如AppCompat会拦截Button创建为AppCompatButton
        if (mFactory2 != null) {
            view = mFactory2.onCreateView(parent, name, context, attrs);
        } else if (mFactory != null) {
            view = mFactory.onCreateView(name, context, attrs);
        } else if (view == null && mPrivateFactory != null) {
            view = mPrivateFactory.onCreateView(parent, name, context, attrs);
        }
    
    	// 如果没有Factory拦截
    	if (view == null) {
            final Object lastContext = mConstructorArgs[0];
            mConstructorArgs[0] = context;
            try {
                if (-1 == name.indexOf('.')) {
                    // 如果name不包含‘.’,说明是系统控件，调用onCreateView，
                    view = onCreateView(context, parent, name, attrs);
                } else {
                    // 否则说明是自定义view，调用createView创建
                    view = createView(context, name, null, attrs);
                }
            } finally {
                mConstructorArgs[0] = lastContext;
            }
        }

        return view;
    }
```



#### `onCreateView`



```java
	protected View onCreateView(String name, AttributeSet attrs)
            throws ClassNotFoundException {
        return createView(name, "android.view.", attrs);
    }

	public final View createView(String name, String prefix, AttributeSet attrs)
            throws ClassNotFoundException, InflateException {
        // 对于前缀prefix，默认有三种：
        // 1、"android.view." 表示系统控件
        // 2、"android.webkit." 表示WebView
        // 3、"android.app."    一般指fragment
        Context context = (Context) mConstructorArgs[0];
        if (context == null) {
            context = mContext;
        }
        return createView(context, name, prefix, attrs);
    }
```



#### `createView`

```java
public final View createView(Context viewContext, String name, String prefix, AttributeSet attrs)
            throws ClassNotFoundException, InflateException {
        Objects.requireNonNull(viewContext);
        Objects.requireNonNull(name);
    
    	// sConstructorMap：保存着所有实例化过的View对应的构造函数，避免重复反射获取
        Constructor<? extends View> constructor = sConstructorMap.get(name);
        if (constructor != null && !verifyClassLoader(constructor)) {
            constructor = null;
            sConstructorMap.remove(name);
        }
        Class<? extends View> clazz = null;

        try {
		   // sConstructorMap没找到对应的构造函数，就通过prefix+name的方式查找类的构造函数，然后存储起来
            if (constructor == null) {
                clazz = Class.forName(prefix != null ? (prefix + name) : name, false,
                        mContext.getClassLoader()).asSubclass(View.class);
                constructor = clazz.getConstructor(mConstructorSignature);
                constructor.setAccessible(true);
                sConstructorMap.put(name, constructor);
            } 

            Object lastContext = mConstructorArgs[0];
            mConstructorArgs[0] = viewContext;
            Object[] args = mConstructorArgs;
            args[1] = attrs;

            // 实例化view
            try {
                final View view = constructor.newInstance(args);
                if (view instanceof ViewStub) {
                    // Use the same context when inflating ViewStub later.
                    final ViewStub viewStub = (ViewStub) view;
                    viewStub.setLayoutInflater(cloneInContext((Context) args[0]));
                }
                return view;
            } finally {
                mConstructorArgs[0] = lastContext;
            }
        }
    }
```

> LayoutInflater全局单例的好处是**加速view实例化的过程，共用反射的构造函数的缓存**



### `rInflate`

```java
	final void rInflateChildren(XmlPullParser parser, View parent, AttributeSet attrs, boolean finishInflate) 				throws XmlPullParserException, IOException {
        rInflate(parser, parent, parent.getContext(), attrs, finishInflate);
    }
	
    void rInflate(XmlPullParser parser, View parent, Context context, AttributeSet attrs, boolean finishInflate) 			throws XmlPullParserException, IOException {

        final int depth = parser.getDepth();
        int type;
        boolean pendingRequestFocus = false;

        // 循环当前xml的节点解析内部的标签，直到到了标签的末尾(“/>”)
        while (((type = parser.next()) != XmlPullParser.END_TAG ||
                parser.getDepth() > depth) && type != XmlPullParser.END_DOCUMENT) {

            if (type != XmlPullParser.START_TAG) {
                continue;
            }

            final String name = parser.getName();

            if (TAG_REQUEST_FOCUS.equals(name)) {
                // 如果name == "requestFocus"，view需要requestFoucs，则设置标志为聚焦
                pendingRequestFocus = true;
                consumeChildElements(parser);
            } else if (TAG_TAG.equals(name)) {
                // name == "tag"
                parseViewTag(parser, parent, attrs);
            } else if (TAG_INCLUDE.equals(name)) {
                // name == "include"
                if (parser.getDepth() == 0) {
                    throw new InflateException("<include /> cannot be the root element");
                }
                parseInclude(parser, context, parent, attrs);
            } else if (TAG_MERGE.equals(name)) {
                // name == "merge"，发现标签是merge则报错
                throw new InflateException("<merge /> must be the root element");
            } else {
                // 正常生成View，并且添加当前的父布局中
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



## 总结

1. ActivityThread：实例化activity

2. Activity：

   * 创建并初始化PhoneWindow
   * 绑定ActivityThread、ActivityInfo，Application等重要的信息
   * 调用 `setContentView()`

3. PhoneWindow

   * 创建 DecorView 作为最顶部的 View
   * 在 DecorView 内加载系统的默认布局，如顶部栏ActionBar和 真正存放应用布局文件的 FrameLayout
   * 让 LayoutInflate 去实例化 activity 传进来的布局

4. LayoutInflate 

   * 把 layout.xml 文件交给 ResourcesImpl 和 AssetManager 去解析和加载资源，得到 XmlResourceParser

   * 处理 XmlResourceParser ，根据 tag 去 创建view

     > 由于创建view是通过类名去反射获取构造函数，花销较大。
     >
     > 因此系统会通过 map 去保存所有实例化过的View对应的构造函数，避免重复反射获取

