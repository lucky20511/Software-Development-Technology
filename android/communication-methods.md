# Communication Methods

The following are the list of method for the communication between activities and services:

### Intra-Process \(Early Binding\)

1. .Jar Library
2. Binder to Service

### Inter-Process \(Running Time Binding\)

1. Intent \(along with Bundles\)    **unBounded Service**
2. Broadcast      **unBounded Service** 
3. Messenger     Bounded Service
4. AIDL            Bounded Service
5. Http Server-Client    UnBounded



An`Intent`is "sent" when one app or`Activity`wants to launch another to do something very specific. For example, a file-manager might want to launch an image viewer or video player. Your app might want to launch a very specific`Activity`within another one of your apps, etc. The communication by specific intents \(i.e. including package name and component name\) can not easily be intercepted, so it's somewhat more secure. Most importantly, there's only and exactly one "receiver" -- if none can be found, the`Intent`will fail.

Further, a`BroacastReceiver`will be active within an`Activity`or`Service`and received broadcasts will generally only change state and/or do minor UI updates... for example, you might disable a few actions if your internet connectivity is dropped. By comparison, a specific Intent will usually launch a new`Activity`or bring an existing one to the foreground.

1\)IPC does not send data from one component to another \(it can, but its an inefficient way to do that\). IPC sends data from one process to another. An Android app is generally one process, although it doesn't have to be \(services are sometimes placed into another process by the developer\). The reason this is an important difference is that processes cannot share memory, so special methods like IPC are needed to send any data between them.

2\)Data sent between components do not have to be a Parcel. That's one way, and its the way Android uses when sending startup parameters around. But it's not necessary.

3\)Using a Binder to talk to a service is only possible if the two are in the same process. Its a method to totally avoid using IPC.

4\)AIDL is a wrapper around an IPC method. AIDL uses IPC, it just tries to make it look like normal function calls to the client.

5\)An Intent object is an abstraction for all the data needed to start a service or activity in Android. It will include parameters, which may or may not be in Parcels. It may or may not use IPC to send those parameters \(if the target Activitiy or Service is in another APK it will. If it isn't it may not\).

I think the problem here is you don't really understand what a process is, what an Android component is, and how processes actually communicate. I suggest doing some studying up on that.

@AIDL is just one way to communicate with a bound service. There is no such thing as a 'AIDL based servcie', it's just a bound service that returns a Binder generated based on an AIDL interface.

@In the ADIL guide on Android developer says

**"Note: Using AIDL is necessary only if you allow clients from different applications to access your service for IPC and want to handle multithreading in your service. If you do not need to perform concurrent IPC across different applications, you should create your interface by implementing a Binder"**

so what does this mean ?

@It means what is says: if you use AIDL you allow multiple clients to connect concurrently and you should be able to handle this. If you use a messenger, it will serialize requests for you. If you use the service in the same process, you can access it directly \(no IPC\) via your own class that extends Binder.

Bound Service

[Extending the Binder class](https://developer.android.com/guide/components/bound-services.html#Binder)

If your service is private to your own application and runs in the same process as the client \(which is common\), you should create your interface by extending the[`Binder`](https://developer.android.com/reference/android/os/Binder.html)class and returning an instance of it from[`onBind()`](https://developer.android.com/reference/android/app/Service.html#onBind%28android.content.Intent%29). The client receives the[`Binder`](https://developer.android.com/reference/android/os/Binder.html)and can use it to directly access public methods available in either the[`Binder`](https://developer.android.com/reference/android/os/Binder.html)implementation or the[`Service`](https://developer.android.com/reference/android/app/Service.html).

This is the preferred technique when your service is merely a background worker for your own application. The only reason you would not create your interface this way is because your service is used by other applications or across separate processes.

[Using a Messenger](https://developer.android.com/guide/components/bound-services.html#Messenger)

If you need your interface to work across different processes, you can create an interface for the service with a[`Messenger`](https://developer.android.com/reference/android/os/Messenger.html). In this manner, the service defines a[`Handler`](https://developer.android.com/reference/android/os/Handler.html)that responds to different types of[`Message`](https://developer.android.com/reference/android/os/Message.html)objects. This[`Handler`](https://developer.android.com/reference/android/os/Handler.html)is the basis for a[`Messenger`](https://developer.android.com/reference/android/os/Messenger.html)that can then share an[`IBinder`](https://developer.android.com/reference/android/os/IBinder.html)with the client, allowing the client to send commands to the service using[`Message`](https://developer.android.com/reference/android/os/Message.html)objects. Additionally, the client can define a[`Messenger`](https://developer.android.com/reference/android/os/Messenger.html)of its own, so the service can send messages back.

This is the simplest way to perform interprocess communication \(IPC\), because the[`Messenger`](https://developer.android.com/reference/android/os/Messenger.html)queues all requests into a single thread so that you don't have to design your service to be thread-safe.

[Using AIDL](https://developer.android.com/guide/components/aidl.html)

Android Interface Definition Language \(AIDL\) decomposes objects into primitives that the operating system can understand and marshals them across processes to perform IPC. The previous technique, using a[`Messenger`](https://developer.android.com/reference/android/os/Messenger.html), is actually based on AIDL as its underlying structure. As mentioned above, the[`Messenger`](https://developer.android.com/reference/android/os/Messenger.html)creates a queue of all the client requests in a single thread, so the service receives requests one at a time. If, however, you want your service to handle multiple requests simultaneously, then you can use AIDL directly. In this case, your service must be thread-safe and capable of multi-threading.

To use AIDL directly, you must create an`.aidl`file that defines the programming interface. The Android SDK tools use this file to generate an abstract class that implements the interface and handles IPC, which you can then extend within your service.

### Is bindService necessary for a local \(same process\) Service?

It depends on whether you need to call instance methods on that service or not. StartService gets the service up and running and working on whatever Intent you sent it but you still can't interact with it because you don't have a reference to the service object. Calling bindService is what gives that reference so that you can call instance methods on it. If you don't need that, you don't need bindService.

# 

# Binder

Activity

```java
//Activity implements the Callbacks interface which defined in the Service  
    public class MainActivity extends ActionBarActivity implements MyService.Callbacks{

    ToggleButton toggleButton;
    ToggleButton tbStartTask;
    TextView tvServiceState;
    TextView tvServiceOutput;
    Intent serviceIntent;
    MyService myService;
    int seconds;
    int minutes;
    int hours;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
         serviceIntent = new Intent(MainActivity.this, MyService.class);
        setViewsWidgets();
    }

    private void setViewsWidgets() {
        toggleButton = (ToggleButton)findViewById(R.id.toggleButton);
        toggleButton.setOnClickListener(btListener);
        tbStartTask = (ToggleButton)findViewById(R.id.tbStartServiceTask);
        tbStartTask.setOnClickListener(btListener);
        tvServiceState = (TextView)findViewById(R.id.tvServiceState);
        tvServiceOutput = (TextView)findViewById(R.id.tvServiceOutput);

    }

    private ServiceConnection mConnection = new ServiceConnection() {

        @Override
        public void onServiceConnected(ComponentName className,
                                       IBinder service) {
            Toast.makeText(MainActivity.this, "onServiceConnected called", Toast.LENGTH_SHORT).show();
            // We've binded to LocalService, cast the IBinder and get LocalService instance
            MyService.LocalBinder binder = (MyService.LocalBinder) service; 
            myService = binder.getServiceInstance(); //Get instance of your service! 
            myService.registerClient(MainActivity.this); //Activity register in the service as client for callabcks! 
            tvServiceState.setText("Connected to service...");
            tbStartTask.setEnabled(true);
        }

        @Override
        public void onServiceDisconnected(ComponentName arg0) {
            Toast.makeText(MainActivity.this, "onServiceDisconnected called", Toast.LENGTH_SHORT).show();
            tvServiceState.setText("Service disconnected");
            tbStartTask.setEnabled(false);
        }
    };

    View.OnClickListener btListener =  new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            if(v == toggleButton){
                if(toggleButton.isChecked()){
                    startService(serviceIntent); //Starting the service 
                    bindService(serviceIntent, mConnection,            Context.BIND_AUTO_CREATE); //Binding to the service! 
                    Toast.makeText(MainActivity.this, "Button checked", Toast.LENGTH_SHORT).show();
                }else{
                    unbindService(mConnection);
                    stopService(serviceIntent);
                    Toast.makeText(MainActivity.this, "Button unchecked", Toast.LENGTH_SHORT).show();
                    tvServiceState.setText("Service disconnected");
                    tbStartTask.setEnabled(false);
                }
            }

            if(v == tbStartTask){
                if(tbStartTask.isChecked()){
                      myService.startCounter();
                }else{
                    myService.stopCounter();
                }
            }
        }
    };

    @Override
    public void updateClient(long millis) {
        seconds = (int) (millis / 1000) % 60 ;
        minutes = (int) ((millis / (1000*60)) % 60);
        hours   = (int) ((millis / (1000*60*60)) % 24);

        tvServiceOutput.setText((hours>0 ? String.format("%d:", hours) : "") + ((this.minutes<10 && this.hours > 0)? "0" + String.format("%d:", minutes) :  String.format("%d:", minutes)) + (this.seconds<10 ? "0" + this.seconds: this.seconds));
    }
}
```

Service

```java
public class MyService extends Service {
    NotificationManager notificationManager;
    NotificationCompat.Builder mBuilder;
    Callbacks activity;
    private long startTime = 0;
    private long millis = 0;
    private final IBinder mBinder = new LocalBinder();
    Handler handler = new Handler();
    Runnable serviceRunnable = new Runnable() {
        @Override
        public void run() {
            millis = System.currentTimeMillis() - startTime;
            activity.updateClient(millis); //Update Activity (client) by the implementd callback
            handler.postDelayed(this, 1000);
        }
    };


    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {

        //Do what you need in onStartCommand when service has been started
        return START_NOT_STICKY;
    }

    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }
    //returns the instance of the service
    public class LocalBinder extends Binder{
        public MyService getServiceInstance(){
            return MyService.this;
        }
    }

    //Here Activity register to the service as Callbacks client
    public void registerClient(Activity activity){
        this.activity = (Callbacks)activity;
    }

    public void startCounter(){
        startTime = System.currentTimeMillis();
        handler.postDelayed(serviceRunnable, 0);
        Toast.makeText(getApplicationContext(), "Counter started", Toast.LENGTH_SHORT).show();
    }

    public void stopCounter(){
        handler.removeCallbacks(serviceRunnable);
    }


    //callbacks interface for communication with service clients! 
    public interface Callbacks{
        public void updateClient(long data);
    }
}
```

# IPC

Activity

```java
public class MainActivity extends Activity implements ServiceConnection{

    Messenger mServiceMessenger;
    Messenger mClientMessenger = new Messenger(new ClientHandler());

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Intent intent = new Intent(this,TestService.class);
        bindService(intent,this, Context.BIND_AUTO_CREATE);
    }


    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        mServiceMessenger = new Messenger(service);
        Message m = Message.obtain();
        m.replyTo = mClientMessenger;
        try {
            mServiceMessenger.send(m);
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void onServiceDisconnected(ComponentName name) {}

    public class ClientHandler extends Handler {
        @Override
        public void handleMessage(Message msg) {
            Log.d("Spam","Message Received");
        }
    }
}
```

Service

```java
public class TestService extends Service {

    private Messenger mServiceMessenger = new Messenger(new ServiceHandler());
    private Messenger mClientMessenger;
    private Random r = new Random();

    public TestService() {
        super();
    }

    @Override
    public void onCreate() {
        super.onCreate();
    }


    @Override
    public IBinder onBind(Intent intent) {
        return mServiceMessenger.getBinder();
    }

    public void initSpam(){
        for(int i=0;i<10;i++) {
            TimerTask task = new TimerTask() {
                @Override
                public void run() {
                    Bundle b = new Bundle();
                    b.putInt("INT",r.nextInt());
                    b.putLong("LONG",r.nextLong());
                    b.putBoolean("BOOL",r.nextBoolean());
                    b.putFloat("FLOAT",r.nextFloat());
                    b.putDouble("DOUBLE",r.nextDouble());
                    b.putString("STRING",String.valueOf(r.nextInt()));
                    Message msg = Message.obtain();
                    msg.setData(b);

                    try {
                        mClientMessenger.send(msg);
                    } catch (RemoteException e) {
                        e.printStackTrace();
                    }
                }
            };
            Timer timer = new Timer();
            timer.scheduleAtFixedRate(task,1,1);
        }
    }

    public class ServiceHandler extends Handler {
        @Override
        public void handleMessage(Message msg) {
            mClientMessenger = msg.replyTo;
            initBarrage();

        }
    }
}
```

# AIDL

Activity

```java
public class MainActivity extends Activity implements ServiceConnection{

    IRemoteService mService;
    TextView countTextView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Intent intent = new Intent(this,TestService.class);
        bindService(intent,this, Context.BIND_AUTO_CREATE);
    }

    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {

        mService = IRemoteService.Stub.asInterface(service);
        try {
            mService.registerCallback(mClientBinder);
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void onServiceDisconnected(ComponentName name) {}


    public final IServiceAidlCallback.Stub mClientBinder = new IServiceAidlCallback.Stub(){
        public void basicTypes(int anInt, long aLong, boolean aBoolean,
                               float aFloat, double aDouble, String aString){
            Log.d("Spam","Callback Received");
        }
    };
}
```

Service

```java
public class TestService extends Service {

    private Random r = new Random();

    private IServiceAidlCallback mClientCallback;

    public TestService() {
        super();
    }

    @Override
    public void onRebind(Intent intent) {
        super.onRebind(intent);
    }

    @Override
    public void onCreate() {
        super.onCreate();
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        return super.onStartCommand(intent, flags, startId);
    }

    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }

    public final IRemoteService.Stub mBinder = new IRemoteService.Stub(){
        public void registerCallback(IBinder callback){

            mClientCallback = IServiceAidlCallback.Stub.asInterface(callback);
            initSpam();

        }
    };

    public void initSpam(){
        for(int i=0;i<10;i++) {
            TimerTask task = new TimerTask() {
                @Override
                public void run() {
                    try {
                        mClientCallback.basicTypes(
                                r.nextInt(),
                                r.nextLong(),
                                r.nextBoolean(),
                                r.nextFloat(),
                                r.nextDouble(),
                                String.valueOf(r.nextInt()));
                    } catch (RemoteException e) {
                        e.printStackTrace();
                    }
                }
            };
            Timer timer = new Timer();
            timer.scheduleAtFixedRate(task,1,1);
        }
    }
}
```



