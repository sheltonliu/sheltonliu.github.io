---
title: Android插件化原理
date: 2019-02-28 17:01:32
categories: "插件化"
tags:
	- "插件化"
	- "Android"
---

<font size=4 >

# 动态加载技术

1. 热修复和插件化都属于动态加载技术，动态加载技术在很多领域都有应用，不仅仅只在Android中应用。
2. 热修复技术主要用来修复bug, 插件化主要解决应用越来越庞大，以及功能模块的解耦

# 插件化

插件化的客户端由宿主和插件组成。宿主指先被安装到手机中的APK， 插件一般指经过处理的apk,so和dex等文件

1. 如果加载的插件不需要和宿主有任何解耦，也无需和宿主进行通信，比如加载第三方App,那么推荐使用RePlugin.
2. 其他情况推荐使用VirtualApk.

# Activity插件化

四大组件的插件话目前主流是采用Hook技术来实现。Hook技术实现主要有两种技术

1. 通过Hook IActivityManager来实现
2. 通过Hook Instrumentation来实现

## Hook IActivityManager方案
主要步骤如下：

1. 注册Activity进行占坑，用来通过AMS的校验 
2. 使用插件Activity来替换占坑的Activity（也就是还原）

		public class IActivityManagerProxy implements InvocationHandler {
    	private Object mActivityManager;
    	private static final String TAG = "IActivityManagerProxy";

    	public IActivityManagerProxy(Object activityManager) {
        	this.mActivityManager = activityManager;
    	}

    	@Override
    	public Object invoke(Object o, Method method, Object[] args) throws Throwable {
        	if ("startActivity".equals(method.getName())) {
            	Intent intent = null;
            	int index = 0;
            	for (int i = 0; i < args.length; i++) {
                if (args[i] instanceof Intent) {
                    index = i;
                    break;
                }
            	}
            	intent = (Intent) args[index];
            	Intent subIntent = new Intent();
            	String packageName = "com.example.liuwangshu.pluginactivity";
            	//StubActivity这个是占坑的Activity
            	subIntent.setClassName(packageName,packageName+".StubActivity");
         		
         		//注意这里putExtra中的value值是intent，这个intent是真实插件构建的intent,因为后续需要还原，就用到这个intent
         		subIntent.putExtra(HookHelper.TARGET_INTENT,intent);
         		args[index] = subIntent;
            	Log.d(TAG, "hook成功");
        		}
        		return method.invoke(mActivityManager, args);
    		}
		}

下面来看下代理类IActivityManagerProxy 来替换 IActivityManager

		public class HookHelper {
    		public static final String TARGET_INTENT = "target_intent";
    		public static final String TARGET_INTENT_NAME = "target_intent_name";
    		public static void hookAMS() throws Exception {
        		Object defaultSingleton=null;
        		if (Build.VERSION.SDK_INT >= 26) {
            		Class<?> activityManagerClazz = Class.forName("android.app.ActivityManager");
            		defaultSingleton=  FieldUtil.getField(activityManagerClazz,null,"IActivityManagerSingleton");
        		} else {
            		Class<?> activityManagerNativeClass = Class.forName("android.app.ActivityManagerNative");
            		defaultSingleton=  FieldUtil.getField(activityManagerNativeClass,null,"gDefault");
        		}
        		Class<?> singletonClazz = Class.forName("android.util.Singleton");
        		Field mInstanceField= FieldUtil.getField(singletonClazz,"mInstance");
        		Object iActivityManager = mInstanceField.get(defaultSingleton);
        		Class<?> iActivityManagerClass = Class.forName("android.app.IActivityManager");
        		Object proxy = Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(),
                new Class<?>[] { iActivityManagerClass }, new IActivityManagerProxy(iActivityManager));
        		mInstanceField.set(defaultSingleton, proxy);
    		}
    		public static void hookHandler() throws Exception {
        		Class<?> activityThreadClazz = Class.forName("android.app.ActivityThread");
        		Object currentActivityThread= FieldUtil.getField(activityThreadClazz,null,"sCurrentActivityThread");
        		Field mHField = FieldUtil.getField(activityThreadClazz,"mH");
        		Handler mH = (Handler) mHField.get(currentActivityThread);
        		FieldUtil.setField(Handler.class,mH,"mCallback",new HCallback(mH));
    		}
    		public static void hookInstrumentation(Context context) throws Exception {
        		Class<?> contextImplClazz = Class.forName("android.app.ContextImpl");
        		Field mMainThreadField  =FieldUtil.getField(contextImplClazz,"mMainThread");
        		Object activityThread = mMainThreadField.get(context);
        		Class<?> activityThreadClazz = Class.forName("android.app.ActivityThread");
        		Field 
        		mInstrumentationField=FieldUtil.getField(activityThreadClazz,"mInstrumentation");
	
		FieldUtil.setField(activityThreadClazz,activityThread,"mInstrumentation",new InstrumentationProxy((Instrumentation)mInstrumentationField.get(activityThread),
                context.getPackageManager()));
    		}
		}


主要是通过反射来替换代理对象，其中这里用到的方法是hookAMS()

		Intent intent = new Intent(MainActivity.this, TargetActivity.class);
        startActivity(intent);		
        
所以这里当启动Activity的时候，其实启动的是StubActivity。

### 还原插件Activity

前面用占坑Activity通过了AMS的校验，但我们要启动的是TargetActivity还需要还原。 
还原的Hook点在哪呢？ 在Handler的dispatchMessage中的mCallback对象

	public class HCallback implements Handler.Callback{
    	public static final int LAUNCH_ACTIVITY = 100;
    	Handler mHandler;

    	public HCallback(Handler handler) {
        	mHandler = handler;
    	}
    	@Override
    	public boolean handleMessage(Message msg) {
        	if (msg.what == LAUNCH_ACTIVITY) {
            	Object r = msg.obj;
            try {
                Intent intent = (Intent) FieldUtil.getField(r.getClass(), r, "intent");
                
                //这里拿到插件Activity中的intent
                Intent target = intent.getParcelableExtra(HookHelper.TARGET_INTENT);
                intent.setComponent(target.getComponent());
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        	mHandler.handleMessage(msg);
        	return true;
    }
	}
	
然后替换代理的方法用到了上面HookHelper的hookHandler()

## Hook Instrementation 方案

1. 就是在Instrumentation的execStartActivity方法中用占坑的SubActivity来通过AMS的校验
2. 在Instrumentation的newActivity方法中还原TargetActivity


总结一下： 先在清单文件中注册Activity来进行占坑，主要为了通过AMS的校验， 然后在合适的时机用插件Activity来替换占坑的Activity


# Service的插件化

这个主要是采用代理分发来实现的。

1. 当启动插件Service时，先启动代理Service
2. 代理Service启动后调用它的onStartCommand等方法进行分发，执行插件TargetService中的onCreate方法。 

	

         