# View 准备绘制

## View.AttachInfo

~~~java
	final static class AttachInfo {

        interface Callbacks {
            void playSoundEffect(int effectId);
            boolean performHapticFeedback(int effectId, boolean always);
        }

        // 
        static class InvalidateInfo {

            InvalidateInfo() {
            }

            private static final int POOL_LIMIT = 10;

            private static final SynchronizedPool<InvalidateInfo> sPool =
                    new SynchronizedPool<InvalidateInfo>(POOL_LIMIT);

            View target;

            int left;
            int top;
            int right;
            int bottom;

            public static InvalidateInfo obtain() {
                InvalidateInfo instance = sPool.acquire();
                return (instance != null) ? instance : new InvalidateInfo();
            }

            public void recycle() {
                target = null;
                sPool.release(this);
            }
        }

        // 连通 IMS 和 WMS 等服务
        final IWindowSession mSession;

        final IWindow mWindow;

        final IBinder mWindowToken;

        Display mDisplay;

        final Callbacks mRootCallbacks;

        IWindowId mIWindowId;
        WindowId mWindowId;

        // Views
        View mRootView;

        IBinder mPanelParentWindowToken;

        boolean mHardwareAccelerated;
        boolean mHardwareAccelerationRequested;
        ThreadedRenderer mThreadedRenderer;
        List<RenderNode> mPendingAnimatingRenderNodes;

        /**
         * The state of the display to which the window is attached, as reported
         * by {@link Display#getState()}.  Note that the display state constants
         * declared by {@link Display} do not exactly line up with the screen state
         * constants declared by {@link View} (there are more display states than
         * screen states).
         */
        @UnsupportedAppUsage(maxTargetSdk = Build.VERSION_CODES.R, trackingBug = 170729553)
        int mDisplayState = Display.STATE_UNKNOWN;

        /**
         * Scale factor used by the compatibility mode
         */
        @UnsupportedAppUsage
        float mApplicationScale;

        /**
         * Indicates whether the application is in compatibility mode
         */
        @UnsupportedAppUsage
        boolean mScalingRequired;

        /**
         * Left position of this view's window
         */
        int mWindowLeft;

        /**
         * Top position of this view's window
         */
        int mWindowTop;

        /**
         * Indicates whether views need to use 32-bit drawing caches
         */
        boolean mUse32BitDrawingCache;

        /**
         * For windows that are full-screen but using insets to layout inside
         * of the screen decorations, these are the current insets for the
         * content of the window.
         */
        @UnsupportedAppUsage(maxTargetSdk = VERSION_CODES.Q,
                publicAlternatives = "Use {@link WindowInsets#getInsets(int)}")
        final Rect mContentInsets = new Rect();

        /**
         * For windows that are full-screen but using insets to layout inside
         * of the screen decorations, these are the current insets for the
         * actual visible parts of the window.
         */
        @UnsupportedAppUsage(maxTargetSdk = VERSION_CODES.Q,
                publicAlternatives = "Use {@link WindowInsets#getInsets(int)}")
        final Rect mVisibleInsets = new Rect();

        /**
         * For windows that are full-screen but using insets to layout inside
         * of the screen decorations, these are the current insets for the
         * stable system windows.
         */
        @UnsupportedAppUsage(maxTargetSdk = VERSION_CODES.Q,
                publicAlternatives = "Use {@link WindowInsets#getInsets(int)}")
        final Rect mStableInsets = new Rect();

        /**
         * Current caption insets to the display coordinate.
         */
        final Rect mCaptionInsets = new Rect();

        final DisplayCutout.ParcelableWrapper mDisplayCutout =
                new DisplayCutout.ParcelableWrapper(DisplayCutout.NO_CUTOUT);

        /**
         * In multi-window we force show the system bars. Because we don't want that the surface
         * size changes in this mode, we instead have a flag whether the system bars sizes should
         * always be consumed, so the app is treated like there are no virtual system bars at all.
         */
        boolean mAlwaysConsumeSystemBars;

        /**
         * The internal insets given by this window.  This value is
         * supplied by the client (through
         * {@link ViewTreeObserver.OnComputeInternalInsetsListener}) and will
         * be given to the window manager when changed to be used in laying
         * out windows behind it.
         */
        @UnsupportedAppUsage
        final ViewTreeObserver.InternalInsetsInfo mGivenInternalInsets
                = new ViewTreeObserver.InternalInsetsInfo();

        /**
         * Set to true when mGivenInternalInsets is non-empty.
         */
        boolean mHasNonEmptyGivenInternalInsets;

        /**
         * All views in the window's hierarchy that serve as scroll containers,
         * used to determine if the window can be resized or must be panned
         * to adjust for a soft input area.
         */
        @UnsupportedAppUsage
        final ArrayList<View> mScrollContainers = new ArrayList<View>();

        @UnsupportedAppUsage
        final KeyEvent.DispatcherState mKeyDispatchState
                = new KeyEvent.DispatcherState();

        /**
         * Indicates whether the view's window currently has the focus.
         */
        @UnsupportedAppUsage
        boolean mHasWindowFocus;

        /**
         * The current visibility of the window.
         */
        int mWindowVisibility;

        /**
         * Indicates the time at which drawing started to occur.
         */
        @UnsupportedAppUsage
        long mDrawingTime;

        /**
         * Indicates whether the view's window is currently in touch mode.
         */
        @UnsupportedAppUsage
        boolean mInTouchMode;

        /**
         * Indicates whether the view has requested unbuffered input dispatching for the current
         * event stream.
         */
        boolean mUnbufferedDispatchRequested;

        /**
         * Indicates that ViewAncestor should trigger a global layout change
         * the next time it performs a traversal
         */
        @UnsupportedAppUsage
        boolean mRecomputeGlobalAttributes;

        /**
         * Always report new attributes at next traversal.
         */
        boolean mForceReportNewAttributes;

        /**
         * Set during a traveral if any views want to keep the screen on.
         */
        @UnsupportedAppUsage
        boolean mKeepScreenOn;

        /**
         * Set during a traveral if the light center needs to be updated.
         */
        boolean mNeedsUpdateLightCenter;

        /**
         * Bitwise-or of all of the values that views have passed to setSystemUiVisibility().
         */
        int mSystemUiVisibility;

        /**
         * Hack to force certain system UI visibility flags to be cleared.
         */
        int mDisabledSystemUiVisibility;

        /**
         * True if a view in this hierarchy has an OnSystemUiVisibilityChangeListener
         * attached.
         */
        boolean mHasSystemUiListeners;

        /**
         * Set if the visibility of any views has changed.
         */
        @UnsupportedAppUsage
        boolean mViewVisibilityChanged;

        /**
         * Set to true if a view has been scrolled.
         */
        @UnsupportedAppUsage
        boolean mViewScrollChanged;

        /**
         * Set to true if a pointer event is currently being handled.
         */
        boolean mHandlingPointerEvent;

        /**
         * The offset of this view's window when it's on an embedded display that is re-parented
         * to another window.
         */
        final Point mLocationInParentDisplay = new Point();

        /**
         * The screen matrix of this view when it's on a {@link SurfaceControlViewHost} that is
         * embedded within a SurfaceView.
         */
        Matrix mScreenMatrixInEmbeddedHierarchy;

        /**
         * Global to the view hierarchy used as a temporary for dealing with
         * x/y points in the transparent region computations.
         */
        final int[] mTransparentLocation = new int[2];

        /**
         * Global to the view hierarchy used as a temporary for dealing with
         * x/y points in the ViewGroup.invalidateChild implementation.
         */
        final int[] mInvalidateChildLocation = new int[2];

        /**
         * Global to the view hierarchy used as a temporary for dealing with
         * computing absolute on-screen location.
         */
        final int[] mTmpLocation = new int[2];

        /**
         * Global to the view hierarchy used as a temporary for dealing with
         * x/y location when view is transformed.
         */
        final float[] mTmpTransformLocation = new float[2];

        /**
         * The view tree observer used to dispatch global events like
         * layout, pre-draw, touch mode change, etc.
         */
        @UnsupportedAppUsage
        final ViewTreeObserver mTreeObserver;

        /**
         * A Canvas used by the view hierarchy to perform bitmap caching.
         */
        Canvas mCanvas;

        /**
         * The view root impl.
         */
        final ViewRootImpl mViewRootImpl;

        /**
         * A Handler supplied by a view's {@link android.view.ViewRootImpl}. This
         * handler can be used to pump events in the UI events queue.
         */
        @UnsupportedAppUsage
        final Handler mHandler;

        /**
         * Temporary for use in computing invalidate rectangles while
         * calling up the hierarchy.
         */
        final Rect mTmpInvalRect = new Rect();

        /**
         * Temporary for use in computing hit areas with transformed views
         */
        final RectF mTmpTransformRect = new RectF();

        /**
         * Temporary for use in computing hit areas with transformed views
         */
        final RectF mTmpTransformRect1 = new RectF();

        /**
         * Temporary list of rectanges.
         */
        final List<RectF> mTmpRectList = new ArrayList<>();

        /**
         * Temporary for use in transforming invalidation rect
         */
        final Matrix mTmpMatrix = new Matrix();

        /**
         * Temporary for use in transforming invalidation rect
         */
        final Transformation mTmpTransformation = new Transformation();

        /**
         * Temporary for use in querying outlines from OutlineProviders
         */
        final Outline mTmpOutline = new Outline();

        /**
         * Temporary list for use in collecting focusable descendents of a view.
         */
        final ArrayList<View> mTempArrayList = new ArrayList<View>(24);

        /**
         * The id of the window for accessibility purposes.
         */
        int mAccessibilityWindowId = AccessibilityWindowInfo.UNDEFINED_WINDOW_ID;

        /**
         * Flags related to accessibility processing.
         *
         * @see AccessibilityNodeInfo#FLAG_INCLUDE_NOT_IMPORTANT_VIEWS
         * @see AccessibilityNodeInfo#FLAG_REPORT_VIEW_IDS
         */
        int mAccessibilityFetchFlags;

        /**
         * The drawable for highlighting accessibility focus.
         */
        Drawable mAccessibilityFocusDrawable;

        /**
         * The drawable for highlighting autofilled views.
         *
         * @see #isAutofilled()
         */
        Drawable mAutofilledDrawable;

        /**
         * Show where the margins, bounds and layout bounds are for each view.
         */
        boolean mDebugLayout = DisplayProperties.debug_layout().orElse(false);

        /**
         * Point used to compute visible regions.
         */
        final Point mPoint = new Point();

        /**
         * Used to track which View originated a requestLayout() call, used when
         * requestLayout() is called during layout.
         */
        View mViewRequestingLayout;

        /**
         * Used to track the identity of the current drag operation.
         */
        IBinder mDragToken;

        /**
         * The drag shadow surface for the current drag operation.
         */
        public Surface mDragSurface;


        /**
         * The view that currently has a tooltip displayed.
         */
        View mTooltipHost;

        /**
         * The initial structure has been reported so the view is ready to report updates.
         */
        boolean mReadyForContentCaptureUpdates;

        /**
         * Map(keyed by session) of content capture events that need to be notified after the view
         * hierarchy is traversed: value is either the view itself for appearead events, or its
         * autofill id for disappeared.
         */
        SparseArray<ArrayList<Object>> mContentCaptureEvents;

        /**
         * Cached reference to the {@link ContentCaptureManager}.
         */
        ContentCaptureManager mContentCaptureManager;

        /**
         * Listener used to fit content on window level.
         */
        OnContentApplyWindowInsetsListener mContentOnApplyWindowInsetsListener;

        /**
         * The leash token of this view's parent when it's in an embedded hierarchy that is
         * re-parented to another window.
         */
        IBinder mLeashedParentToken;

        /**
         * The accessibility view id of this view's parent when it's in an embedded
         * hierarchy that is re-parented to another window.
         */
        int mLeashedParentAccessibilityViewId;

        /**
         *
         */
        ScrollCaptureInternal mScrollCaptureInternal;

        /**
         * Creates a new set of attachment information with the specified
         * events handler and thread.
         *
         * @param handler the events handler the view must use
         */
        AttachInfo(IWindowSession session, IWindow window, Display display,
                ViewRootImpl viewRootImpl, Handler handler, Callbacks effectPlayer,
                Context context) {
            mSession = session;
            mWindow = window;
            mWindowToken = window.asBinder();
            mDisplay = display;
            mViewRootImpl = viewRootImpl;
            mHandler = handler;
            mRootCallbacks = effectPlayer;
            mTreeObserver = new ViewTreeObserver(context);
        }

        @Nullable
        ContentCaptureManager getContentCaptureManager(@NonNull Context context) {
            if (mContentCaptureManager != null) {
                return mContentCaptureManager;
            }
            mContentCaptureManager = context.getSystemService(ContentCaptureManager.class);
            return mContentCaptureManager;
        }

        void delayNotifyContentCaptureInsetsEvent(@NonNull Insets insets) {
            if (mContentCaptureManager == null) {
                return;
            }

            ArrayList<Object> events = ensureEvents(
                        mContentCaptureManager.getMainContentCaptureSession());
            events.add(insets);
        }

        private void delayNotifyContentCaptureEvent(@NonNull ContentCaptureSession session,
                @NonNull View view, boolean appeared) {
            ArrayList<Object> events = ensureEvents(session);
            events.add(appeared ? view : view.getAutofillId());
        }

        @NonNull
        private ArrayList<Object> ensureEvents(@NonNull ContentCaptureSession session) {
            if (mContentCaptureEvents == null) {
                // Most of the time there will be just one session, so intial capacity is 1
                mContentCaptureEvents = new SparseArray<>(1);
            }
            int sessionId = session.getId();
            // TODO: life would be much easier if we provided a MultiMap implementation somwhere...
            ArrayList<Object> events = mContentCaptureEvents.get(sessionId);
            if (events == null) {
                events = new ArrayList<>();
                mContentCaptureEvents.put(sessionId, events);
            }

            return events;
        }

        @Nullable
        ScrollCaptureInternal getScrollCaptureInternal() {
            if (mScrollCaptureInternal != null) {
                mScrollCaptureInternal = new ScrollCaptureInternal();
            }
            return mScrollCaptureInternal;
        }
    }
~~~



## ViewRootimpl

### `setView()`

```java
	public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView,
            int userId) {
        synchronized (this) {
            if (mView == null) {
                mView = view;

                /**
                 * 1、更新 AttachInfo 的 displayState
                 */
                mAttachInfo.mDisplayState = mDisplay.getState();
                /**
                 * 2、把handle 注册到 DisplayManager 中监听回调
                 */
                mDisplayManager.registerDisplayListener(mDisplayListener, mHandler);

                ......

                /**
                 * 3、从Display中获取当前屏幕的兼容信息，并获取坐标转化器。
                 * 最后把这些信息都存放到AttachInfo中的mApplicationScale，并在AttachInfo记录根部View(DecorView)
                 */
                CompatibilityInfo compatibilityInfo =
                        mDisplay.getDisplayAdjustments().getCompatibilityInfo();
                mTranslator = compatibilityInfo.getTranslator();
                ......
                /**
                 * 4、如果获取到坐标转化器，说明此时需要屏幕中的内容需要进行缩放。
                 */
                if (mTranslator != null) {
                    mSurface.setCompatibilityTranslator(mTranslator);
                    restore = true;
                    attrs.backup();
                    mTranslator.translateWindowLayout(attrs);
                }
                ......
			   /**
                 * 5、最后把这些信息都存放到AttachInfo中的mApplicationScale，
                 * 并在AttachInfo记录根部View(DecorView)
                 */
                mSoftInputMode = attrs.softInputMode;
                mWindowAttributesChanged = true;
                mAttachInfo.mRootView = view;
                mAttachInfo.mScalingRequired = mTranslator != null;
                mAttachInfo.mApplicationScale =
                        mTranslator == null ? 1.0f : mTranslator.applicationScale;
                if (panelParentView != null) {
                    mAttachInfo.mPanelParentWindowToken
                            = panelParentView.getApplicationWindowToken();
                }
                mAdded = true;
                int res; /* = WindowManagerImpl.ADD_OKAY; */

                /**
                 * 5. 先请求布局
                 */
                requestLayout();
                InputChannel inputChannel = null;
                if ((mWindowAttributes.inputFeatures
                        & WindowManager.LayoutParams.INPUT_FEATURE_NO_INPUT_CHANNEL) == 0) {
                    /**
                 	 * 6. 创建 InputChannel 去监听点击事件
                 	 */
                    inputChannel = new InputChannel();
                }
                mForceDecorViewVisibility = (mWindowAttributes.privateFlags
                        & PRIVATE_FLAG_FORCE_DECOR_VIEW_VISIBILITY) != 0;
                try {
                    mOrigWindowType = mWindowAttributes.type;
                    mAttachInfo.mRecomputeGlobalAttributes = true;
                    collectViewAttributes();
                    adjustLayoutParamsForCompatibility(mWindowAttributes);
                    /**
                 	 * 6. 添加window到WMS
                 	 */
                    res = mWindowSession.addToDisplayAsUser(mWindow, mSeq, mWindowAttributes,
                            getHostVisibility(), mDisplay.getDisplayId(), userId, mTmpFrame,
                            mAttachInfo.mContentInsets, mAttachInfo.mStableInsets,
                            mAttachInfo.mDisplayCutout, inputChannel,
                            mTempInsets, mTempControls);
                    setFrame(mTmpFrame);
                } 
                ......
            }
        }
    }
```



### `requestLayout()`

```java
	public void requestLayout() {
        // 如果此时正在 onLayout(), 那么就不会调用scheduleTraversals
        if (!mHandlingLayoutInLayoutRequest) {
            checkThread();
            mLayoutRequested = true;
            scheduleTraversals();
        }
    }

	void checkThread() {
        // 判断当前进程和ViewRooyImpl创建进程是不是同一个，
        if (mThread != Thread.currentThread()) {
            throw new CalledFromWrongThreadException(
                    "Only the original thread that created a view hierarchy can touch its views.");
        }
    }
```



### `scheduleTraversals()`

通过mChoreographer的postCallback方法发送一个CALLBACK_TRAVERSAL

```java
	void scheduleTraversals() {
        if (!mTraversalScheduled) {
            mTraversalScheduled = true;
            mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
            // 发送一个 CALLBACK_TRAVERSAL 用来监听Vsync信号，
            // 当 Vsync 时，执行 TraversalRunnable.runnable(),
            mChoreographer.postCallback(
                    Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
            notifyRendererOfFramePending();
            pokeDrawLockIfNeeded();
        }
    }

	final class TraversalRunnable implements Runnable {
        @Override
        public void run() {
            doTraversal();
        }
    }
```



### `doTraversal()`

```java
	void doTraversal() {
        if (mTraversalScheduled) {
            mTraversalScheduled = false;
            mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);

            if (mProfile) {
                Debug.startMethodTracing("ViewAncestor");
            }
			
            // 整个View树的绘制核心，performTraversals
            performTraversals();

            if (mProfile) {
                Debug.stopMethodTracing();
                mProfile = false;
            }
        }
    }
```



### `performTraversals()`

在 `onMeasure()` 之前

* 在 `setView()` 的时候 ，已经通过 `WindowSession.addToDisplayAsUser()` 去初步计算window的各个间距屏幕数值
* `dispatchAttachedToWindow()`
* `dispatchApplyWindowInsets()`
* `relayoutWindow()` 计算窗体的大小以及位置
* 准备硬件渲染

1. ViewRootImpl 首次绘制
2. ViewRootImpl 非首次绘制

```java
	    Rect frame = mWinFrame;
        if (mFirst) {
            /**
             * 1、ViewRootImpl 首次绘制
             */
            mFullRedrawNeeded = true;
            mLayoutRequested = true;

            final Configuration config = mContext.getResources().getConfiguration();
            if (shouldUseDisplaySize(lp)) {
                // 如果是 System的ui，如statusbar，音量等，
                // 就会设置desiredWindowWidth和desiredWindowHeight为屏幕的宽高
                Point size = new Point();
                mDisplay.getRealSize(size);
                desiredWindowWidth = size.x;
                desiredWindowHeight = size.y;
            } else if (lp.width == ViewGroup.LayoutParams.WRAP_CONTENT
                    || lp.height == ViewGroup.LayoutParams.WRAP_CONTENT) {
                // 对于设置 WRAP_CONTENT 的悬浮窗来说，高度还不确定，
                // 就会设置desiredWindowWidth和desiredWindowHeight为当前窗口可用宽高
                desiredWindowWidth = dipToPx(config.screenWidthDp);
                desiredWindowHeight = dipToPx(config.screenHeightDp);
            } else {
                // 设置desiredWindowWidth和desiredWindowHeight为
                // setView()时已经通过WindowSession.addToDisplayAsUser()计算好的屏幕宽高
                desiredWindowWidth = frame.width();
                desiredWindowHeight = frame.height();
            }

            ......
            // 设置View数的渲染方向
            if (mViewLayoutDirectionInitial == View.LAYOUT_DIRECTION_INHERIT) {
                host.setLayoutDirection(config.getLayoutDirection());
            }
            // 从DecorView开始，往下分发 onAttatchToWindow 和 onApplyInsets 回调
            host.dispatchAttachedToWindow(mAttachInfo, 0);
            mAttachInfo.mTreeObserver.dispatchOnWindowAttachedChange(true);
            dispatchApplyInsets(host);
        } else {
            /**
             * 2、ViewRootImpl 非首次绘制
             * 设置desiredWindowWidth和desiredWindowHeight为
             * setView()时已经通过WindowSession.addToDisplayAsUser()计算好的屏幕宽高
             */
            desiredWindowWidth = frame.width();
            desiredWindowHeight = frame.height();
            if (desiredWindowWidth != mWidth || desiredWindowHeight != mHeight) {
                if (DEBUG_ORIENTATION) Log.v(mTag, "View " + host + " resized to: " + frame);
                mFullRedrawNeeded = true;
                mLayoutRequested = true;
                windowSizeMayChange = true;
            }
        }
```

对于一般的界面，都会使用WindowSession.addToDisplayAsUser()计算好的屏幕宽高设置给desiredWindowWidth和desiredWindowHeight。

在实际的计算逻辑在 `DisplayPolicy.getLayoutHint()`

#### `DisplayPolicy.getLayoutHint()`

![img](https://img-blog.csdnimg.cn/20190628094221622.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Z2aWNjYw==,size_16,color_FFFFFF,t_70)

1. overscanScreen区域，包含了overscan区域，相当于整个屏幕大小
2. RestrictedOverscanScreen区域，不包含导航栏
3. RestrictedScreen区域，不包含屏幕上的overscan区域，不包含导航条
4. UnrestrictedScreen区域，不包含overscan区域，包含状态条和导航栏
5. StableFullScreen区域，包含状态栏，不包含导航栏
6. Decor区域，不包含状态栏，不包含导航栏，包含输入法的区域
7. Current区域，不包含状态栏，不包含导航栏，不包含输入法的区域



```java
boolean getLayoutHint(LayoutParams attrs, WindowToken windowToken, Rect outFrame,
            Rect outContentInsets, Rect outStableInsets,
            DisplayCutout.ParcelableWrapper outDisplayCutout) {
        final int fl = PolicyControl.getWindowFlags(null, attrs);
        final int pfl = attrs.privateFlags;
        final int requestedSysUiVis = PolicyControl.getSystemUiVisibility(null, attrs);
        final int sysUiVis = requestedSysUiVis | getImpliedSysUiFlagsForLayout(attrs);

        final boolean layoutInScreen = (fl & FLAG_LAYOUT_IN_SCREEN) != 0;
        final boolean layoutInScreenAndInsetDecor = layoutInScreen
                && (fl & FLAG_LAYOUT_INSET_DECOR) != 0;
        final boolean screenDecor = (pfl & PRIVATE_FLAG_IS_SCREEN_DECOR) != 0;

        final boolean isFixedRotationTransforming =
                windowToken != null && windowToken.isFixedRotationTransforming();
        final ActivityRecord activity = windowToken != null ? windowToken.asActivityRecord() : null;
        final Task task = activity != null ? activity.getTask() : null;
        final Rect taskBounds = isFixedRotationTransforming
                // Use token (activity) bounds if it is rotated because its task is not rotated.
                ? windowToken.getBounds()
                : (task != null ? task.getBounds() : null);
        final DisplayFrames displayFrames = isFixedRotationTransforming
                ? windowToken.getFixedRotationTransformDisplayFrames()
                : mDisplayContent.mDisplayFrames;

        if (layoutInScreenAndInsetDecor && !screenDecor) {
            if ((sysUiVis & SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION) != 0
                    || (attrs.getFitInsetsTypes() & Type.navigationBars()) == 0) {
                outFrame.set(displayFrames.mUnrestricted);
            } else {
                outFrame.set(displayFrames.mRestricted);
            }

            final boolean isFloatingTask = task != null && task.isFloating();
            final Rect sf = isFloatingTask ? null : displayFrames.mStable;
            final Rect cf;
            if (isFloatingTask) {
                cf = null;
            } else if ((sysUiVis & View.SYSTEM_UI_FLAG_LAYOUT_STABLE) != 0) {
                if ((fl & FLAG_FULLSCREEN) != 0) {
                    cf = displayFrames.mStableFullscreen;
                } else {
                    cf = displayFrames.mStable;
                }
            } else if ((fl & FLAG_FULLSCREEN) != 0) {
                cf = displayFrames.mUnrestricted;
            } else {
                cf = displayFrames.mCurrent;
            }

            if (taskBounds != null) {
                outFrame.intersect(taskBounds);
            }
            InsetUtils.insetsBetweenFrames(outFrame, cf, outContentInsets);
            InsetUtils.insetsBetweenFrames(outFrame, sf, outStableInsets);
            outDisplayCutout.set(displayFrames.mDisplayCutout.calculateRelativeTo(outFrame)
                    .getDisplayCutout());
            return mForceShowSystemBars;
        } else {
            if (layoutInScreen) {
                outFrame.set(displayFrames.mUnrestricted);
            } else {
                outFrame.set(displayFrames.mStable);
            }
            if (taskBounds != null) {
                outFrame.intersect(taskBounds);
            }

            outContentInsets.setEmpty();
            outStableInsets.setEmpty();
            outDisplayCutout.set(DisplayCutout.NO_CUTOUT);
            return mForceShowSystemBars;
        }
    }
```



#### `dispatchAttachedToWindow()`

遍历绑定在ViewGroup中所有子View的dispatchAttachedToWindow方法

#### `dispatchApplyWindowInsets()`

#### `dispatchWindowVisibilityChanged()`



## 总结





