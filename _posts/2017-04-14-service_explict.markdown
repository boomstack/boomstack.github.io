---

layout: post 

title: "Service Intent must be explicit的两种解决方法" 

date: 2017-04-14 18:10:09 +0800 

categories: Android

---

crash的原因是5.0以上service不能使用隐式intent启动，但是使用AIDL进行进程间通信时并不能得到具体的类名，so, 问题还得解决。
## 方法一
最简单的是新建Intent的时候带入packagename，直接调用setPackage方法，把service所在的包名设置进intent。

```java
	Intent intent = new Intent();
        intent.setAction("com.boomstack.aidl");
        intent.setPackage("com.boomstack.testaidlserver");
        bindService(intent, serviceConnection, BIND_AUTO_CREATE);
```
## 方法二
根据隐式intent寻找具体的component，是根据PackageManager的
queryIntentServices方法获取所有符合intent的service，然后根据获取到的信息构建Component实例。
代码来自网络：
```java
    public Intent createExplicitFromImplicitIntent(Context context, Intent implicitIntent) {
        // Retrieve all services that can match the given intent
        PackageManager pm = context.getPackageManager();
        List<ResolveInfo> resolveInfo = pm.queryIntentServices(implicitIntent, 0);

        // Make sure only one match was found
        if (resolveInfo == null || resolveInfo.size() != 1) {
            return null;
        }

        // Get component info and create ComponentName
        ResolveInfo serviceInfo = resolveInfo.get(0);
        String packageName = serviceInfo.serviceInfo.packageName;
        String className = serviceInfo.serviceInfo.name;
        ComponentName component = new ComponentName(packageName, className);

        // Create a new intent. Use the old one for extras and such reuse
        Intent explicitIntent = new Intent(implicitIntent);

        // Set the component to be explicit
        explicitIntent.setComponent(component);

        return explicitIntent;
    }

```
原理是一样的，都是为了获取具体的组件信息，方法二比方法一更通用一些。