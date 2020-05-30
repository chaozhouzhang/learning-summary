# StartActivity流程概览



## 1、在APP进程发起startActivity
在Activity中的startActivityForResult()调用Instrumentation的execStartActivity()：
```java
Instrumentation.ActivityResult ar =
    mInstrumentation.execStartActivity(
        this, mMainThread.getApplicationThread(), mToken, this,
        intent, requestCode, options);
```
在Instrumentation的execStartActivity()调用ATMS的startActivity()：
```java
int result = ActivityTaskManager.getService()
    .startActivity(whoThread, who.getBasePackageName(), intent,
            intent.resolveTypeIfNeeded(who.getContentResolver()),
            token, target != null ? target.mEmbeddedID : null,
            requestCode, 0, null, options);
```
其中ATMS实例是从AMS中获得的：
```java
/** @hide */
public static IActivityTaskManager getService() {
    return IActivityTaskManagerSingleton.get();
}

@UnsupportedAppUsage(trackingBug = 129726065)
private static final Singleton<IActivityTaskManager> IActivityTaskManagerSingleton =
        new Singleton<IActivityTaskManager>() {
            @Override
            protected IActivityTaskManager create() {
                final IBinder b = ServiceManager.getService(Context.ACTIVITY_TASK_SERVICE);
                return IActivityTaskManager.Stub.asInterface(b);
            }
        };
```

## 2、在ATMS进程发起startActivity
在ATMS的startActivityAsUser()调用ActivityStarter的execute()：
```
// TODO: Switch to user app stacks here.
return getActivityStartController().obtainStarter(intent, "startActivityAsUser")
        .setCaller(caller)
        .setCallingPackage(callingPackage)
        .setResolvedType(resolvedType)
        .setResultTo(resultTo)
        .setResultWho(resultWho)
        .setRequestCode(requestCode)
        .setStartFlags(startFlags)
        .setProfilerInfo(profilerInfo)
        .setActivityOptions(bOptions)
        .setMayWait(userId)
        .execute();

```
在ActivityStarter的startActivityUnchecked()调用RootActivityContainer的resumeFocusedStacksTopActivities()：
```
mRootActivityContainer.resumeFocusedStacksTopActivities();
```
在RootActivityContainer的resumeFocusedStacksTopActivities()调用ActivityStack
的resumeTopActivityUncheckedLocked()：
```
boolean result = false;
if (targetStack != null && (targetStack.isTopStackOnDisplay()
        || getTopDisplayFocusedStack() == targetStack)) {
    result = targetStack.resumeTopActivityUncheckedLocked(target, targetOptions);
}
```
## 2.1、onPause()
在ActivityStack的startPausingLocked()调用ClientLifecycleManager的scheduleTransaction()，此处操作对应生命周期的onPause()流程：

```
mService.getLifecycleManager().scheduleTransaction(prev.app.getThread(),
        prev.appToken, PauseActivityItem.obtain(prev.finishing, userLeaving,
                prev.configChangeFlags, pauseImmediately));
```
## 2.2、onCreate()
或者先在ActivityStack的resumeTopActivityInnerLocked()调用ActivityStackSupervisor的startSpecificActivityLocked()：
```
mStackSupervisor.startSpecificActivityLocked(next, true, false);
```
然后在ActivityStackSupervisor的realStartActivityLocked()调用ClientLifecycleManager的scheduleTransaction()，此处操作对应生命周期的onCreate()流程：
```
// Schedule transaction.
mService.getLifecycleManager().scheduleTransaction(clientTransaction);
```
## 2.3、安排事务到APP进程
在ClientLifecycleManager的scheduleTransaction()中调用ClientTransaction的schedule()：
```
void scheduleTransaction(ClientTransaction transaction) throws RemoteException {
    final IApplicationThread client = transaction.getClient();
    transaction.schedule();
}
```

## 3、回到APP进程

在ClientTransaction的schedule()中调用ActivityThread内部类ApplicationThread的scheduleTransaction()：
```
public void schedule() throws RemoteException {
    mClient.scheduleTransaction(this);
}
```
在ApplicationThread的scheduleTransaction()调用了ActivityThread也就是其父类ClientTransactionHandler的scheduleTransaction()：
```
@Override
public void scheduleTransaction(ClientTransaction transaction) throws RemoteException {
    ActivityThread.this.scheduleTransaction(transaction);
}
```
在ClientTransactionHandler的scheduleTransaction()，通过H发送了EXECUTE_TRANSACTION消息：
```
/** Prepare and schedule transaction for execution. */
void scheduleTransaction(ClientTransaction transaction) {
    transaction.preExecute(this);
    sendMessage(ActivityThread.H.EXECUTE_TRANSACTION, transaction);
}
```
在H中接收EXECUTE_TRANSACTION消息，并调用了TransactionExecutor的execute()：
```
case EXECUTE_TRANSACTION:
    final ClientTransaction transaction = (ClientTransaction) msg.obj;
    mTransactionExecutor.execute(transaction);
    if (isSystem()) {
        // Client transactions inside system process are recycled on the client side
        // instead of ClientLifecycleManager to avoid being cleared before this
        // message is handled.
        transaction.recycle();
    }
    // TODO(lifecycler): Recycle locally scheduled transactions.
    break;
```
## 3.1、onPause()
如果对应的是onPause()流程，将会在TransactionExecutor的execute()调用executeLifecycleState()，并继续调用ActivityLifecycleItem也就是子类PauseActivityItem的execute()：
```
lifecycleItem.execute(mTransactionHandler, token, mPendingActions);
```

在PauseActivityItem的execute()调用ClientTransactionHandler也就是子类ActivityThread的handlePauseActivity()：
```
@Override
public void execute(ClientTransactionHandler client, IBinder token,
        PendingTransactionActions pendingActions) {
    Trace.traceBegin(TRACE_TAG_ACTIVITY_MANAGER, "activityPause");
    client.handlePauseActivity(token, mFinished, mUserLeaving, mConfigChanges, pendingActions,
            "PAUSE_ACTIVITY_ITEM");
    Trace.traceEnd(TRACE_TAG_ACTIVITY_MANAGER);
}
```

而后将继续在ActivityThread的performPauseActivityIfNeeded()调用Instrumentation的callActivityOnPause()：
```
mInstrumentation.callActivityOnPause(r.activity);
```
而后调用Activity的performPause()：
```
public void callActivityOnPause(Activity activity) {
    activity.performPause();
}
```
最后调用Activity的onPause()：
```
final void performPause() {
    dispatchActivityPrePaused();
    mDoReportFullyDrawn = false;
    mFragments.dispatchPause();
    mCalled = false;
    onPause();
    writeEventLog(LOG_AM_ON_PAUSE_CALLED, "performPause");
    mResumed = false;
    if (!mCalled && getApplicationInfo().targetSdkVersion
            >= android.os.Build.VERSION_CODES.GINGERBREAD) {
        throw new SuperNotCalledException(
                "Activity " + mComponent.toShortString() +
                " did not call through to super.onPause()");
    }
    dispatchActivityPostPaused();
}
```


## 3.2、onCreate()
如果对应的是onCreate()流程，将会在TransactionExecutor的execute()调用executeCallbacks()，并继续调用ClientTransactionItem
也就是子类LaunchActivityItem的execute()：
```
item.execute(mTransactionHandler, token, mPendingActions);
```

在LaunchActivityItem的execute()调用ClientTransactionHandler也就是子类ActivityThread的handleLaunchActivity()：
```
@Override
public void execute(ClientTransactionHandler client, IBinder token,
        PendingTransactionActions pendingActions) {
    Trace.traceBegin(TRACE_TAG_ACTIVITY_MANAGER, "activityStart");
    ActivityClientRecord r = new ActivityClientRecord(token, mIntent, mIdent, mInfo,
            mOverrideConfig, mCompatInfo, mReferrer, mVoiceInteractor, mState, mPersistentState,
            mPendingResults, mPendingNewIntents, mIsForward,
            mProfilerInfo, client, mAssistToken);
    client.handleLaunchActivity(r, pendingActions, null /* customIntent */);
    Trace.traceEnd(TRACE_TAG_ACTIVITY_MANAGER);
}
```

而后将继续在ActivityThread的performLaunchActivity()调用Instrumentation的newActivity()使用反射来创建Activity实例：

```
java.lang.ClassLoader cl = appContext.getClassLoader();
activity = mInstrumentation.newActivity(
            cl, component.getClassName(), r.intent);
```

```
public Activity newActivity(ClassLoader cl, String className,
        Intent intent)
        throws InstantiationException, IllegalAccessException,
        ClassNotFoundException {
    String pkg = intent != null && intent.getComponent() != null
            ? intent.getComponent().getPackageName() : null;
    return getFactory(pkg).instantiateActivity(cl, className, intent);
}
```

```
public @NonNull Activity instantiateActivity(@NonNull ClassLoader cl, @NonNull String className,
        @Nullable Intent intent)
        throws InstantiationException, IllegalAccessException, ClassNotFoundException {
    return (Activity) cl.loadClass(className).newInstance();
}
```
而后继续在ActivityThread的performLaunchActivity()调用Activity的attach()：
```
activity.attach(appContext, this, getInstrumentation(), r.token,
        r.ident, app, r.intent, r.activityInfo, title, r.parent,
        r.embeddedID, r.lastNonConfigurationInstances, config,
        r.referrer, r.voiceInteractor, window, r.configCallback,
        r.assistToken);
```

创建好Activity实例后，将会调用Instrumentation的callActivityOnCreate()：
```
if (r.isPersistable()) {
    mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
} else {
    mInstrumentation.callActivityOnCreate(activity, r.state);
}
```


而后调用Activity的performCreate()：
```
public void callActivityOnCreate(Activity activity, Bundle icicle,
        PersistableBundle persistentState) {
    prePerformCreate(activity);
    activity.performCreate(icicle, persistentState);
    postPerformCreate(activity);
}
```
最后调用Activity的onCreate()：
```
@UnsupportedAppUsage
final void performCreate(Bundle icicle, PersistableBundle persistentState) {
    dispatchActivityPreCreated(icicle);
    mCanEnterPictureInPicture = true;
    restoreHasCurrentPermissionRequest(icicle);
    if (persistentState != null) {
        onCreate(icicle, persistentState);
    } else {
        onCreate(icicle);
    }
    writeEventLog(LOG_AM_ON_CREATE_CALLED, "performCreate");
    mActivityTransitionState.readState(icicle);

    mVisibleFromClient = !mWindow.getWindowStyle().getBoolean(
            com.android.internal.R.styleable.Window_windowNoDisplay, false);
    mFragments.dispatchActivityCreated();
    mActivityTransitionState.setEnterActivityOptions(this, getActivityOptions());
    dispatchActivityPostCreated(icicle);
}
```



