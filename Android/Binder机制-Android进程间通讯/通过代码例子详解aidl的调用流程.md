

# 使用aidl进行跨进程通信

![](https://github.com/chaozhouzhang/learning-summary/blob/master/Android/Binder%E6%9C%BA%E5%88%B6-Android%E8%BF%9B%E7%A8%8B%E9%97%B4%E9%80%9A%E8%AE%AF/aidl-uml.png?raw=true)

## 1、首先创建aidl文件，并声明接口：
```
// IRemoteService.aidl
package source.analysis.android29.aidl;
// Declare any non-default types here with import statements
parcelable User;

interface IRemoteService {
    int getValue();
    User getUser(in String username);
}
```
## 2、创建接口需要的Parcelable实体类：
```
public class User implements Parcelable {
    private int age;
    private String nickname;
    private int gender;

    public int getAge() {
        return age;
    }

    public User setAge(int age) {
        this.age = age;
        return this;
    }

    public String getNickname() {
        return nickname;
    }

    public User setNickname(String nickname) {
        this.nickname = nickname;
        return this;
    }

    public int getGender() {
        return gender;
    }
    
    public User setGender(int gender) {
        this.gender = gender;
        return this;
    }

    public User() {
    }

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeInt(this.age);
        dest.writeString(this.nickname);
        dest.writeInt(this.gender);
    }

    protected User(Parcel in) {
        this.age = in.readInt();
        this.nickname = in.readString();
        this.gender = in.readInt();
    }

    public static final Creator<User> CREATOR = new Creator<User>() {
        @Override
        public User createFromParcel(Parcel source) {
            return new User(source);
        }

        @Override
        public User[] newArray(int size) {
            return new User[size];
        }
    };
}
```
## 3、编译后自动生成接口和空的实体类：
IRemoteService
```
/*
 * This file is auto-generated.  DO NOT MODIFY.
 */
package source.analysis.android29.aidl;
// Declare any non-default types here with import statements

public interface IRemoteService extends android.os.IInterface {
    /**
     * Default implementation for IRemoteService.
     */
    public static class Default implements source.analysis.android29.aidl.IRemoteService {
        @Override
        public int getValue() throws android.os.RemoteException {
            return 0;
        }

        @Override
        public source.analysis.android29.aidl.User getUser(java.lang.String username) throws android.os.RemoteException {
            return null;
        }

        @Override
        public android.os.IBinder asBinder() {
            return null;
        }
    }

    /**
     * Local-side IPC implementation stub class.
     */
    public static abstract class Stub extends android.os.Binder implements source.analysis.android29.aidl.IRemoteService {
        private static final java.lang.String DESCRIPTOR = "source.analysis.android29.aidl.IRemoteService";

        /**
         * Construct the stub at attach it to the interface.
         */
        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }

        /**
         * Cast an IBinder object into an source.analysis.android29.aidl.IRemoteService interface,
         * generating a proxy if needed.
         */
        public static source.analysis.android29.aidl.IRemoteService asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof source.analysis.android29.aidl.IRemoteService))) {
                return ((source.analysis.android29.aidl.IRemoteService) iin);
            }
            return new source.analysis.android29.aidl.IRemoteService.Stub.Proxy(obj);
        }

        @Override
        public android.os.IBinder asBinder() {
            return this;
        }

        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
            java.lang.String descriptor = DESCRIPTOR;
            switch (code) {
                case INTERFACE_TRANSACTION: {
                    reply.writeString(descriptor);
                    return true;
                }
                case TRANSACTION_getValue: {
                    data.enforceInterface(descriptor);
                    int _result = this.getValue();
                    reply.writeNoException();
                    reply.writeInt(_result);
                    return true;
                }
                case TRANSACTION_getUser: {
                    data.enforceInterface(descriptor);
                    java.lang.String _arg0;
                    _arg0 = data.readString();
                    source.analysis.android29.aidl.User _result = this.getUser(_arg0);
                    reply.writeNoException();
                    if ((_result != null)) {
                        reply.writeInt(1);
                        _result.writeToParcel(reply, android.os.Parcelable.PARCELABLE_WRITE_RETURN_VALUE);
                    } else {
                        reply.writeInt(0);
                    }
                    return true;
                }
                default: {
                    return super.onTransact(code, data, reply, flags);
                }
            }
        }

        private static class Proxy implements source.analysis.android29.aidl.IRemoteService {
            private android.os.IBinder mRemote;

            Proxy(android.os.IBinder remote) {
                mRemote = remote;
            }

            @Override
            public android.os.IBinder asBinder() {
                return mRemote;
            }

            public java.lang.String getInterfaceDescriptor() {
                return DESCRIPTOR;
            }

            @Override
            public int getValue() throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                int _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    boolean _status = mRemote.transact(Stub.TRANSACTION_getValue, _data, _reply, 0);
                    if (!_status && getDefaultImpl() != null) {
                        return getDefaultImpl().getValue();
                    }
                    _reply.readException();
                    _result = _reply.readInt();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }

            @Override
            public source.analysis.android29.aidl.User getUser(java.lang.String username) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                source.analysis.android29.aidl.User _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    _data.writeString(username);
                    boolean _status = mRemote.transact(Stub.TRANSACTION_getUser, _data, _reply, 0);
                    if (!_status && getDefaultImpl() != null) {
                        return getDefaultImpl().getUser(username);
                    }
                    _reply.readException();
                    if ((0 != _reply.readInt())) {
                        _result = source.analysis.android29.aidl.User.CREATOR.createFromParcel(_reply);
                    } else {
                        _result = null;
                    }
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }

            public static source.analysis.android29.aidl.IRemoteService sDefaultImpl;
        }

        static final int TRANSACTION_getValue = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
        static final int TRANSACTION_getUser = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);

        public static boolean setDefaultImpl(source.analysis.android29.aidl.IRemoteService impl) {
            if (Stub.Proxy.sDefaultImpl == null && impl != null) {
                Stub.Proxy.sDefaultImpl = impl;
                return true;
            }
            return false;
        }

        public static source.analysis.android29.aidl.IRemoteService getDefaultImpl() {
            return Stub.Proxy.sDefaultImpl;
        }
    }

    public int getValue() throws android.os.RemoteException;

    public source.analysis.android29.aidl.User getUser(java.lang.String username) throws android.os.RemoteException;
}
```

User.java
```
// This file is intentionally left blank as placeholder for parcel declaration.
```
## 4、实现接口方法
RemoteServiceImpl
```
public class RemoteServiceImpl extends IRemoteService.Stub {
    @Override
    public int getValue() throws RemoteException {
        return 100;
    }

    @Override
    public User getUser(String username) throws RemoteException {
        User user = new User();
        user.setAge(18);
        user.setGender(0);
        user.setNickname("昵称");
        return user;
    }
}
```
## 5、创建独立进程服务
RemoteService
```
public class RemoteService extends Service {
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return new RemoteServiceImpl();
    }
}
```
在AndroidManifest.xml中注册独立进程服务：
```
<service android:name=".aidl.RemoteService" android:process=":remote"/>
```
## 6、绑定独立进程服务
绑定独立进程服务
```
Intent intent = new Intent(this, RemoteService.class);
bindService(intent, this, Context.BIND_AUTO_CREATE);
```

## 7、获取独立进程服务代理
```
public class AidlActivity extends AppCompatActivity implements ServiceConnection {

/**
 * Called when the connection with the service is established
 */
@Override
public void onServiceConnected(ComponentName name, IBinder service) {
    // Following the example above for an AIDL interface,
    // this gets an instance of the IRemoteInterface, which we can use to call on the service
    iRemoteService = IRemoteService.Stub.asInterface(service);
}

/**
 * Called when the connection with the service disconnects unexpectedly
 */
@Override
public void onServiceDisconnected(ComponentName name) {
    Log.e(TAG, "Service has unexpectedly disconnected");
    iRemoteService = null;
}
```

## 8、通过独立进程服务代理进行通信

```
iRemoteService.getValue()
```
```
iRemoteService.getUser("android")
```


# aidl跨进程通信调用流程

# 1、绑定服务

![](https://github.com/chaozhouzhang/learning-summary/blob/master/Android/Binder%E6%9C%BA%E5%88%B6-Android%E8%BF%9B%E7%A8%8B%E9%97%B4%E9%80%9A%E8%AE%AF/aidl-bindService.png?raw=true)

1.1、Activity中通过bindService()方法绑定远程服务RemoteService，具体实现方法是ContextImpl的bindService()方法。

1.2、远程服务RemoteService绑定成功后会回调onBinder()方法，返回远程服务RemoteService的具体实现RemoteServiceImpl()。

1.3、远程服务绑定成功后，会回调onServiceConnected()方法，通知当前进程也就是AidlActivity，远程服务RemoteService已经连接成功，可以获取远程服务的代理进行进程间的通信了。
# 2、获取服务代理并通过服务代理进行通信

![](https://github.com/chaozhouzhang/learning-summary/blob/master/Android/Binder%E6%9C%BA%E5%88%B6-Android%E8%BF%9B%E7%A8%8B%E9%97%B4%E9%80%9A%E8%AE%AF/adil-callService.png?raw=true)

2.1、获取IRemoteService.Stub获取远程服务的代理，也就是IRemoteService.Stub.Proxy。

2.2、通过代理IRemoteService.Stub.Proxy调用远程服务的方法，代理IRemoteService.Stub.Proxy通过asBinder()方法获取BinderProxy调用transact()方法通知远程服务，而transact()方法使用JNI技术调用了native方法transactNative()方法。

2.3、transactNative()方法最后会调用远程服务IRemoteService.Stub的onTransact()方法，onTransact()方法会通过asBinder()方法获取RemoteServiceImpl，并调用RemoteServiceImpl的实际实现方法。



欢迎关注Android技术堆栈，专注于Android技术学习的公众号，致力于提高Android开发者们的专业技能！

![Android技术堆栈](https://mmbiz.qpic.cn/mmbiz_jpg/MADc6NnIysDjTRbKsg6y2G5eqqQkPDiak4V8jqKLmntDgAfFE8LOibxnSdfJESLJEM8ibrN9RGiamib4rYCt3cU08aQ/0?wx_fmt=jpeg)


