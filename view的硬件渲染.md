# 硬件渲染



## ViewRootImpl

### `setView()`

```java
public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView,
            int userId) {
        synchronized (this) {
            if (mView == null) {
                mView = view;

                ......

                if (view instanceof RootViewSurfaceTaker) {
                    mSurfaceHolderCallback =
                            ((RootViewSurfaceTaker)view).willYouTakeTheSurface();
                    if (mSurfaceHolderCallback != null) {
                        mSurfaceHolder = new TakenSurfaceHolder();
                        mSurfaceHolder.setFormat(PixelFormat.UNKNOWN);
                        mSurfaceHolder.addCallback(mSurfaceHolderCallback);
                    }
                }
			   .....	

                // If the application owns the surface, don't enable hardware acceleration
                if (mSurfaceHolder == null) {
                    // While this is supposed to enable only, it can effectively disable
                    // the acceleration too.
                    enableHardwareAcceleration(attrs);
                    final boolean useMTRenderer = MT_RENDERER_AVAILABLE
                            && mAttachInfo.mThreadedRenderer != null;
                    if (mUseMTRenderer != useMTRenderer) {
                        // Shouldn't be resizing, as it's done only in window setup,
                        // but end just in case.
                        endDragResizing();
                        mUseMTRenderer = useMTRenderer;
                    }
                }
			   ......	
            }
        }
    }
```



### `enableHardwareAcceleration()`



```java
    private void enableHardwareAcceleration(WindowManager.LayoutParams attrs) {
        // 先把硬件加速 相关属性置为false
        mAttachInfo.mHardwareAccelerated = false;
        mAttachInfo.mHardwareAccelerationRequested = false;

        // 当应用程序处于兼容模式时不要启用硬件加速 
        if (mTranslator != null) return;

        // 判断当前 window 是否打开硬件加速
        final boolean hardwareAccelerated =
                (attrs.flags & WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED) != 0;

        if (hardwareAccelerated) {
            // 目前R版本上默认view层级支持硬件渲染
            if (!ThreadedRenderer.isAvailable()) {
                return;
            }

            final boolean fakeHwAccelerated = (attrs.privateFlags &
                    WindowManager.LayoutParams.PRIVATE_FLAG_FAKE_HARDWARE_ACCELERATED) != 0;
            final boolean forceHwAccelerated = (attrs.privateFlags &
                    WindowManager.LayoutParams.PRIVATE_FLAG_FORCE_HARDWARE_ACCELERATED) != 0;

            if (fakeHwAccelerated) {
                // This is exclusively for the preview windows the window manager
                // shows for launching applications, so they will look more like
                // the app being launched.
                mAttachInfo.mHardwareAccelerationRequested = true;
            } else if (!ThreadedRenderer.sRendererDisabled 
                    || (ThreadedRenderer.sSystemRendererDisabled && forceHwAccelerated)) {
                if (mAttachInfo.mThreadedRenderer != null) {
                    mAttachInfo.mThreadedRenderer.destroy();
                }

                ......
				
                // 创建 ThreadedRenderer
                mAttachInfo.mThreadedRenderer = ThreadedRenderer.create(mContext, translucent,
                        attrs.getTitle().toString());
                mAttachInfo.mThreadedRenderer.setWideGamut(wideGamut);
                updateForceDarkMode();
                if (mAttachInfo.mThreadedRenderer != null) {
                    mAttachInfo.mHardwareAccelerated =
                            mAttachInfo.mHardwareAccelerationRequested = true;
                }
            }
        }
    }

```



## ThreadedRenderer

ThreadedRenderer 继承了 HardwareRenderer， create实际上会调用 HardwareRenderer 的构造方法

* `nCreateRootRenderNode ()` 创建native层的 RootRenderNode ，也就是所有 RenderNode 的根
* `RenderNode.adopt()` 根据native层的 RootRenderNode 创建 java层的 RootRenderNode
* `nCreateProxy()` 创建RenderNode的代理者
* ProcessInitializer的初始化graphicsstats服务

### `create()`

```java
public final class ThreadedRenderer extends HardwareRenderer {
    ......
	public static ThreadedRenderer create(Context context, boolean translucent, String name) {
        ThreadedRenderer renderer = null;
        if (isAvailable()) { // R版本上默认为true
            renderer = new ThreadedRenderer(context, translucent, name);
        }
        return renderer;
    }

	ThreadedRenderer(Context context, boolean translucent, String name) {
        super();
        setName(name);
        setOpaque(!translucent);

        final TypedArray a = context.obtainStyledAttributes(null, R.styleable.Lighting, 0, 0);
        mLightY = a.getDimension(R.styleable.Lighting_lightY, 0);
        mLightZ = a.getDimension(R.styleable.Lighting_lightZ, 0);
        mLightRadius = a.getDimension(R.styleable.Lighting_lightRadius, 0);
        float ambientShadowAlpha = a.getFloat(R.styleable.Lighting_ambientShadowAlpha, 0);
        float spotShadowAlpha = a.getFloat(R.styleable.Lighting_spotShadowAlpha, 0);
        a.recycle();
        setLightSourceAlpha(ambientShadowAlpha, spotShadowAlpha);
    }
    ......
}

public class HardwareRenderer {
	public HardwareRenderer() {
        // 创建native层的 RootRenderNode ，也就是所有 RenderNode 的根
        // 根据native层的 RootRenderNode 创建 java层的 RootRenderNode
        mRootNode = RenderNode.adopt(nCreateRootRenderNode());
        mRootNode.setClipToBounds(false);
        // 创建RenderNode的代理者
        mNativeProxy = nCreateProxy(!mOpaque, mIsWideGamut, mRootNode.mNativeRenderNode);
        if (mNativeProxy == 0) {
            throw new OutOfMemoryError("Unable to create hardware renderer");
        }
        Cleaner.create(this, new DestroyContextRunnable(mNativeProxy));
        // 初始化graphicsstats服务
        ProcessInitializer.sInstance.init(mNativeProxy);
    }
}
```



## ViewRootImpl

### `performTraversals()`

```java
    private void performTraversals() {
        
        ......

        /**
         * 开始 measure
         */
        if (mFirst || windowShouldResize || viewVisibilityChanged || cutoutChanged || params != null
                || mForceNextWindowRelayout) {
            mForceNextWindowRelayout = false;

            .....

            if (mSurfaceHolder != null) {
                mSurfaceHolder.mSurfaceLock.lock();
                mDrawingAllowed = true;
            }

            boolean hwInitialized = false;
            boolean dispatchApplyInsets = false;
            boolean hadSurface = mSurface.isValid();

            try {

                if (mAttachInfo.mThreadedRenderer != null) {
                    // relayoutWindow may decide to destroy mSurface. As that decision
                    // happens in WindowManager service, we need to be defensive here
                    // and stop using the surface in case it gets destroyed.
                    if (mAttachInfo.mThreadedRenderer.pause()) {
                        // Animations were running so we need to push a frame
                        // to resume them
                        mDirty.set(0, 0, mWidth, mHeight);
                    }
                    mChoreographer.mFrameInfo.addFlags(FrameInfo.FLAG_WINDOW_LAYOUT_CHANGED);
                }
                ......

                cutoutChanged = !mPendingDisplayCutout.equals(mAttachInfo.mDisplayCutout);
                surfaceSizeChanged = (relayoutResult
                        & WindowManagerGlobal.RELAYOUT_RES_SURFACE_RESIZED) != 0;
                final boolean alwaysConsumeSystemBarsChanged =
                        mPendingAlwaysConsumeSystemBars != mAttachInfo.mAlwaysConsumeSystemBars;
                final boolean colorModeChanged = hasColorModeChanged(lp.getColorMode());
                surfaceCreated = !hadSurface && mSurface.isValid();
                surfaceDestroyed = hadSurface && !mSurface.isValid();
                surfaceReplaced = (surfaceGenerationId != mSurface.getGenerationId())
                        && mSurface.isValid();
			   ......

                if (surfaceCreated) {
                    // If we are creating a new surface, then we need to
                    // completely redraw it.
                    mFullRedrawNeeded = true;
                    mPreviousTransparentRegion.setEmpty();

                    if (mAttachInfo.mThreadedRenderer != null) {
                        try {
                            // 初始化 ThreadedRenderer 
                            hwInitialized = mAttachInfo.mThreadedRenderer.initialize(mSurface);
                            if (hwInitialized && (host.mPrivateFlags
                                            & View.PFLAG_REQUEST_TRANSPARENT_REGIONS) == 0) {
                                // Don't pre-allocate if transparent regions
                                // are requested as they may not be needed
                                mAttachInfo.mThreadedRenderer.allocateBuffers();
                            }
                        } catch (OutOfResourcesException e) {
                            handleOutOfResourcesException(e);
                            return;
                        }
                    }
                    notifySurfaceCreated();
                } else if (surfaceDestroyed) {
                    // If the surface has been removed, then reset the scroll
                    // positions.
                    if (mLastScrolledFocus != null) {
                        mLastScrolledFocus.clear();
                    }
                    mScrollY = mCurScrollY = 0;
                    if (mView instanceof RootViewSurfaceTaker) {
                        ((RootViewSurfaceTaker) mView).onRootViewScrollYChanged(mCurScrollY);
                    }
                    if (mScroller != null) {
                        mScroller.abortAnimation();
                    }
                    // 销毁 ThreadedRenderer
                    if (mAttachInfo.mThreadedRenderer != null &&
                            mAttachInfo.mThreadedRenderer.isEnabled()) {
                        mAttachInfo.mThreadedRenderer.destroy();
                    }
                } else if ((surfaceReplaced || surfaceSizeChanged || windowRelayoutWasForced || colorModeChanged)
                        && mSurfaceHolder == null
                        && mAttachInfo.mThreadedRenderer != null
                        && mSurface.isValid()) {
                    mFullRedrawNeeded = true;
                    try {
                        // 更新 Surface
                        mAttachInfo.mThreadedRenderer.updateSurface(mSurface);
                    } catch (OutOfResourcesException e) {
                        handleOutOfResourcesException(e);
                        return;
                    }
                }

                if (!surfaceCreated && surfaceReplaced) {
                    notifySurfaceReplaced();
                }

                
            } catch (RemoteException e) {
            }

            ......

            final ThreadedRenderer threadedRenderer = mAttachInfo.mThreadedRenderer;
            if (threadedRenderer != null && threadedRenderer.isEnabled()) {
                if (hwInitialized
                        || mWidth != threadedRenderer.getWidth()
                        || mHeight != threadedRenderer.getHeight()
                        || mNeedsRendererSetup) {
                    threadedRenderer.setup(mWidth, mHeight, mAttachInfo,
                            mWindowAttributes.surfaceInsets);
                    mNeedsRendererSetup = false;
                }
            }

            ......
            /**
         	 * performMeasure()
         	 */
            ......
            
        } else {
            // Not the first pass and no window/insets/visibility change but the window
            // may have moved and we need check that and if so to update the left and right
            // in the attach info. We translate only the window frame since on window move
            // the window manager tells us only for the new frame but the insets are the
            // same and we do not want to translate them more than once.
            maybeHandleWindowMove(frame);
        }

        if (surfaceSizeChanged || surfaceReplaced || surfaceCreated || windowAttributesChanged) {
            updateBoundsLayer();
        }

        /**
         * 开始layout
         */
        ......

        /**
         * 开始 处理焦点
         */
        ......

        /**
         * 开始 draw
         */
        ......
    }

```



## ThreadedRenderer

初始化 ThreadedRenderer

### `initialize()`

```java
public final class ThreadedRenderer extends HardwareRenderer {
	boolean initialize(Surface surface) throws OutOfResourcesException {
        boolean status = !mInitialized;
        mInitialized = true;
        updateEnabledState(surface);
        setSurface(surface);
        return status;
    }

	public void setSurface(Surface surface) {
        // TODO: Do we ever pass a non-null but isValid() = false surface?
        // This is here to be super conservative for ViewRootImpl
        if (surface != null && surface.isValid()) {
            super.setSurface(surface);
        } else {
            super.setSurface(null);
        }
    }
}

public class HardwareRenderer {
	public void setSurface(@Nullable Surface surface, boolean discardBuffer) {
        if (surface != null && !surface.isValid()) {
            throw new IllegalArgumentException("Surface is invalid. surface.isValid() == false.");
        }
        // 
        nSetSurface(mNativeProxy, surface, discardBuffer);
    }
}
	
```



