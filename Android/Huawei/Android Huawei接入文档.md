# Android Huawei接入文档

## SDK结构
支持平台：Android</br>
系统要求: Android5.0+ </br>
环境要求: Android Studio</br>
支持语言：阿语，英语</br>

## 1.接入流程
### 1.1集成AAR包
首先把AAR包复制到项目的lib目录下，然后在主项目APP的工程下build.gradle 中的dependencies加入
 ``` Groovy
 //SDK基础库
 implementation(name: 'YllGameSdk_1.0.1', ext: 'aar') 
 ```
 ### 1.2设置项目的libs文件目录和过滤so
 在主项目APP的工程下build.gradle 中的android加入
  ``` Groovy
 repositories {
        flatDir {
            dirs 'libs'
        }
    }
    packagingOptions {
        doNotStrip "*/*/libijiami*.so"
    }
 ```
 ### 1.3 导入SDK中所需第三方库
   ``` Groovy
   //Android X支持库  必须添加
    api 'androidx.appcompat:appcompat:1.2.0'
    //okhttp网络请求库 必须添加
    api("com.squareup.okhttp3:okhttp:4.9.0")
    //gson数据解析库 必须添加
    api 'com.google.code.gson:gson:2.8.5'
    //Facebook登陆依赖库 必须添加
    api 'com.facebook.android:facebook-login:13.0.0'
    //Facebook分享
    api 'com.facebook.android:facebook-share:13.0.0'
    //Google登陆依赖库 必须添加
    api 'com.google.android.gms:play-services-auth:19.0.0'
    api "com.google.android.gms:play-services-ads-identifier:17.0.0"
    //Google支付依赖库 必须添加
    api "com.android.billingclient:billing:4.0.0"
    //数据库依赖库 必须添加
    def room_version = "2.2.5"
    api "androidx.room:room-runtime:$room_version"
    annotationProcessor "androidx.room:room-compiler:$room_version"
    api "net.zetetic:android-database-sqlcipher:4.4.2"
    //数据统计依赖库 必须添加
    api 'com.appsflyer:af-android-sdk:6.2.3@aar'
    api 'com.appsflyer:oaid:6.2.4'
    api 'com.android.installreferrer:installreferrer:2.2'
    //FCM 推送相关
    api platform('com.google.firebase:firebase-bom:26.4.0')
    api 'com.google.firebase:firebase-messaging'
    api 'com.google.firebase:firebase-analytics'
    api 'com.google.firebase:firebase-core:16.0.1'
    api 'com.aliyun.dpa:oss-android-sdk:2.9.9'
    //Bugly
    api 'com.tencent.bugly:crashreport:3.3.92'
    //其中latest.release指代最新Bugly SDK版本号，也可以指定明确的版本号，例如2.1.9
    api 'com.tencent.bugly:nativecrashreport:3.9.0'
    //华为登陆 hms
    api 'com.huawei.hms:hwid:5.2.0.300'
    api 'com.huawei.hms:ads-identifier:3.4.39.302'
    //华为支付 hms
    api 'com.huawei.hms:iap:5.1.0.300'
 ```

## 2.项目配置，初始化

### 2.1Androidmanifest.xml中添加权限
``` xml
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
```
### 2.2初始化Application
在项目的application的onCreate函数中调用SDK的初始化函数，并且调用SDK的设置语言函数。
- SDK初始化函数：``` YllGameSdk.getInstance().init(); ```
``` java 
    /**
     * 初始化
     *
     * @param application
     * @param appId            游戏的gameAppId
     * @param googleClientId   游戏的googleClientId
     * @param appsFlyersDevKey 游戏的appsFlyersDevKey
     * @param buglyAppId          游戏的buglyAppId
     */
    public void init(Application application, String appId, String googleClientId, String appsFlyersDevKey,String buglyAppId)
 ```
 - SDK 设置语言集合函数：``` YllGameSdk.setLanguageList(); ```
``` java 
    /**
     * 设置SDK支持语言
     *
     * @param languageList 游戏支持语言集合 现支持 ar 阿语 en英语 该集合默认第一个是SDK的默认语言
     */
    public static void setLanguageList(List<String> languageList) 
 ```
 - SDK设置语言函数函数：``` YllGameSdk.setLanguage(); ```
``` java 
    /**
     * 设置SDK默认语言
     *
     * @param localLanguage ar 阿语 en英语
     */
    public static void setLanguage(String localLanguage)
 ```
 
**（注：项目的application要在AndroidManifest中注册，项目中初始化参数的key 找运营方）**
### 2.3配置Facebook
在项目中的AndroidManifest中添加
``` xml
        <activity
            android:name="com.facebook.FacebookActivity"
            android:configChanges="keyboard|keyboardHidden|screenLayout|screenSize|orientation"
            android:label="@string/app_name" />
        <activity
            android:name="com.facebook.CustomTabActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
                <data android:scheme="@string/fb_login_protocol_scheme" />
            </intent-filter>
        </activity>
        <meta-data
            android:name="com.facebook.sdk.ApplicationId"
            android:value="@string/facebook_app_id" /> 
```
``` xml
    <string name="facebook_app_id" translatable="false">15793xxxxxxxxxx</string>
    <string name="fb_login_protocol_scheme" translatable="false">fb15793xxxxxxxxxx</string>
```
**（注：facebook_app_id和fb_login_protocol_scheme 在Android项目string里，需要接入者自行变更为游戏的Facebook的APPID）**
### 2.4注册登陆Receiver
在项目中的AndroidManifest中注册
``` xml
        <receiver
            android:name="游戏包名.ygapi.YGLoginReceiver"
            android:exported="true">
            <intent-filter>
                <action android:name="com.yllgame.sdk.loginReceiver" />
            </intent-filter>
        </receiver>
```
``` java
public class YGLoginReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        //Constants.BROADCAST_RECEIVER_LOGIN_ACTION SDK中登陆的action
        if (intent.getAction() == YGConstants.BROADCAST_RECEIVER_LOGIN_ACTION) {
            //拿到登陆之后的用户信息
            GameUserInfoEntity userInfoEntity = (GameUserInfoEntity) intent.getExtras().getSerializable(YGConstants.BROADCAST_RECEIVER_LOGIN_INFO_KEY);
            //该示例中通过EventBus通知并且更新主界面的更新 具体的结合自身需求修改
            if (userInfoEntity.getType() == GameUserInfoEntity.TYPE_LOGIN_ACCOUNT_SUCCESS) {
                //登陆成功
            } else if (userInfoEntity.getType() == GameUserInfoEntity.TYPE_LOGIN_ACCOUNT_FAIL) {
                //登陆失败
            } else if (userInfoEntity.getType() == GameUserInfoEntity.TYPE_FAIL_ACCOUNT_REMOTE) {
                //账号异地登录 SDK内部会有弹窗 必须退出到登陆界面清除用户信息
            } else if (userInfoEntity.getType() == GameUserInfoEntity.TYPE_FAIL_ACCOUNT_BLOCKED) {
                //账号被封 SDK内部会有弹窗 退出到登陆界面
            } else if (userInfoEntity.getType() == GameUserInfoEntity.TYPE_FAIL_TOKEN_OVERDUE) {
                //账号Token过期 退出到登陆界面
            } else if (userInfoEntity.getType() == GameUserInfoEntity.TYPE_ACCOUNT_CHANGE_USERNAME) {
                //修改昵称成功
            } else if (userInfoEntity.getType() == GameUserInfoEntity.TYPE_ACCOUNT_LOGIN_OUT) {
                //退出登录
            }
        }
    }
}
```
注：项目中所有登陆以都会通过广播通知并且在下发用户信息，YGLoginReceiver为固定写法，该广播放在项目包名.ygapi下
退出登录要退出到登陆界面并且清除本地用户信息
### 2.5注册YGReceiver
在项目中的AndroidManifest中注册
``` xml
        <receiver
            android:name="游戏包名.ygapi.YGReceiver"
            android:exported="true">
            <intent-filter>
                <action android:name="com.yllgame.sdk.payDropOrderReceiver" />
             <action android:name="com.yllgame.sdk.accountBindReceiver" />
            </intent-filter>
        </receiver>
```
``` java
public class YGReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        if (intent.getAction() == YGConstants.BROADCAST_RECEIVER_PAY_DROP_ORDER_ACTION) {
            String orderId = intent.getExtras().getString(YGConstants.BROADCAST_RECEIVER_PAY_ORDER_ID_KEY);
            LogUtils.logEForDeveloper("游戏充值订单编号：" + orderId);
        } else if (intent.getAction() == YGConstants.BROADCAST_RECEIVER_ACCOUNT_BIND_ACTION) {
            GameUserAccountBindEntity gameUserAccountBindEntity = (GameUserAccountBindEntity) intent.getExtras().getSerializable(YGConstants.BROADCAST_RECEIVER_ACCOUNT_BIND_INFO_KEY);
            /**
             *     private int isBindFacebook = 1 绑定facebook
             *     private int isBindGoogle = 1 绑定谷歌
             *     private int isBindPhone = 1 绑定手机
             *     private int isBindHuaWei = 1 绑定华为
             */
        }
    }
}
```
注：YGReceiver为固定写法，该广播放在项目包名.ygapi
### 2.6 设置allowBackup配置
``` android:allowBackup="false" ```</br>
**注：新生成的项目allowBackup为true须在项目的AndroidManifest中application设置allowBackup为false**
## 3.登陆

### 3.1登陆界面
- SDK调起登陆函数：``` YGLoginApi.getInstance().login(); ```
``` java
    /**
     * 登陆界面
     *
     * @param activity 当前Activity
     */
    public void login(Activity activity)
```
## 4.用户
### 4.1.1 管理界面
- SDK调起用户管理界面的函数：``` YGUserApi.getInstance().openAccountManager(this) ```
``` java 
    /**
     * 修改用户昵称
     *
     * @param activity
     */
    public void openAccountManager(Activity activity)
```
### 4.1.2 修改昵称
 - SDK调起修改昵称的函数：``` YGUserApi.getInstance().showUpdateNickNameDialog() ```
``` java 
    /**
     * 修改用户昵称
     *
     * @param activity
     * @param listener 修改昵称回调
     */
    public void showUpdateNickNameDialog(Activity activity, UpdateUserNameListener listener)
```
### 4.1.3 同步用户角色
 - SDK调起同步用户角色的函数：``` YGUserApi.getInstance().syncRoleInfo(); ```
``` java 
    /**
     * 同步角色信息
     *
     * @param roleId：角色id              ：int 必要参数
     * @param roleName：角色名称          ：string 必要参数
     * @param roleLevel：角色等级         ：int 必要参数
     * @param roleVipLevel：角色Vip等级   ：int 必要 没有默认0
     * @param serverId：角色所在服务器id   ：int 必要参数
     * @param roleCastleLevel：城堡等级   ：int 必要 没有默认0
     * @param callback：同步角色回调:       true 同步成功 false 同步失败
     */
    public void syncRoleInfo(String roleId,
                             String roleName,
                             String roleLevel,
                             String roleVipLevel,
                             String serverId,
                             String roleCastleLevel,
                             YGBooleanCallBack callback)
```
**注：建议游戏每次登陆、角色创建、用户角色升级，用户VIP升级必须调用该函数**
### 4.1.4检查SDK最新版本
- SDK调起用户版本更新的函数为：`` YGUserApi.getInstance().getVersionInfo() ``
 ``` java 
     /**
     * 获取版本
     */
    public void getVersionInfo()
 ```
### 4.1.5 设置界面
- SDK调起设置界面的函数为：`` YGUserApi.getInstance().showSettingsView ``
``` java
    /**
     * 设置界面
     *
     * @param activity  当前activity
     * @param serviceId 角色服务器id
     * @param roleId    角色id
     */
    public void showSettingsView(Activity activity, String serviceId, String roleId)
```
### 4.1.6展示绑定界面
- SDK展示绑定界面的函数为：`` YGUserApi.getInstance().showAccountBindView ``
  ``` java 
     /**
     * 展示绑定界面
     *
     * @param activity
     */
    public void showAccountBindView(Activity activity)
  ```
### 4.1.7账号升级提示
- SDK调起账号升级的函数为：`` YGUserApi.getInstance().checkBindStat ``
``` java 
    /**
     * 检查绑定状态 如果未绑定 则会显示账户绑定弹窗
     *
     * @param activity
     * @return 是否绑定 true 绑定 false 未绑定
     */
    public boolean checkBindStat(Activity activity)
```
### 4.1.8游戏举报
- SDK调起游戏举报的函数为：`` YGUserApi.getInstance().showReportView ``
``` java 
    /**
     * 游戏举报
     *
     * @param activity       当前activity
     * @param reportRoleId   举报者角色id
     * @param reportedRoleId 被举报者角色id
     * @param serviceId      角色服务器id
     */
    public void showReportView(Activity activity, String reportRoleId, String reportedRoleId, String serviceId)
```
### 4.1.9语聊举报
- SDK调起语聊举报的函数为：`` YGUserApi.getInstance().showRoomReportView ``
``` java 
    /**
     * 语聊举报
     *
     * @param activity          当前activity
     * @param reportType        1房间 2角色
     * @param reportRoleId      举报者角色id
     * @param reportServiceId   举报者区服Id
     * @param reportedRoleId    被举报者角色id
     * @param reportedServiceId 被举举报者区服Id
     * @param roomId            房间Id
     */
    public void showRoomReportView(Activity activity, String reportType, String reportRoleId, String reportServiceId, String reportedRoleId, String reportedServiceId, String roomId)
```
### 4.2.0 获取手机绑定状态
- SDK调起语聊举报的函数为：`` YGUserApi.getInstance().getPhoneBindState ``
``` java 
    /**
     * 获取手机绑定状态
     *
     * @param callBack true 已绑定手机 false 未绑定手机
     */
    public void getPhoneBindState(YGBooleanCallBack callBack)
```
### 4.2.1 获取活动信息
- SDK调起语聊举报的函数为：`` YGUserApi.getInstance().getActivityInfos ``
``` java 
    /**
     * 获取活动信息
     *
     * @param serviceId 服务器id
     * @param roleId    角色id
     * @param callBack  回调
     */
    public void getActivityInfos(String serviceId, String roleId, YGCallBack<List<GameActivityEntity>> callBack)
```
### 4.2.2 展示手机号绑定页面
- SDK调起语聊举报的函数为：`` YGUserApi.getInstance().showPhoneBindView ``
``` java 
    /**
     * 显示绑定界面
     *
     * @param activity          当前Activity
     * @param ygBooleanCallBack true：绑定成功
     */
    public void showPhoneBindView(Activity activity, YGBooleanCallBack ygBooleanCallBack)
```
## 5.支付
### 5.1 导入华为json文件 配置清单文件信息
- 导入agconnect-services.json文件，文件需找运营方要 </br>
![image](https://user-images.githubusercontent.com/19358621/119936162-4a397080-bfbb-11eb-9364-55d80fee0af8.png)
- 在APP的AndroidManifest.xml 中配置appid和cpid，appid和cpid在json中获取
``` xml
        <meta-data
            android:name="com.huawei.hms.client.appid"
            android:value="appid=xxx"></meta-data>
        <meta-data
            android:name="com.huawei.hms.client.cpid"
            android:value="cpid=xxx"></meta-data>
```
### 5.2 导入华为plugins
- 在APP的app/build.gradle文件的plugins目录导入华为插件
```
id 'com.huawei.agconnect'
```
### 5.3 配置华为HMS Core Maven仓库
- 在项目的build.gradle文件夹</br>
![image](https://user-images.githubusercontent.com/19358621/119936866-68ec3700-bfbc-11eb-9db9-e725f3bc63ad.png)
```
classpath 'com.huawei.agconnect:agcp:1.4.2.300'
```
### 5.4 设置华为支付回调（此函数已废弃，接入文档8.3 ）
- 在当前Activity的onActivityResult函数里面调用：`` YGPayApi.onActivityResult() ``
``` java 
     /**
     * 设置activity的回调
     *
     * @param requestCode
     * @param resultCode
     * @param data
     */
    public static void onActivityResult(int requestCode, int resultCode, Intent data)
```
### 5.5 华为支付
- SDK调起华为支付的函数为：`` YGPayApi.pay() ``
``` java 
    /**
     * 支付
     *
     * @param activity      当前activity
     * @param roleId        角色id
     * @param roleServiceId 角色服务器Id
     * @param sku           商品的sku
     * @param cpNo          支付的订单号
     * @param cpTime        支付订单的创建时间
     * @param number        支付的数量 目前为1
     * @param amount        支付的金额
     * @param pointId       支付的充值点
     * @param listener      支付的回调
     */
    public static synchronized void pay(Activity activity, String roleId,
                                        String roleServiceId, String sku,
                                        String cpNo, String cpTime,
                                        String number, String amount,
                                        String pointId, YGPaymentListener listener)
```
## 6.推送
### 6.1 配置Google 服务插件和导入google-service.json
在项目的build.gradle配置com.google.gms.google-services插件并且导入google-service.json文件
 ![image](https://user-images.githubusercontent.com/19358621/118480934-7ea06780-b745-11eb-94fc-d0048ef543da.png)
![image](https://user-images.githubusercontent.com/19358621/118481332-f1a9de00-b745-11eb-9627-58079b444310.png)

### 6.2清单文件配置推送Service和推送图标
``` xml
        <service
            android:name=".MyFirebaseMessagingService"
            android:exported="false">
            <intent-filter>
                <action android:name="com.google.firebase.MESSAGING_EVENT" />
            </intent-filter>
        </service>
        <meta-data
            android:name="com.google.firebase.messaging.default_notification_icon"
            android:resource="@drawable/yll_game_ic_logo" />
```
``` java
public class MyFirebaseMessagingService extends FirebaseMessagingService {
    @Override
    public void onMessageReceived(@NonNull RemoteMessage remoteMessage) {
        //判断是否是SDK内部消息，是的话 就返回true 返回false则表示非SDK内部消息 需游戏方自行处理
        if (!YGMessageApi.getInstance().handlePushMessage(remoteMessage))
            Log.d("FirebaseMessaging", "游戏内部消息");
        super.onMessageReceived(remoteMessage);
    }
}
```
**注：com.google.firebase.messaging.default_notification_icon 图标为透明单色图标
YGMessageApi.getInstance().handlePushMessage(remoteMessage);函数必须接入，在推送消息的service的onMessageReceived里。**
### 6.3推送通知栏点击处理
 ``` java
 public class LauncherActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (getIntent().getExtras() != null) {
            YGMessageApi.getInstance().clickPushMessage(SPStaticUtils.getRoleId(), SPStaticUtils.getRoleServiceId(), getIntent().getExtras());
        }
        setContentView(R.layout.activity_launcher);
    }
}
 ```
**注：用户点击通知栏的时候会默认进入launch的activity，在当前界面获取传递的数据并调SDK的clickPushMessage函数**
### 6.4获取推送token
-调用获取token调用方式：YGMessageApi.getInstance().getPushToken() 
 ``` java
     /**
     * 获取推送token
     *
     * @param callBack 回调的token
     */
    public void getPushToken(@NonNull YGCallBack<String> callBack)
 ```
## 7.0数据统计
### 7.1通用事件埋点（不带参数）
- 调用埋点函数为：`` YGEventApi.onEvent() ``
``` java
    /**
     * @param event_name 事件名称
     */
    public static void onEvent(String event_name) 
```
### 7.2通用事件埋点（带参数）
- 调用埋点函数为：`` YGEventApi.onEvent() ``
``` java
    /**
     * @param event_name 事件名称
     * @param params     事件额外参数
     */
    public static void onEvent(String event_name, Map params) 
```
event_name分为游戏通用埋点和自定义埋点的事件名称
通用事件埋点event_name 都需要按规定书写
由YGEventConstants提供，列如YGEventConstants.YG_GAME_UPDATE_BEGIN

自定义事件埋点event_name 由游戏方约束
详情文档可参考[Yll_Android埋点](https://github.com/yllgame2021/yllgamesdk/blob/master/%E5%9F%8B%E7%82%B9%E9%9C%80%E6%B1%82/Android/%E7%BB%9F%E8%AE%A1%E5%9F%8B%E7%82%B9Android.md)
### 7.3Firebase埋点事件
- 调用埋点函数为：`` YGEventApi.onFireBaseEvent() ``
``` java
    /**
     * Facebook事件
     *
     * @param eventName 事件名称
     * @param params    事件参数 没有参数直接传null
     */
    public static void onFireBaseEvent(String eventName, Map params)
```
### 7.4Facebook埋点事件
- 调用埋点函数为：`` YGEventApi.onFacebookEvent() ``
``` java
    /**
     * Facebook事件
     *
     * @param eventName 事件名称
     * @param params    事件参数 没有参数直接传null
     */
    public static void onFacebookEvent(String eventName, Map params)
```
### 7.5AppsFlyer埋点事件
- 调用埋点函数为：`` YGEventApi.onAppsFlyerEvent() ``
``` java
    /**
     * AppsFlyer事件
     *
     * @param eventName 事件名称
     * @param params    事件参数 没有参数直接传null
     */
    public static void onAppsFlyerEvent(String eventName, Map params)
```
### 7.6所有三方埋点事件
- 调用埋点函数为：`` YGEventApi.onThirdEvent() ``
``` java
    /**
     * 上报Facebook&Firebase&AppsFlyer平台事件
     *
     * @param eventName 事件名称
     * @param params    事件参数 没有参数直接传null
     */
    public static void onThirdEvent(String eventName, Map params)
```
## 8.0其它
### 8.1获取SDK 版本号名称
- 调用获取SDK函数为：`` YGCommonApi.getSDKVersionName() ``
``` java
     /**
     * @return 返回版本名称
     */
    public static String getSDKVersionName()
```
### 8.2获取SDK 版本号Code
- 调用埋点函数为：``YGCommonApi.getSDKVersionCode() ``
``` java
    /**
     * @return 返回版本号
     */
    public static int getSDKVersionCode()
```
### 8.3设置activity回调（必接）
- 调用设置activity函数为：``YGCommonApi.setCallback() `` 
``` java
    /**
     * 设置回调
     *
     * @param requestCode
     * @param resultCode
     * @param data
     */
    public static void setCallback(int requestCode, int resultCode, Intent data)
```
注：该函数必须接入，有些第三方依赖库必须依赖Activity的onActivityResult的回调来传值和回调。在Activity 重写onActivityResult
``` java
    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        YGCommonApi.setCallback(requestCode, resultCode, data);
        super.onActivityResult(requestCode, resultCode, data);
    }
```
## 9.0弱联网
### 9.1设置弱联网模式
- 调用设置弱联网函数为：``  YllGameSdk.setNetMode(int mode); ``
``` java
     /**
     * 设置SDK联网默认模式 默认强联网模式
     *
     * @param mode  YGConstants.SDK_STRONG_NET 强联网 YGConstants.SDK_WEAK_NET 弱联网
     */
    public static void setNetMode(int mode)
```
## 10 Facebook好友列表和分享（必须接入8.3）
###  AndroidManifest.xml
``` xml
        <!--将 ContentProvider 添加至 AndroidManifest.xml 文件，并将 {APP_ID} 设置为您的应用编号：-->
        <provider
            android:name="com.facebook.FacebookContentProvider"
            android:authorities="com.facebook.app.FacebookContentProvider{APP_ID}"
            android:exported="true" />
        <!--如果您的应用程序面向 Android 11 或更高版本，请向 AndroidManifest.xml 文件添加以下查询块，以使 Facebook 应用对您的应用可见-->
        <queries>
            <provider android:authorities="com.facebook.katana.provider.PlatformProvider" />
        </queries>
```
### 10.1获取Facebook好友列表
- 获取Facebook好友列表的函数为：`` YGTripartiteApi.getInstance().getFacebookFriends ``
``` java 
    /**
     * 获取Facebook好友
     *
     * @param activity 当前的activity
     * @param callBack 好友列表回调
     */
    public void getFacebookFriends(Activity activity, YGCallBack<List<GameFacebookFriendEntity>> callBack)
```
### 10.2Facebook分享链接
- Facebook分享链接的函数为：`` YGTripartiteApi.getInstance().shareLink ``
``` java 
    /**
     * 分享链接
     *
     * @param activity 当前的activity
     * @param quote    分享的标题
     * @param url      分享的url
     * @param callback 分享回调
     */
    public void shareLink(Activity activity, String quote, String url, FacebookCallback<Sharer.Result> callback)
```
### 10.2Facebook分享链接
- Facebook分享链接的函数为：`` YGTripartiteApi.getInstance().shareLink ``
``` java 
    /**
     * 分享链接
     *
     * @param activity 当前的activity
     * @param quote    分享的标题
     * @param url      分享的url
     * @param callback 分享回调
     */
    public void shareLink(Activity activity, String quote, String url, FacebookCallback<Sharer.Result> callback)
```
### 10.3Facebook公共页面
- Facebook公共页面：`` YGTripartiteApi.getInstance().openFacebookPage ``
``` java 
    /**
     * Facebook公共页面
     *
     * @param pageId 账号id
     */
    public void openFacebookPage(Activity activity,String pageId)
```
### 10.4阿里云上传图片
文件读写权限获取：
``` java 
    /**
     * 检查是否拥有打开裁剪所需的所有权限
     *
     * @param context 上下文
     * @return true：已拥有，false：存在未拥有的权限
     */
    public boolean checkCropPermission(Context context)
```
``` java 
    /**
     * 申请裁剪所需权限
     * 申请回调请通过传入的activity中的{@link android.app.Activity#onRequestPermissionsResult}方法判断
     *
     * @param activity activity
     */
    public void requestCropPermissions(Activity activity)
```
- 阿里云上传图片的函数为：`` YGTripartiteApi.getInstance().updatePicture ``
``` java 
        /**
     * 阿里云上传图片
     * @param activity  当前Activity
     * @param clipRatio 裁剪比例, {@link YGConstants#CLIP_RATIO_NONE} 表示不裁剪 {@link YGConstants#CLIP_RATIO_1_1} 裁剪
     * @param callBack  上传回调返回图片绝对路劲
     */
    public void updatePicture(Activity activity, int clipRatio, YGCallBack<String> callBack)
```
代码示例
``` java 

    private void start() {
        if (YGTripartiteApi.getInstance().checkCropPermission(this))
            startUpload();
        else
            YGTripartiteApi.getInstance().requestCropPermissions(this);
    }

    private void startUpload() {
        YGTripartiteApi.getInstance().updatePicture(this, new YGCallBack<String>() {
            @Override
            public void onSuccess(String s) {
            }

            @Override
            public void onFail(int code) {
            }
        });
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        if (requestCode == YGConstants.CAMERA_CROP_PERMISSION) {
            if (grantResults.length == 1 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                startUpload();
            }
        }
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        YGCommonApi.setCallback(requestCode, resultCode, data);
        super.onActivityResult(requestCode, resultCode, data);
    }
```
