# Android.Interview-experience
自己面试过程中遇到的一些题目加以分类总结，顺便梳理下知识。
## 在快要毕业之际，整理下以前面试的题目做个大概的梳理，主要包括android。java计网算法等另做整理。
### 1.Activity
> Q1:请描述下Activity的生命周期?

> Ans: 
* onCreate 创建做initView等操作 
* onStart 很短但是此时只是可见并没有焦点
* onResume 获得焦点 就是真正的开始活动了可见并且在前台
* onPause 可见失去焦点 可以做数据存储的操作
* onStop 直接不可见了 可以注销广播之类的
* onDestory 资源的释放 重新开始则 onRestart --> onResume则重新到前台可见有焦点
* ps:前台有onResume onPause  可见有 onStart到onStop 实际使用其实区别不大 比如上面的注销广播其实onPause一样注销
切换横竖屏 onPause onStop onDestory onCreate onStart onResume 
> Q2:Activity A启动另一个Activity B会回调哪些方法？如果Activity B是完全透明呢？如果启动的是一个对话框Activity呢？

> Ans: 
* 就看B会不会影响到A的可见和焦点问题
* Activity A启动另一个Activity B会回调的方法：Activity A的onPause() -->Activity B的onCreate()-->onStart()-->onResume()-->Activity A的onStop()；如果Activity B是完全透明的，则最后不会调用Activity A的onStop()；如果是对话框Activity，同后种情况。
>Q3:谈谈onSaveInstanceState()方法？何时会调用？与onPause()的区别？

>Ans:
* 当被因为内存不足等原因强制杀死的时候会调用onSavaInstanceState()来保存状态 但是值临时性的状态保存 而onPause做的可以是对数据的持久化保存。
* 在之后重建的时候会调用onRestoreInstanceState在onStart之后 保存的Bundle传递进去并取出数据进行恢复。
>Q4:如何避免配置改变时Activity重建？

>Ans:
* 由于配置的改变可能会导致Activity的重建 可以在AndroidManifest.xml中对应的Activity中设置android:configChanges="orientation|screenSize"。
这样再次旋转的时候就不好呗重建了 只要重写onConfigurationChanged方法即可。
>Q5:Android中的4种启动模式

>Ans:
* standard模式（闹钟模式） 就是普通模式 就是栈模式 
* singleTop模式 （浏览器书签） 在栈顶就复用 否则新建
* singleTask模式（浏览器主页面） 是否有  必须只能一个  有则上面撸光弄上来复用
* singleInstance模式（来电通话）一个呆逼一个栈无则创建 有则不管在哪提头来见转到前台
* ps:singleTop模式可以存在多个 可以重复创建 而singleTask模式只能有一个
>Q6:如何启动其他应用的Activity？

>Ans:
* 显示跳转不谈了
* 隐式跳转的话 首先.setAction 要执行的操作 .setData 要访问的数据 另外IntentFilter过滤匹配原则是，必须完全完全匹配才能启动该Acitivty。但是与此同时Activity可以有多个IntentFilter 只要有一个中了就行了
>Q7:Activity的启动过程?

>Ans:
* 首先只要涉及到启动就一定一定要想起AMS：ActivityManagerServie
* 具体过程-->AMS.startActivity然后通过IPC也就是Binder类回到了ActivityThread里的ApplicationThread中 最后调用scheduleLaunchActivty来交给了HandlerH处理--> handleLaunchActivity最后处理咯
* 首先请求由1.Instrumentation（工具）来处理  然后通过Binder向2.AMS（activity manager service）发请求  AMS中有个3.Activitystack并负责activity的状态同步  AMS通过ActivityThread去同步activity的状态  从而最后完成生命周期方法的调用
在Activitystack中有个.removeTopActivityInnerLocked方法用来onPause栈顶的activity 然后最终在Activitystacksupervisor（监督）中有.realStartActivityLocked方法.scheduleLaunchActivity（app.Thread）来完成生命周期的调用过程（具体通过.handleLaunchActivity来完成）  所以旧Activity先onPause再新的
>Q8:Activity之间如何传递数据

>Ans:
* 还是Intent这个骚东西 .putExtra来携带数据 另外如何回传 
* 在1中.startActivityForResult来开启2 在2中.putExtra  
* 在1中.onActivityResult  里面.getStringExtra方法
>Q9:如何安全退出多个Activity

>Ans:
* 强制退出  通过抛出异常  
* 通过父类 在oncreate中创建一个add方法 把这些小东西都加进来  最后创建一个killAll方法 复制一份集合在遍历后关闭  
* 发送一个广播  这个广播的逻辑最好在父类中实现  最后退出的时候也仅需发送action为注册时的antion  
>Q10:Activity中怎么测量View的高宽：

>Ans:
* 首先可以在activity中调用View.getWidth、View.getHeight啥的 但是在onCreate、onStart、onResume中都会返回0  因为当前activity还没有真正的加入到WindowPhone的DecorView 所以此时真的为0。
* 真正有效的方法：1）在回调过程中 比如在点击事件啥的 这时候已经在DecroView中了 
2）或者在Resume的最后开线程延迟300ms就可以正常的View.getWidth、View.getHeight因为在onResume本来就是短暂的 完成后就会加载出页面了
3）或者在onCreate里为View手动添加回调：getViewTreeObserve.addOnGlobalLayoutListener

### 2.Fragment
>Q11:谈一谈Fragment的生命周期？

>Ans:
* Fragment从创建到销毁整个生命周期中涉及到的方法依次为：onAttach()->onCreate()-> onCreateView()->onActivityCreated()->onStart()->onResume()->onPause()->onStop()->onDestroyView()->onDestroy()->onDetach()
* onAttach:Fragment与Activity建立连接的调用
* onCreateView:当Fragment创建视图的时候调用
* onActivityCreated:当相关联的Activity完成oncreate的时候调用
* onDestoryView:当Fragment中的View被摧毁时调用
* onDetach:当与Activity解绑时调用
>Q12:Activity和Fragment的异同？

>Ans:
* 相同点：都包含布局文件、有自己的生命周期且高度相似
* 不同点：因为Fragment是依托在Activity上的所以多了很多相关的生命周期 另外其生命周期是由它自己的老大Activity调用的 Fragment里的都是public也证明了这一点。
>Q13:Activity和Fragment的关系？

>Ans:
* 首先一个Activity中可以拥有多个Fragment 一个Fragment也可以在多个Activity中使用
* Activity通过FragmentManager这个类来调用他的Fragment的生命周期 只要一致就可以继续的调用
>Q14:何时会考虑使用Fragment?

>Ans:
* 比如一个平板一个普通手机两种界面布局 就一个使用两个Fragment但是只有一套代码就能适配了 更加的方便轻量化

### 3.Service
>Q15:谈一谈Service的生命周期？

>Ans:
* onCreate:第一次被创建
* onStartComand:被启动时调用
* onBind:被绑定时调用
* onUnBind:被解绑时调用
* onDestroy:被停止时调用 这里不是onStop
>Q16:Service的两种启动方式？区别在哪？

>Ans:
* 组件调用Context.startService() 启动并且回调service的onStartComand 如果第一次则回调：onCreate-->onStartComand
启动后就一直保持运行状态了 直到stopService被组件调用了 则回调onDestory 另外：无论启动了多少次只要一次stopService则服务停止
* 组件调用Context.bindService() 绑定会回调onBind方法 如果第一次则回调：onCreate-->onBind 之后onBind返回的是IBinder对象 
这也就实现了跨线程的通信 直到unbindService才会停止服务 回调Service的onUnBind、onDestroy
>Q17:一个Activty先start一个Service后，再bind时会回调什么方法？此时如何做才能回调Service的destory()方法？

>Ans:
* 仅仅只会回调onBind方法 想onDestory必须同时调用stopService和unbindService才行
>Q18:Service如何和Activity进行通信？

>Ans:
* bindService就可以实现Acitivty向Service通信 因为返回的是IBinder对象
* 通过广播实现Service向Activity发送消息
>Q19:用过哪些系统Service？

>Ans:
* 肯定POWER_SERVICE-->PowerManager ACTIVITY_SERVICE-->ActivityManager
>Q20:是否能在Service进行耗时操作？如果非要可以怎么做？

>Ans:
* 虽然Service比UI界面的响应时间可以多一点但是同样必须开一个子线程否则还是可能ANR
>Q21:前台服务是什么？和普通服务的不同？如何去开启一个前台服务？

>Ans:
* 前台服务所以与后台服务不同的是可以被看到 会一直有一个正在运行的LOGO在系统的状态栏里 很类似通知的效果并且服务被干掉时也会被移除
* 和发送通知一样调用的是startForeground()
>Q21:如何保证Service不被杀死？

>Ans:
* 在Service的回调方法onStartCommand中设置flags为START_STICKY 这样一杀死就会再次启动Service
* 在Activity中的onDestory方法中发送广播 并在receive的重写中启动Service

### 4.Broadcast Receiver
>Q22:广播有几种形式？什么特点？

>Ans:
* 普通广播：完全的异步广播 基本上所有的广播接收器都会同时接受到这条广播或者说顺序是随机的
* 有序广播：与上者不同是同步的 一个时刻只有一个接收器会接受到这条广播 只有在第一个onReceive中执行完毕时才会继续传递按照优先级 因此也可以被接收器拦截下来
* 本地广播：相对安全，因为只会在应用程序的内部进行传递
* 粘性广播：会一直滞留 除非有来找他的接收器才会被接受到
>Q23:广播的两种注册形式？区别在哪？

>Ans:
* 四大组件中也只有这一个骚东西是可以又动态又静态的了
* 相同点：无论哪种情况都完成了对接收器和能接受哪种广播的定义
* 不同点：静态的不启动程序也会接受广播 一开机就行了 动态的就必须启动了

### 5.ContentProvider
>Q24:ContentProvider了解多少？

>Ans:
* 首先吧 这东西太像数据库了也有CURD 另外他和文件IO存储啊 SP SQLite都是提供数据存储的
* 不同的是CP可以也主要是提供给别的应用程序使用的 比如通讯录啊 但是可以选择只对哪部分进行共享的
* 底层还是Binder机制 因为resolver永远是做的代理类  而Provider是真正的做CURD的 就是A调用B的方法 B通过A传递进来的参数做真正的CURD

### 6.数据存储
>Q25:Android中提供哪些数据持久存储的方法？

>Ans:
* File：其实就是Java里的IO操作 不过注意下是IS还是OS一切正对于内存 另外OS时记得用byte[]这个小卡车
* SP：很轻量级的 用来存储比如简单的登录的用户名之类的 就是个键值对的XML 并发容易凉
* SQLite：虽然是轻量级但是也是数据库啊  是关系型的类似于excel文件 可以存储大量复杂的关系型数据
* CP：很骚不仅本地可以 还可以提供给其他的应用程序见上
>Q26:SharePreferences适用情形？使用中需要注意什么？

>Ans:
* 只能是简单的 并且系统对SP的存储有缓存策略但是记得好像只有5M 所以在并发多进程的时候读写可能会丢包
>Q27:了解SQLite中的事务处理吗？是如何做的？

>Ans:
* 首先SQLite做CURD都是默认开启transaction的 然后把SQL语句翻译成对应的SQLiteStatement并调用CURD 
* 但是一直是在rollback journal这个临时文件上的 只有完成了操作才会更新.db数据库
>Q28:使用SQLite做批量操作有什么好的方法吗？

>Ans:
* 用transaction啊事务啊 用SQLiteDatabase的begin transaction来开启一个事务然后就可以把批量的SQL语句转化成SQLiteStatement进行操作了
结束后再endTransaction
>Q29:如果现在要删除SQLite中表的一个字段如何做？

>Ans:
* 不支持不支持 只能创建一个新表满足相应的要求
>Q30:使用SQLite时会有哪些优化操作?

>Ans:
* 做批量操作时使用transaction
* 及时关闭cursor 以防内存泄露
* 在添加和更新时 的CV contentValues因为是采用了HashMap来存储键值对的一旦扩容很坑 所以估量很重要
* 因为其实大量也是耗时操作 所以尽量放在异步线程中

### 7.IPC
>Q31:Android中进程和线程的关系？

>Ans:
* 安卓系统就是土壤 进程就是APP应用程序就是工厂 线程就是工厂里的工厂线 UI线程就是主生产线
* 一个APP一般对应一个进程和有限个线程 不可以无限因为创建和销毁都有开销
>Q32:为何需要进行IPC？多进程通信可能会出现什么问题？

>Ans:
* 所有运行在不同进程的四大组件，只要它们之间需要通过内存在共享数据，都会共享失败
因为android为不同的app分配了不同的虚拟机 而不同的虚拟机则分配了不同的地址空间 这样的会产生多份副本 不进行进程间通信就凉了
* 1.静态变量和单例模式的失效 因为独立的虚拟机 所以每个虚拟机都会有不止一个的单例
2.线程同步失效 因为独立的虚拟机 3.SP不行了 因为2个进程同时读写不支持缓存也有可能丢包 4.application重复创建 因为每次分配虚拟机相当于又把应用程序重启了一遍
>Q33:什么是序列化？Serializable接口和Parcelable接口的区别？为何推荐使用后者？

>Ans:
* 说白了就是把一个对象转换成一种可存储可传输的状态 也就可以传输在网络也可以存储在本地了 比如在Intent和Binder里需要传输的对象都必须完成序列化
* S是java里的直接转换的 简单但是开销大 就是适合SD卡或者网络中的
* P是android的 是将对象分解然后每一部分都支持传递了 高效但也麻烦 所以适合内存因为高效
>Q34:Android中为何新增Binder来作为主要的IPC方式？

>Ans:
* 主要涉及到一个传输效率的问题 而这个主要受内存拷贝次数的影响 正常的情况来说都是从发送方的缓存区拷贝到内核的缓存区 再从内核的缓存区拷贝到接收方的缓存区 所以一共两次拷贝
* 而Binder厉害的就是第二次拷贝的接收方缓存区和内核缓存区其实是映射到的同一块物理地址的 节省了一次的数据拷贝过程
* 第二厉害的就是Linux下其实IPC除了Socket都不是CS架构的 而Binder则是CS架构的 Server端和Client端相对独立 稳定性就很好了
* 最后就是安全性高 因为Binder的接收方可以获得对方进程可靠的UID/PID 并且会以此检查有效性
>Q35:使用Binder进行数据传输的具体过程？AIDL?

>Ans:
* 首先Binder是服务端提供给Client的 Client通过AIDL的asInterface将这个Binder转换成Proxy代理类 并通过这个代理类发起请求 这个请求会在服务端的onTransact中处理 最终返回给Client
* 总结下就是首先服务端的方法才是真正做操作的 而自动生成的AIDL只有空方法 说到底只是用来做标识的 而Client也只是拿到Proxy代理类不做实际操作 真正的操作还是在服务端的 最后返回结果给Client就是啦

>Q36:Binder框架中各个角色的作用？

>Ans:
* Server/Client:以下都做比拟 就好比打电话的两个人A和B 打电话的A和接电话的B 他们都需要在SM上注册自己的电话号码
* SM：就好比电话本 在打电话前Client也就是A可以在电话本中查到B的联系方式 也就是Client可以通过Binder的名字获得Server中的Binder实体的引用只是引用哦
* Binder驱动：就好比是电话基站 负责进程之间的Binder通信的建立 说白了就是电话号码拨下去你得通啊 其中包括了一些的驱动和接口的协议
也提供了open、poll等方法
>Q37:Android中有哪些基于Binder的IPC方式？简单对比下？

>Ans:
* Bundle:简单 但是只能四大组件
* 文件存储：简单 明显不适合高并发 并且其实无法做到即时即时即时的通信
* AIDL：强就完事了 支持一对多也支持即使
* CP：支持一对多 可以理解为小AIDL 但是主要提供简单的数据源的CURD的操作
* Socket：主要用来网络数据的交换的 也支持一对多
>Q38:是否了解AIDL？原理是什么？如何优化多模块都使用AIDL的情况？

>Ans:
* 名字：Andorid接口定义语言
* 过程：一个进程向调用另一个进程的方法 可以通过自动生成的AIDL生成的可序列化的参数 AIDL可以生成服务器的代理类代理类代理类 然后客户端就可以通过这个代理类间接间接的做方法调用 其实就是告诉服务端我要调用哪个方法了 真正的操作还是放在服务端中进行的
* Stub类：在服务端的，是Binder的实现类就是个Binder
* Proxy类：在服务端的，是代理类给客户端调用服务端方法用的
* asInterface：在客户端的，将服务端的Stub也就是Binder对象转换成客户端需要的AIDL的接口对象 同时返回：如果同进程就是Stub本身 不是的话则Stub.proxy
* transact：在客户端的，就是用来客户端发起请求的
* onTransact:在服务端的Binde的线程池中 因为可能又多个业务模块一起IPC哦，当客户端发起请求的时候 交给这个方法来真正处理的

### 8.View
>Q39:MotionEvent是什么？包含几种事件？什么条件下会产生？

>Ans:
* ACTION_DOWN：手指刚接触屏幕
* ACTION_MOVE：手指在屏幕上滑动
* ACTION_UP：手指在屏幕上松开的一瞬间
* ACTION_CANCEL：手指保持按下操作，并从当前控件转移到外层控件时会触发
>Q40:scrollTo()和scrollBy()的区别？

>Ans:
* scrollBy内部调用了scrollTo，它是基于当前位置的相对滑动；而scrollTo是绝对滑动，因此如果利用相同输入参数多次调用scrollTo()方法，由于View初始位置是不变只会出现一次View滚动的效果而不是多次。
>Q41:谈一谈View的事件分发机制？

>Ans:
* 本质：就是MotionEvent发送的时候 需要将这个点击事件传递给一个具体的View
* 传递的顺序：Activity-->ViewGroup-->View 
* 先dispatchTouchEvent 返回boolean 看看要不要继续分发了 然后onInterceptTouchEvent 这个只有ViewGroup中有 一旦拦截则执行ViewGroup的onTouchEvent
则不继续分发给View了 后面的也都交给VG处理了 
* onTouchEvent 就是进行事件处理的
>Q42:如何解决View的滑动冲突？

>Ans:
* 处理规则：主要根据内外部滑动的方向一致否来处理 一致则根据业务需求 规定何时让外部View拦截何时内部。不一致则 可以根据滑动的方向来判断谁来拦截
* 实现方法：
* 外部拦截法：指点击事件都先经过父容器的拦截处理，如果父容器需要此事件就拦截，否则就不拦截。具体方法：需要重写父容器的onInterceptTouchEvent方法，在内部做出相应的拦截。
* 内部拦截法：指父容器不拦截任何事件，而将所有的事件都传递给子容器，如果子容器需要此事件就直接消耗，否则就交由父容器进行处理。具体方法：需要配合requestDisallowInterceptTouchEvent方法。
>Q43:谈一谈View的工作原理？

>Ans:
* 简单的说就是先measure测量啊用于对View的测量宽高，再layout布局用于确定四个顶点的位置和实际宽高，最后就是draw绘制过程了将View绘制在屏幕上
* View的绘制流程是从ViewRoot和performTraversals开始 也就是开始遍历
* performTraversals()依次调用performMeasure()、performLayout()和performDraw()三个方法，分别完成顶级 View的绘制
* 其中，performMeasure()会调用measure()，measure()中又调用onMeasure()，实现对其所有子元素的measure过程，这样就完成了一次measure过程；接着子元素会重复父容器的measure过程，如此反复至完成整个View树的遍历。layout和draw同理。对就是像个树一样的先所有左右子树再每个左右子树high
>Q44:onTouch()、onTouchEvent()和onClick()关系？

>Ans:
* 优先度onTouch()>onTouchEvent()>onClick()。因此onTouchListener的onTouch()方法会先触发；
* 如果onTouch()返回false才会接着触发onTouchEvent()，同样的，内置诸如onClick()事件的实现等等都基于onTouchEvent()；
* 如果onTouch()返回true，这些事件将不会被触发。
>Q45:SurfaceView和View的区别？

>Ans:
* 首先SurfaceView是View中派生出来的
* View需要在UI线程中做更新 而Surface可以在子线程中做更新
* View适用于主动刷新 而Surface适用于被动频繁的刷新也很好理解 因为频繁不堵塞只能在子线程中了
* 最后Surface底层实现了双缓冲机制 而View么得所以更适合频繁的被动刷新
>Q46:invalidate()和postInvalidate()的区别？

>Ans:
* invalidate()与postInvalidate()都用于刷新View，主要区别是invalidate()在主线程中调用，若在子线程中使用需要配合handler
而postInvalidate()可在子线程中直接调用

### 9.Drawable等资源
>Q47.了解哪些Drawable？适用场景？

>Ans:
* BitmapDrawable表示一张图片
* NinePatchDrawable可自动地根据所需的宽/高对图片进行相应的缩放并保证不失真
* ShapeDrawable表示纯色、有渐变效果的基础几何图形
* StateListDrawable表示一个Drawable的集合且每个Drawable对应着View的一种状态
* LayerDrawable可通过将不同的Drawable放置在不同的层上面从而达到一种叠加后的效果
>Q48:mipmap系列中xxxhdpi、xxhdpi、xhdpi、hdpi、mdpi和ldpi存在怎样的关系？

>Ans:
* 表示不同密度的图片资源，像素从高到低依次排序为xxxhdpi>xxhdpi>xhdpi>hdpi>mdpi>ldpi，根据手机的dpi不同加载不同密度的图片
>Q49.dp、dpi、px的区别？

>Ans:
* px：像素，如分辨率1920x1080表示高为1920个像素、宽为1080个像素
* dpi：每英寸的像素点，如分辨率为1920x1080的手机尺寸为4.95英寸，则该手机DPI为（1920x1920+ 1080x1080）½/4.95≈445dpi
* dp：密度无关像素，是个相对值 比如xml里的80dp
* sp：google的建议TextView的字号最好使用sp做单位
>Q50:res目录和assets目录的区别？

>Ans:
* res/raw中的文件会被映射到R.java文件中，访问时可直接使用资源ID，不可以有目录结构
* assets文件夹下的文件不会被映射到R.java中，访问时需要AssetManager类，可以创建子文件夹

### 10.Animation
>Q51:Android中有哪几种类型的动画？

>Ans:
* View动画（View Animation）/补间动画（Tween animation）：对View进行平移、缩放、旋转和透明度变化的动画，不能真正的改变view的位置。应用如布局动画、Activity切换动画
* 属性动画（Property Animation）：对该类对象进行动画操作，真正改变了对象的属性
* 逐帧动画（Drawable Animation）：其实也不算动画只是会按照顺序播放一组预先定义好的图片
>Q52:View动画为何不能真正改变View的位置？而属性动画为何可以？

>Ans:
* View动画改变的只是View的显示，而没有改变View的响应区域
* 而属性动画则是通过反射机制 真正的获得get、set方法 然后执行真正的改变了对象的属性值
>Q53:属性动画插值器和估值器的作用？

>ANS:
* 插值器：就是根据时间的流逝来计算当前属性改变的百分比 就可以确定变化的模式了 匀速变化、加速变化-->线性插值器、加速减速插值器两头慢中间快、减速插值器
* 类型估值器：根据当前的属性变化的百分比计算出改变后的属性值 只针对属性动画

### 11.Window
>Q54.Activity、View、Window三者之间的关系？

>Ans:
* 只要知道Window是Activity和View的桥梁
* Activity通过setContentView将View设置到了PhoneWindow
* View通过WindowManager的addView()、removeView()、updateViewLayout()对View进行管理
* 而在Activity启动过程其中的attach()方法中初始化了PhoneWindow
