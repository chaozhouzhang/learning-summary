示例代码：
```
public class SingleInstance {

    private static SingleInstance sInstance;

    private Context mContext;

    public SingleInstance(Context context) {
        mContext = context;
    }

    /**
     *
     * @param context
     * @return
     */
    public static SingleInstance getSingleInstance(Context context) {
        if (sInstance == null) {
            synchronized (SingleInstance.class) {
                if (sInstance == null) {
                    sInstance = new SingleInstance(context);
                }
            }
        }
        return sInstance;
    }

    public void hello(){
        System.out.println("hello");
    }
}
```

```
public class SingleInstanceActivity extends AppCompatActivity {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_single_instance);
        SingleInstance.getSingleInstance(this).hello();
    }

    public void finish(View view) {
        finish();
    }
}
```

Android Studio Profiler：


LeakCanary：
```
debugImplementation 'com.squareup.leakcanary:leakcanary-android:2.2'
```

```java
 14:39:56.726 : Installing AppWatcher
 14:40:13.696 : Watching instance of androidx.fragment.app.FragmentManagerViewModel (androidx.fragment.app.FragmentManagerViewModel received ViewModel#onCleared() callback) with key 36308936-e952-417f-b91d-9f96046ea5cb
 14:40:13.697 : Watching instance of leakcanary.internal.ViewModelClearedWatcher (leakcanary.internal.ViewModelClearedWatcher received ViewModel#onCleared() callback) with key ddaee485-5391-4469-a9f7-2a0e2be92168
 14:40:13.697 : Watching instance of androidx.lifecycle.ReportFragment (androidx.lifecycle.ReportFragment received Fragment#onDestroy() callback) with key a6736b89-2b06-4a09-b169-63a7ff3a5b7c
 14:40:13.697 : Watching instance of source.analysis.android29.singleinstance.SingleInstanceActivity (source.analysis.android29.singleinstance.SingleInstanceActivity received Activity#onDestroy() callback) with key f11d2e8b-7493-4cfa-ad90-bd6cee9988f6
 14:40:18.701 : Scheduling check for retained objects because found new object retained
 14:40:18.702 : Ignoring request to check for retained objects (found new object retained), already scheduled in -1ms
 14:40:18.703 : Ignoring request to check for retained objects (found new object retained), already scheduled in -1ms
 14:40:18.703 : Ignoring request to check for retained objects (found new object retained), already scheduled in -2ms
 14:40:18.954 : Rescheduling check for retained objects in 2000ms because found only 3 retained objects (< 5 while app visible)
 14:40:21.112 : Rescheduling check for retained objects in 2000ms because found only 3 retained objects (< 5 while app visible)
 14:40:23.263 : Rescheduling check for retained objects in 2000ms because found only 3 retained objects (< 5 while app visible)
 14:40:25.414 : Rescheduling check for retained objects in 2000ms because found only 3 retained objects (< 5 while app visible)
 14:40:27.571 : Rescheduling check for retained objects in 2000ms because found only 3 retained objects (< 5 while app visible)
 14:40:29.719 : Rescheduling check for retained objects in 2000ms because found only 3 retained objects (< 5 while app visible)
 14:40:31.875 : Rescheduling check for retained objects in 2000ms because found only 3 retained objects (< 5 while app visible)
 14:40:34.028 : Rescheduling check for retained objects in 2000ms because found only 3 retained objects (< 5 while app visible)
 14:40:36.186 : Rescheduling check for retained objects in 2000ms because found only 3 retained objects (< 5 while app visible)
 14:40:38.322 : Rescheduling check for retained objects in 2000ms because found only 3 retained objects (< 5 while app visible)
 14:40:40.475 : Rescheduling check for retained objects in 2000ms because found only 3 retained objects (< 5 while app visible)
 14:40:42.623 : Rescheduling check for retained objects in 2000ms because found only 3 retained objects (< 5 while app visible)
 14:40:44.778 : Rescheduling check for retained objects in 2000ms because found only 3 retained objects (< 5 while app visible)
 14:40:46.928 : Rescheduling check for retained objects in 2000ms because found only 3 retained objects (< 5 while app visible)
 14:40:49.082 : Rescheduling check for retained objects in 2000ms because found only 3 retained objects (< 5 while app visible)
 14:40:51.235 : Rescheduling check for retained objects in 2000ms because found only 3 retained objects (< 5 while app visible)
 14:40:53.390 : Rescheduling check for retained objects in 2000ms because found only 3 retained objects (< 5 while app visible)
 14:40:55.550 : Rescheduling check for retained objects in 2000ms because found only 3 retained objects (< 5 while app visible)
 14:40:57.706 : Rescheduling check for retained objects in 2000ms because found only 3 retained objects (< 5 while app visible)
 14:40:59.842 : Rescheduling check for retained objects in 2000ms because found only 3 retained objects (< 5 while app visible)
 14:41:02.007 : Rescheduling check for retained objects in 2000ms because found only 3 retained objects (< 5 while app visible)
 14:41:04.166 : Rescheduling check for retained objects in 2000ms because found only 3 retained objects (< 5 while app visible)
 14:41:06.325 : Rescheduling check for retained objects in 2000ms because found only 3 retained objects (< 5 while app visible)
 14:41:08.466 : Rescheduling check for retained objects in 2000ms because found only 3 retained objects (< 5 while app visible)
 14:41:09.155 : Ignoring request to check for retained objects (app became invisible), already scheduled in 1312ms
 14:41:10.658 : Rescheduling check for retained objects in 2000ms because found only 3 retained objects (< 5 while app visible)
 14:41:12.829 : Rescheduling check for retained objects in 2000ms because found only 3 retained objects (< 5 while app visible)
 14:41:14.961 : Check for retained objects found 3 objects, dumping the heap
 14:41:15.014 : WRITE_EXTERNAL_STORAGE permission not granted, ignoring
 14:41:16.619 : Analysis in progress, working on: PARSING_HEAP_DUMP
 14:41:18.410 : Analysis in progress, working on: EXTRACTING_METADATA
 14:41:18.660 : Analysis in progress, working on: FINDING_RETAINED_OBJECTS
 14:41:19.054 : Analysis in progress, working on: FINDING_PATHS_TO_RETAINED_OBJECTS
 14:41:24.917 : Analysis in progress, working on: FINDING_DOMINATORS
 14:41:25.611 : Found 3 retained objects
 14:41:25.611 : Analysis in progress, working on: COMPUTING_NATIVE_RETAINED_SIZE
 14:41:25.934 : Analysis in progress, working on: COMPUTING_RETAINED_SIZE
 14:41:25.991 : Analysis in progress, working on: BUILDING_LEAK_TRACES
 14:41:25.996 : Found 3 paths to retained objects, down to 1 after removing duplicated paths
 14:41:26.008 : Analysis in progress, working on: REPORTING_HEAP_ANALYSIS
 14:41:26.018 : ====================================
    HEAP ANALYSIS RESULT
    ====================================
    1 APPLICATION LEAKS
    
    References underlined with "~~~" are likely causes.
    Learn more at https://squ.re/leaks.
    
    250183 bytes retained by leaking objects
    Signature: 31385ce5b60c66d4aa43faac1d0e659ea14d6ec
    ┬───
    │ GC Root: System class
    │
    ├─ source.analysis.android29.singleinstance.SingleInstance class
    │    Leaking: NO (a class is never leaking)
    │    ↓ static SingleInstance.sInstance
    │                            ~~~~~~~~~
    ├─ source.analysis.android29.singleinstance.SingleInstance instance
    │    Leaking: UNKNOWN
    │    ↓ SingleInstance.mContext
    │                     ~~~~~~~~
    ╰→ source.analysis.android29.singleinstance.SingleInstanceActivity instance
    ​     Leaking: YES (ObjectWatcher was watching this because source.analysis.android29.singleinstance.SingleInstanceActivity received Activity#onDestroy() callback and Activity#mDestroyed is true)
    ​     key = f11d2e8b-7493-4cfa-ad90-bd6cee9988f6
    ​     watchDurationMillis = 61303
    ​     retainedDurationMillis = 56297
    ====================================
    0 LIBRARY LEAKS
    
    Library Leaks are leaks coming from the Android Framework or Google libraries.
    ====================================
    METADATA
    
    Please include this in bug reports and Stack Overflow questions.
    
    Build.VERSION.SDK_INT: 26
    Build.MANUFACTURER: HUAWEI
    LeakCanary version: 2.2
    App process name: source.analysis.android29
    Analysis duration: 9389 ms
    Heap dump file path: /data/user/0/source.analysis.android29/files/leakcanary/_14-41-15_027.hprof
    Heap dump timestamp: 1584600086008
    ====================================
```

修改：
```
public class SingleInstanceActivity extends AppCompatActivity {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_single_instance);
        SingleInstance.getSingleInstance(this.getApplicationContext()).hello();
    }

    public void finish(View view) {
        finish();
    }
}
```


