###一、进程与线程区别

- **进程**：系统进行资源分配和调度的基本单位，进程是线程的容器

  - ##### **进程的生命周期：**

    - 前台进程
    - 可见进程
    - 服务进程
    - 后台进程
    - 空进程

- **线程**：CPU调度的最小单位。线程是进程中可独立执行的最小单位，也是 CPU 资源（时间片）调度的基本单位。同一个进程中的线程可以共享进程中的资源，如内存空间和文件句柄

  - ##### **线程的生命周期**

    - **新建**：当程序使用new关键字创建了一个线程之后，该线程就处于新建状态，此时仅由JVM为其分配内存，并初始化其成员变量的值
    - **就绪**：当线程对象调用了start()方法之后，该线程处于就绪状态。JVM会为其创建方法调用栈和程序计数器，等待调度运行
    - **运行**:如果处于就绪状态的线程获得了CPU，开始执行run()方法的线程执行体，则该线程处于运行状态
    - **阻塞** : 当处于运行状态的线程失去所占用资源之后，便进入阻塞状态
    - **死亡**:线程会以如下3种方式结束，结束后就处于**死亡状态**：
      - **①** run()或call()方法执行完成，线程正常结束。
      - **②** 线程抛出一个未捕获的Exception或Error。
      - **③** 直接调用该线程stop()方法来结束该线程——该方法容易导致死锁，通常不推荐使用

  

###二、开启多进程方法

####1、Android中的多进程模式：

- 在Android中多进程是指一个应用中存在多个进程的情况,实现方法就是给四大组件在AndroidManifest中指定android:process属性。
- android:process=":remote"代表当前的完整包名加上：remote进程，
  "："开头的进程属于当前应用的私有进程,其他的应用组件不可以和它跑在同一个进程中.
  而进程名不以冒号开头的进程（android:process="com.ryg.chapter_2.remote"),属于全局进程,其他应用通过ShareUID方式可以和它跑在同一个进程中。
- 不同进程的组件会拥有独立的虚拟机，Application以及内存空间

####2、实现跨进程的通信方式：

- Intent来传递数据

- 共享文件和SharedPreferences

- 基于Binder的Messenger和AIDL以及Socket等

  

###三、IPC基础概念介绍

- **序列化**：序列化就是一种用来处理对象流的机制，所谓对象流也就是将对象的内容进行流化，Serializable和Parcelable接口可以完成对象的序列化的过程,当我们需要通过Intent和Binder传输数据的时候,就需要进行序列化。

- **序列化方式：**

  - **实现Serializable接口**（空实现，标志接口）：

    - 需要一个类实现Serializable接口,并声明一个serialVersionUID(不是必须的,它是用来反序列化的,序列化的时候系统会把当前类的serialVersionUID写入序列化的文件中,当反序列化的时候,系统回去检测文件中的serialVersionUID,如果一致,则可以实现反序列化,否则不能实现反序列化）。

    - 序列化：(将文字写入文件）

      ```java
      	User user = new User(0,"jake",true);
      	ObjectOutputStream out = new ObjectOutputStream(new 	FileOutputStrean("cache.txt");
      	out.writeObject(user);
      	out.close();
      ```

      

    - 反序列化(读出文件)

      ```java
      ObjectInputStream in = new ObjectInputStream(new FileOInputStrean("cache.txt");
      	User newUser = (User)in.readObject();
      	out.close();
      ```

    - 静态成员变量属于类，而不属于对象，所以不会参加序列化的过程。并且使用了transient标记的成员变量也不会参与序列化。

      

  - **实现Parcelable接口**：需要一个类实现Parcelable接口,并生成一些方法

    - 序列化功能是由writeToParcel方法来完成的,最终通过Parcel中的一系列write方法来完成的。
    - 反序列化的功能是由CREATOR来完成的,其内容标明了如何创建序列化对象和数组,并通过Parcel的一系列read方法来完成反序列化的过程。
    - 内容描述是通过describeContents来完成的。

  - **Serializable和Parcelable接口的区别**
    - Serializable是Java的序列化接口,使用起来开销很大,因为其序列化过程和反序列化的过程需要大量的IO操作。
    - Parcelable是Android提供的一个序列化方式,缺点就是有点麻烦，但是可以通过插件生成。并且其效率高,主要用在内存的序列化上。

- **Binder**(从Framework的角度来说,Binder是ServiceManager连接各种（ActivityManager,WindowManager等）和ManagerSerice的桥梁
  - 在Android中,Binder主要用在Service,包括AIDL和Messager.在项目中创建一个aidl文件,其核心是其生成的Java文件的内部Stub和Stub的内部代理类Proxy,系统会默认生成一个这样的java文件(这个类主要分为两部分,首先它本身是一个Binder的接口（继承了IInterface，所有在Binder中传输的接口都需要继承自IInterface的接口),其次它的内部是Stub类。
  - Binder中有两个比较重要的方法（linkToDeath和unlinkToDeath),Binder运行在服务端进程,如果服务端进程由于某种原因异常终止,这个时候我们到服务端的Binder链接断裂（称为Binder死亡）,会导致我们的远程调用失败，客户端的功能就会收到影响。linkToDeath是一个Binder死亡代理,当Binder死亡的时候,就会收到通知,从而解决上面的问题。



###四、Android中的IPC方式

####1、Bundle

- 实现了Parcelable接口,所以这个可以在不同的进程间传输,可以在Bundle中附加我们需要传输给远程进程的信息并通过Intent发送出去。

  

####2、使用文件共享

- A进程把数据写入文件，B进程通过读取这个文件来获取数据。但是并发的读/写会造成较大的问题。
- SharePreference：是Android提供的轻量级存储方案,本质是xml文件，通过键值对形式存储数据。但是不建议在进程之间通信的时候使用SharePreference，其对于读/写有一定的缓存策略（内存中会有一份SharePrefernce的缓存），很大几率会造成数据丢失



####3、使用Messenger

- 信使,底层实现方案是AIDL。在Messager中进行数据传递必须将数据放入Message中新建一个Handler,拿到message后做相关操作。Messager是以串行的方式来处理的,不适合并发.

####4、使用AIDL

- 支持的数据类型有基本数据类型，String,CharSequence,ArrayList,HashMap,Parcelable对象
- 注意不要在onServiceConnectioned和onServiceDisconnected中进行耗时的操作。

###5、使用ContentProvider:底层的实现是Binder.自定义Provider就是继承自Provider,并实现其中的onCreate（运行在主线程）,query，getType,insert,delete,update等方法（运行在Binder线程池中）,其增删改查是多线程并发进行的,在运用的时候需要做到多线程同步问题。

###6、Socket：使用套接字，不仅能实现进程间通信，还能实现设备间通信。

	流式套接字TCP：面向连接，提供稳定的双向通信功能，连接的建立需要经过3次握手，画质清晰。
	用户数据报套接字UDP:面向无连接，提供不稳定的单项功能。具有更好的效率，但是不能保证数据的正确性。流畅