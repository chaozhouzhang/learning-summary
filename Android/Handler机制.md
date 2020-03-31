
# Handler机制


## 如何
```java
public class HandlerActivity extends AppCompatActivity {

    private static Handler sHandler;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                Message message = new Message();
                message.obj = "Message From A.";
                //发送消息
                sHandler.sendMessage(message);
            }
        }, "A").start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                //创建Looper和MessageQueue
                Looper.prepare();
                //创建Handler
                sHandler = new Handler(Looper.myLooper()) {
                    @Override
                    public void handleMessage(@NonNull Message msg) {
                        super.handleMessage(msg);
                        //处理消息
                        System.out.println("Message To B:" + msg.obj);
                    }
                };
                //循环Looper
                Looper.loop();
            }
        }, "B").start();
    }
}
```

