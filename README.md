# JoyTap

1. Android端集成
2. Android端用法

## 1. Android端集成
### 1.1 Gradle的集成方式
1. 在Project-->build.gradle中增加`maven { url "https://raw.githubusercontent.com/wingyan1/JoyTapAAR/master" }`

![image.png](https://upload-images.jianshu.io/upload_images/1679203-6484bcaef5c5a5c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2. 在Project-->app-->build.gradle中增加依赖`implementation 'com.joytap:login:1.0'`, `implementation 'com.joytap:pay:1.0'`

![image.png](https://upload-images.jianshu.io/upload_images/1679203-fd664c4ed104786c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3. 如果自己的项目中有依赖`facebook`和`com.android.support`，需要在Project-->app-->build.gradle中增加：

```
// 用于指定所有module引用同一版本的support和facebook
// 如果还有其它依赖冲突，可照此规则制定一个唯一版本
configurations.all {
    resolutionStrategy.eachDependency { DependencyResolveDetails details ->
        def requested = details.requested
        if (requested.group == 'com.android.support') {
            if (!requested.name.startsWith("multidex")) {
                details.useVersion '27.0.2'
            }
        } else if (requested.group == 'com.facebook.android') {
            if (!requested.name.startsWith("multidex")) {
                details.useVersion '5.1.1'
            }
        }
    }
}
```
### 1.2 AAR的集成方式
1. Project-->app-->libs增加：`joytap-pay-[version].aar`,`joytap-login-[version].aar`
2. Project-->app-->build.gradle下增加以下依赖，如果项目中已经有了的依赖则不用添加

```java
// 1. 支付 billing kotlin gson
implementation(name: "joytap-pay-[version]", ext: "aar")
implementation 'com.android.billingclient:billing:1.1'
implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"

// 2. 账号 volley facebook-login play-services-auth gson
implementation(name: "joytap-login-[version]", ext: "aar")
implementation 'com.android.volley:volley:1.1.1'
implementation 'com.facebook.android:facebook-login:[5,6)'
implementation ('com.google.android.gms:play-services-auth:16.0.1'){
	exclude group: 'com.android.support', module:'support-v4'
}
implementation 'com.google.code.gson:gson:2.8.5'
```
3.在Project-->app-->build.gradle下增加libs的引用路径

```gradle
repositories {

    flatDir {
        dirs 'libs'
    }
    jcenter()
}
```

## 2. Android端用法

2.1 初始化和设置测试环境，在初始化成功的回调中可以获取到`uid`,`access_token`, `loginType`

```java
JTAccountManager.getInstance().initClientId("", this, "", new JTLoginCallback() {
	@Override
	public void onLoginSucc(String uid, String access_token, int loginType) {
   		// write your code   
	}

	@Override
	public void onLoginError(VolleyError error) {
		
	}
});
// 1. 设置测试环境
JTAccountManager.getInstance().setDebug(true);
```
2.2 一定要记得重写Activity的**`onActivityResult`**，并在其中调用`FBLoginHelper`和`GPLoginHelper`的onActivityResult方法，否则会影响FB、GP登录和绑定功能的使用

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        FBLoginHelper.onActivityResult(requestCode, resultCode, data);
        GPLoginHelper.onActivityResult(requestCode, resultCode, data);
        super.onActivityResult(requestCode, resultCode, data);
}
```
2.3 登录、退出登录`JTAccountManager`

```java
/*
* 1. 如果曾经登录过，则触发自动登录
* 2. 如果未曾登录过，则弹出选择登录方式的界面
* 3. 自动登录失败，会弹出选择登录方式界面
* */
public void login()
// 2. 退出登录
public void logout(final RequestCallbackBean callback)
```
2.4 绑定Facebook：`FBLoginHelper`

```java
FBLoginHelper.fbLoginOrBind(activity, false, callback);
```
