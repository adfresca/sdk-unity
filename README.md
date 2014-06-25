## Contents
- [Basic Integration](#basic-integration)
    - [Installation](#installation)
    - [Start Session](#start-session)
    - [In-App Messaging](#in-app-messaging)
    - [Push Messaging](#push-messaging)
    - [Test Device Registration](#test-device-registration)
- [IAP & Reward](#iap--reward)
  - [In-App Purchase Tracking (Beta)](#in-app-purchase-tracking-beta)
  - [Give Reward](#give-reward)
- [Dynamic Targeting](#dynamic-targeting)
  - [Custom Parameter](#custom-parameter)
  - [Marketing Moment](#marketing-moment)
- [Advanced](#advanced)
  - [Timeout Interval](#timeout-interval) 
- [Reference](#reference)
  - [Custom URL Schema](#custom-url-schema)
  - [Cross Promotion Configuration](#cross-promotion-configuration)
  - [Proguard Configuration](#proguard-configuration)
- [Troubleshooting](#troubleshooting)
- [Release Notes](#release-notes)

* * *

## Basic Integration

### Installation

Download _Unity Plugin_ at the following link.

[Unity Plugin v2.1.8 Download](https://s3-ap-northeast-1.amazonaws.com/file.adfresca.com/distribution/sdk-for-Unity.zip) (Android SDK v2.3.4, iOS SDK v1.3.5)

[Unity Plugin with IAP Tracking BETA v2.2.0-beta3 Download](https://s3-ap-northeast-1.amazonaws.com/file.adfresca.com/distribution/sdk-for-Unity-iap-beta.zip) (Android SDK v2.4.0-beta4, iOS SDK v1.3.5)

Open your Unity Project and run AdFrescaUnityPlugin.package. It will install following components below

Assets/

    LitJson.dll

Assets/AdFresca/

    Plugin.cs 
    AndroidPlugin.cs
    IOSPlugin.cs
    RewardItem.cs
    Purchase.cs // only for IAP BETA

Assets/Plugins/Android/

    AdFresca.jar 
    AdFrescaPlugin.jar 
    gcm.jar 
    assets 

Assets/Plugins/iOS/

    AdFrescaPlugin.h
    AdFrescaPlugin.mm

Now, we start some installation works for each platform

#### Android

For Android, we have already done with most of installation from package file. You just need to modify AndroidManifest.xml as below

##### Modify AndroidManifest.xml

```xml
<manifest package="your.app.package">
  <application>
    <activity>
    <!-- Enable ForwardNativeEventsToDalvik -->
    <meta-data android:name="unityplayer.ForwardNativeEventsToDalvik" android:value="true" />
    </activity>
    
    <!-- Service for OpenUDID -->
    <service android:name="org.openudid.OpenUDID_service">
      <intent-filter>
        <action android:name="org.openudid.GETUDID" />
      </intent-filter>
    </service>

    <!-- Activity for Reward -->
    <activity android:name="com.adfresca.sdk.reward.AFRewardActivity" />
   
    <!-- Boradcast Receiver for Google Referrer Tracking-->
    <receiver android:name="com.adfresca.sdk.referer.AFRefererReciever" android:exported="true">
      <intent-filter>
        <action android:name="com.android.vending.INSTALL_REFERRER" />
      </intent-filter>
    </receiver>
  </application>
  
  <!-- Permissions -->
  <uses-permission android:name="android.permission.INTERNET"/>
  <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
</manifest>
```

#### iOS

For iOS, Unity requires to do the same installation work as a native project. After checking all the components from package file, build and export Xcode project from Unity.

Install iOS SDK with following ['iOS SDK Installation'](https://github.com/adfresca/sdk-ios/blob/master/README.md#installation) section of our iOS SDK guide.

### Start Session

Now, start to put some simple SDK codes in your app. You first need to call startSession() method with setting your API Key. To get your API Key, go to our [Dashboard](https://admin.adfresca.com) and then click 'Settings - API Keys' button in your app's 'Overview' page.

#### Android

Put StartSession() method when users start an app. Please make sure that this method is called once while your app is running.

```cs
#if UNITY_ANDROID
private static string API_KEY = "YOUR_ANDROID_API_KEY";
#elif UNITY_IPHONE
private static string API_KEY = "YOUR_IOS_API_KEY";
#endif

void Start ()
{
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.Init(API_KEY);
  plugin.StartSession();
}
```

#### iOS

For iOS, you need to put the native SDK codes in your Xcode project. Open AppController.mm file and modify didFinishLaunchingWithOptions() event as follows.

```objective-c
#import <AdFresca/AdFrescaView.h>

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  [AdFrescaView startSession:@"YOUR_API_KEY"];
  ....
} 

```

### In-App Messaging

With the in-app messaging feature, you can deliver a message to your in-app users in real time. Simply put 'Load()' and 'Show()' methods where you want to deliver a message. The type of message can be an interstitial image, text, and iframe webpage. The message is only shown when your user matches the in-app messaging campaign's target logics. We will discuss more details of the in-app messaging's dynamic targeting features in the [Dynamic Targeting](#dynamic-targeting) section.

```cs
void Start ()
{
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.Load();
  plugin.Show();
}
```

When you first call in-app messaging methods, you will see the test message below. If you tap on the image, it will redirect to the product page of the app on the app store. You will hide this test message by chagning the test mode configuration later.

<img src="https://adfresca.zendesk.com/attachments/token/ans53bfy6mwq2e9/?name=4444.png" width="240" />
&nbsp;
<img src="https://adfresca.zendesk.com/attachments/token/ec7byt0qtj00qpb/?name=5555.png" height="240" />

### Push Messaging

You can also deliver your push messages anytime you want. Follow the steps below to configure the push notification settings in your app.

#### Android

Before you start, you need to have your GCM project number from Google API Console, and set GCM API Key value to our [Dashboard](https://admin.adfresca.com). Please refer to '[Android Push Notification Guide](https://adfresca.zendesk.com/entries/28526764)'

1) Add some codes to AndroidManifest.xml

```xml
<manifest>   
  <application>
      .........
      <receiver android:name="YOUR.PACKAGE.NAME.GCMReceiver"
        android:permission="com.google.android.c2dm.permission.SEND">  
        <intent-filter>
          <action android:name="com.google.android.c2dm.intent.RECEIVE" />
          <action android:name="com.google.android.c2dm.intent.REGISTRATION" />
          <category android:name="YOUR.PACKAGE.NAME" />
         </intent-filter>
      </receiver>
      <service android:name="YOUR.PACKAGE.NAME.GCMIntentService" />  

      <activity android:name="com.adfresca.ads.AdFrescaPushActivity" />
      ..........
   </application>
    ..........
    <permission android:name="YOUR.PACKAGE.NAME.permission.C2D_MESSAGE" android:protectionLevel="signature" />
    <uses-permission android:name="YOUR.PACKAGE.NAME.permission.C2D_MESSAGE" />
    <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
    <uses-permission android:name="android.permission.GET_ACCOUNTS" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />

    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
    <uses-permission android:name="android.permission.READ_PHONE_STATE" /> 
    <uses-permission android:name="android.permission.VIBRATE" />
    ..........
</manifest>
```

- If you already have your own GCMReceiver and GCMIntentService classes, you only need to put some SDK codes in your own GCMIntentService class.
- If you haven't implemented any GCM classes yet, you will need to write your own GCM classes in java code. Please use our 'Android Plugin Project' in our unity plugin folder. After importing the sample project in Eclipse ADT, you will need to rename packages in **/src** and **/gen** to your own package name. Also make sure you need to rename 'YOUR.PACKAGE.NAME' in AndroidManifest.xml above.

2) Implement CustomGCMIntentService

```java
public CustomGCMIntentService() {
  super();
}

@Override
protected String[] getSenderIds (Context context) {
  String[] ids = {AdFrescaPlugin.gcmSenderId};
  return ids;
}

@Override
protected void onRegistered(Context context, String registrationId) {
  AdFresca.handlePushRegistration(registrationId);
}

@Override
protected void onUnregistered(Context context, String registrationId) {
  AdFresca.handlePushRegistration(null);
}

@Override
protected void onMessage(Context context, Intent intent) {
  // Check AD fresca notification
  if (AdFresca.isFrescaNotification(intent)) {    

    String title = AdFrescaPlugin.getAppName(context);
    int icon = com.MyCompany.ProductName.R.drawable.app_icon;
    long when = System.currentTimeMillis();
    Class<?> targetActivityClass = null;
    
    if (UnityPlayer.currentActivity != null) {
      targetActivityClass = UnityPlayer.currentActivity.getClass();
    } else {
      targetActivityClass = UnityPlayerProxyActivity.class; // or YourUnityPlayerProxyActivity.class
    }
    
    AFPushNotification notification = AdFresca.generateAFPushNotification(context, intent, targetActivityClass, appName, icon, when);
    notification.setDefaults(Notification.DEFAULT_ALL); 
    AdFresca.showNotification(notification);
  }                
}  
```

3) Implement CustomGCMReceiver

```java
protected String getGCMIntentServiceClassName(Context context) { 
  return "YOUR.PACKAGE.NAME.CustomGCMIntentService"; 
} 
```

4) Export Jar

After updating two classes above, you need to export them as a single jar file into the unity project (/Assets/Plugins/Android/). Please make sure you should only export these two classes, not the entire package.

<img src="https://s3-ap-northeast-1.amazonaws.com/file.adfresca.com/guide/sdk/unity/unity-android-gcm-export.png"/>

5) Unity Code

Now, you add a single line of unity code to use GCM. Add 'SetGCMSenderId(GCM_PROJECT_ID)' method to set Google API Project number. Be carful that project number is not API Key value.

```cs
#if UNITY_ANDROID
private static string GCM_PROJECT_ID = "12345678"; // Google API Proejct Number (ex: 12345678)
#endif

void Start ()
{
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.Init(API_KEY);
  
  plugin.SetGCMSenderId(GCM_PROJECT_ID);
  plugin.StartSession();
}
```

#### iOS

1. Upload your APNS Certificate file (.p12) to our Dashboard
  - You can export your .cer file to .p12 file using Keychain. Please refer to [iOS Push Notification Certificate Guide](https://adfresca.zendesk.com/entries/21714780) to generate .p12 and upload to [Dashboard](https://admin.adfresca.com)

2. Check your provisioning
  - AD fresca only supports APNS production environment. So, you should build your app with App Store or Ad Hoc Provisioning file to enable production mode

3. Add some codes to AppController.mm in Xocde Project 

```mm
#import <AdFresca/AdFrescaView.h>

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  ...
  [[UIApplication sharedApplication] registerForRemoteNotificationTypes:UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge |UIRemoteNotificationTypeSound];   
} 

- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
  [AdFrescaView registerDeviceToken:deviceToken];
}

- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
  if ([AdFrescaView isFrescaNotification:userInfo] && [application applicationState] != UIApplicationStateActive) {
    [AdFrescaView handlePushNotification:userInfo];
  }
} 
```

So, we are now done with the push messaging implementation!

### Test Device Registration

AD fresca supports a test mode feature. With the test mode feature, you can deliver your test message to only registred test devices. 

To register your test device to our dashboard, you need to know your test device ID from our SDK. SDK provides two ways to show test device ID.
 
1) Using PrintTestDeviceIdByLog() method

```cs
AdFresca.Plugin plugin = AdFresca.Plugin.Instance
plugin.Init(API_KEY);
plugin.StartSession();

if(Application.platform == RuntimePlatform.Android) {
  plugin.PrintTestDeviceIdByLog();
} else {
  string testDeviceId = plugin.TestDeviceId();
  Debug.Log("testDeviceId = " + testDeviceId);
}
```

2) Displaying test device ID on your app screen using SetPrintTestDeviceId(bool) method.

```cs
AdFresca.Plugin plugin = AdFresca.Plugin.Instance
plugin.Init(API_KEY);
plugin.StartSession();
plugin.SetPrintTestDeviceId(true);
plugin.Load();
plugin.Show();
```

After you have your test device ID, you have to register it to [Dashboard](https://admin.adfresca.com). You can register your device in the 'Test Device' menu.

* * *

## IAP & Reward

### In-App Purchase Tracking (Beta)

(Android Only)

With In-App-Purchase Tracking , you can analyze all the purchases of your users, and use it for targeting specific user segment to display your campaigns. (targeting feature is coming soon)

There are two types of purchases you can track with our SDK.

1. **Actual Item Purchase Tracking:**  the purchases made by real money. For example, user purchased 'USD 1.99' to get 'Gold 100' cash item.
2. **Virtual Item Purchase Tracking:** the purchases made by virtual money. For example, user purchased 'Gold 10' to get 'Rocket Launcher' item 

You don't need to write down any item list manually. All the Items tracked by SDK are automatically added to our dashboard. To see the list of item, go to 'Overview > Settings > In-App Items' page in our dashboard.

Let's get started to implement SDK codes with examples below. 

### Actual Item Tracking

The purchase of 'Actual Item' is made with the store's billing library such as Google Play Billing. When your user purchased the item successfully, simply create Purchase object and use LogPurchase() method.

Example 1: When you implement 'purchase succeeded' event in Unity 
```cs
AdFresca.Purchase purchase = new AdFresca.PurchaseBuilder(AdFresca.Purchase.Type.ACTUAL_ITEM)
  .WithItemId("gold100")
  .WithCurrencyCode("USD") // The currencyCode must be specified in the ISO 4217 standard. (ex: USD, KRW, JPY)
  .WithPrice(0.99)
  .WithPurchaseDate(purchaseDateTime) // purchaseDateTime from In-app billing library
  .WithReceipt("google_play_order_id", "google_play_receipt_json", "google_play_signature"); // Optional
      
AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
plugin.LogPurchase(purchase);
```

Example 2: When you implement Google Billing Library in the native Android codes.

```java
// Callback for when a purchase is finished
IabHelper.OnIabPurchaseFinishedListener mPurchaseFinishedListener = new IabHelper.OnIabPurchaseFinishedListener() {
  public void onIabPurchaseFinished(IabResult result, Purchase purchase) {
    Log.d(TAG, "Purchase finished: " + result + ", purchase: " + purchase);

    if (mHelper == null || result.isFailure() || !verifyDeveloperPayload(purchase)) {
      ......
      return;
    }

    Log.d(TAG, "Purchase successful.");
    if (purchase.getPurchaseState() == 0) {
      final SkuDetails detail = currentInventory.getSkuDetails(purchase.getSku());
      final Purchase purchase0 = purchase;
      
      UnityPlayer.currentActivity.runOnUiThread(new Runnable(){
        @Override
        public void run() {
          String itemId = purchase0.getSku();
          String currencyCode = "KRW"; // The currencyCode must be specified in the ISO 4217 standard. (ex: USD, KRW, JPY)
          Double price =  parsePrice(detail.getPrice()); // For Google Play, you can get the price value from SkuDetails
          Date purhcaseDate = new Date(purchase0.getPurchaseTime());
          String orderId = purchase0.getOrderId();
          String receiptData = purchase0.getOriginalJson();
          String signature = purchase0.getSignature();

          AFPurchase actualPurchase = new AFPurchase.Builder(AFPurchase.Type.ACTUAL_ITEM)
                                .setItemId(itemId)
                                .setCurrencyCode(currencyCode)
                                .setPrice(price)
                                .setPurchaseDate(purhcaseDate)
                                .setReceipt(orderId, receiptData, signature)
                                .build();

          AdFresca.getInstance(UnityPlayer.currentActivity).logPurchase(actualPurchase);
        }
      });
    }
    
    ......
    }
};
```

The above example is written for Google Play. You can also get the required values form other billing library such as Amazon.

For more details of Purchase object with the actual item, check the table below.

Method | Description
------------ | ------------- | ------------
WithItemId(string) | Set the unique identifier of your item. This value is may not be different per the os platform or app store. We recommend that you make this value unique for all platforms and stores. Our service distinguish each item by this value.
WithCurrencyCode(string) | Set the current code of IOS 4217 standard. For Google Play, use the currency code of 'default price' in your account. For Amazon, set 'USD' value since Amazon only supports USD.
WithPrice(double) | Set the item price. you may use SkuDetails's value or manually set the value from your server.
WithPurchaseDate(date) | Set the date of purchase. You may use purchase.getPurchaseTime() value. If you set null value, it will be automatically recorded by our SDK and server. Please don't use local time of user's device.
WithReceipt(string, string, string) | Set the receipt property of purchase object (Google Play only). We will use it to verify the receipt in the future. 

### Virtual Item Tracking

When users purchased your virtual item in the app, you can also create Purchase object and call LogPurchase() method.

```cs
AdFresca.Purchase purchase = new AdFresca.PurchaseBuilder(AdFresca.Purchase.Type.VIRTUAL_ITEM)
  .WithItemId("long_sword")
  .WithCurrencyCode("gold") 
  .WithPrice(100)
  .WithPurchaseDate(purchaseDateTime);

AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
plugin.LogPurchase(purchase);
```

For more details of Purchase object with the virtual item, check the table below.

Method | Description
------------ | ------------- | ------------
WithItemId(string) | Set the unique identifier of your item. This value is may not be different per the os platform or app store. We recommend that you make this value unique for all platforms and stores. Our service distinguish each item by this value.
WithCurrencyCode(string) | Set the item's virtual currency code. (ex: 'gold', 'gas')
WithPrice(double) | Set the item price. You may get this value from your server. (ex: 100 of gold)
WithPurchaseDate(date) | Set the date of purchase. If you set null value, it will be automatically recorded by our SDK and server. Please don't use local time of user's device.

### IAP Trouble Shooting

After you call LogPurchase() method, the purchase data is updated to our dashboard in real-time. You can see the list of updated item in 'Overview > Settings > In-App Items' menu.

If you can't see any data in our dashboard, your Purchase object may be invalid. Check your Android logs as our plugin is printing error messages. The log format will be "AFPurchaseExceptionListener.onException = {error message}".

* * *

### Give Reward

When you set 'Reward Item' section of the announcement campaign or 'Inventive item' section of the incentivized CPI & CPA campaign, you should implement this 'reward item' code to give an reward item to your users.

Implementing reward item codes, you can check if your user has any reward to receive, and then will be noticed with an reward item info.

To implement codes, we use two codes below:
- CheckRewardItems(): this method is to check if any item is available to receive. we recommend to put this code when app becomes active. 
- SetAndroidRewardItemListener(): when the reward condition is completed with current user, onReward event is automatically called with an item value from Android SDK. For iOS, you will need some work as follows in Xcode. Then you can give an item to the user with RewardItem object.

**Implement AFRewardItemDelegate for iOS**

```objective-c
// UnityAppController.h

@interface UnityAppController : NSObject<UIApplicationDelegate, AFRewardItemDelegate>
{
  ...
}

```

```objective-c
// UnityAppController.mm

- (BOOL)application:(UIApplication*)application didFinishLaunchingWithOptions:(NSDictionary*)launchOptions
{
  ...
  [[AdFrescaView shardAdView] setRewardDelegate:self];
}

- (void)itemRewarded:(AFRewardItem *)item 
{
  UnitySendMessage("YourGameObject", "OnReward", [[item JSON] UTF8String]);
}

```

**Unity Code**

```cs
void Start ()
{
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.Init(API_KEY);
  plugin.SetGCMSenderId(GCM_SENDER_ID);    
  plugin.StartSession();

  plugin.SetAndroidRewardItemListener("YourGameObject", "OnReward");
  plugin.CheckRewardItems();
}

public void OnReward(string json)
{
  RewardItem rewardItem = LitJson.JsonMapper.ToObject<RewardItem>(json);
 
  Debug.Log ("rewardItem.name: " + rewardItem.name);
  Debug.Log ("rewardItem.uniqueValue: " + rewardItem.uniqueValue);

  // 아이템 고유 값 'uniqueValue'을 이용하여 사용자에게 아이템 지급
  SendItemToUser(rewardItem.uniqueValue);
}
```

You will implement your own 'SendItemToUser()' method. This method may send the current user info and item's uniqueValue to your server. Then server gives the item to the user.

onReward event is called when each type of campaign's reward condition is completed.

- Announcement Campaign: the event is called when your user see the campaign contents
- Incentivized CPI Campaign: the event is called when SDK checks Advertising App's install
- Incentivized CPA Campaign: the event is called after SDK checks Advertising App's install and the user called the targeted marketing moment in Advertising App

If your users have any network disconnection or loss in theirs device, our SDK stored the reward data in the app's local storage, and then re-check in the next app session. So, we guarantee users will always get the reward from our SDK.

* * *

## Dynamic Targeting

### Custom Parameter

Our SDK can collect user specific profiles such as level, stage, maximum score and etc. We use it to deliver a personalized and targeted message in real time to specific user segment that you can define.

To implement codes, simply call SetCustomParameter method with passing parameter's index and value. You can get the custom parameter's index in our [Dashboard](https://admin.adfresca.com): 1) Select a App 2) In 'Overview' menu, click 'Settings - Custom Parameters' button.

You will call the method after your app is launched and the values have changed. 

```cs
void Start() {
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.Init(API_KEY);
  plugin.SetCustomParameter(CUSTOM_PARAM_INDEX_LEVEL, User.level);
  plugin.SetCustomParameter(CUSTOM_PARAM_INDEX_AGE, User.age);
  plugin.SetCustomParameter(CUSTOM_PARAM_INDEX_HAS_FB_ACCOUNT, User.hasFacebookAccount);
  plugin.StartSession();
}

.....

void OnUserLevelChanged(int level) {
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.SetCustomParameter(CUSTOM_PARAM_INDEX_LEVEL, level);
}

void OnUserStageChanged(int stage) {
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.SetCustomParameter(CUSTOM_PARAM_INDEX_STAGE, stage);
}
```

* * *

### Marketing Moment

Marketing Moment means the moment you want to engage with your users. For example, you may need to deliver the message when the user completes a quest or enters an item store. You will be able to use it with the [custom parameters](#custom-parameter) so you can deliver the personalized and targeted message in specific moment in real time.

To implement codes, simply call **Load(index)** method with passing marketing moment's index. You can get the marketing moment's index in our [Dashboard](https://admin.adfresca.com): 1) Select a App 2) In 'Overview' menu, click 'Settings - Marketing Moment' button. 

You will call the method after the moment has happened in the app.

```cs
  void OnUserDidEnterItemStore() 
  {
    AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
    plugin.Load(EVENT_INDEX_STORE_PAGE); 
    plugin.Show();
  }

  void OnUserLevelChanged(int level) 
  {
    AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
    plugin.SetCustomParameter(CUSTOM_PARAM_INDEX_LEVEL, level); 
    plugin.Load(EVENT_INDEX_LEVEL_UP); 
    plugin.Show();
  }
```

## Advanced

### Timeout Interval

You can set a timeout interval for messaging request. If message is not loaded within this time interval, Message won't be displayed to users and SDK will return the control to your app.

Default is 5 seconds and you can set from 1 seconds to 5 seconds.

```cs
AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
plugin.SetTimeoutInterval(5);
plugin.Load();
plugin.Show();
```

* * *

## Reference

### Custom URL Schema

You can set your own URL Schema as 'Click URL' of the campaigns. So, you can navigate your users to the specific page or do some custom actions when user clicked the image message. 

#### Use Custom URL for Android

In the native android application, you can simply add schema information to AndroidManifest.xml to use custom url. However, unlike the native android application that uses multiple activities as its pages, Unity engine uses only one player activity and implements engine's own paginations internally. So, there is a problem to add schema information since you cannot set schema on UnityPlayer activity.

To solve this issue, you need to do some extra works as below.

1) Override startActivity(intent) of UnityPlayer Activity to handle Custom URL for Announcement Campaign.

Click URL from Announcement Campaign is always executed on in-game situation. It is never executed from outside of game like a push notification. Also, SDK uses startActivity() method to execute url. Therefore, you can manually handle urls by overriding startActivity() of UnityPlayer activity. 

Firstly, you should create a new Android Project in Eclipse and generate a class named MainActivity and inherited from UnityPlayerActivity. Then, modify AndroidMenefest.xml as below.

```xml
<application android:icon="@drawable/app_icon" android:label="@string/app_name" android:debuggable="true">
  <activity android:name="com.Company.ProductName.MainActivity" android:label="@string/app_name">
    <intent-filter>
      <action android:name="android.intent.action.MAIN" />
      <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
  </activity>
  ..... 
</application>
```

Now, you implement startActivity() method of MainActivity. if custom url with 'myapp://" schema is received, it will pass uri string value to your unity game object.

```java
public class MainActivity extends UnityPlayerActivity {
  ...
  @Override 
  public void startActivity(Intent intent) { 
    boolean isStartActivity = true;

    // Check intent 
    Uri uri = intent.getData(); 
    if (uri != null && uri.getScheme().equals("myapp")) { 
      isStartActivity = false; 
    }

    if (isStartActivity) { 
      super.startActivity(intent); 
    } else { 
      // do something with UnitySendMessage and uri 
      Log.d("TEST", "MainActivity.startActivity() : uri = " + uri.toString());    
      UnityPlayer.UnitySendMessage("Fresca", "OnCustomURL", uri.toString());
    } 
  }
}
```

2) Handle custom url form Push Notification Campaign

When your app receive a push notification with custom url, you can execute your own custom action. A notification is mostly received when user is outside of game. So, we should handle custom url with a little bit different approach. 

Firstly, create a new activity class named 'PushProxyActivity', and register the activity in AndroidMenefest.xml as below

```xml
<activity android:name=".PushProxyActivity">
  <intent-filter> 
    <action android:name="android.intent.action.VIEW" /> 
    <category android:name="android.intent.category.DEFAULT" /> 
    <category android:name="android.intent.category.BROWSABLE" /> 
    <data android:scheme="myapp" android:host="com.adfresca.push" />
  </intent-filter> 
</activity>

.......
```
In this case, you should create custom url like myapp://com.adfresca.push?item=abc in your Push Notification Campaign. 

Then, you should implement PushProxyActivity class. This class is a simple proxy-style activity which only handles url form Android OS and then quits itself. 
However, there is a exceptional situation when a notification is received and your application is not running. In that case, you can't handle custom url in the game engine, so you should manually start your game and pass url to MainActivity as below.

```java
public class PushProxyActivity extends Activity {
  @Override
  public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    
    // hide ui
    requestWindowFeature(Window.FEATURE_NO_TITLE);
    getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,WindowManager.LayoutParams.FLAG_FULLSCREEN);
    getWindow().setBackgroundDrawableResource(android.R.color.transparent);
            
    Uri uri = getIntent().getData();
    if (UnityPlayer.currentActivity != null) { 
      Log.d("TEST", "PushProxyActivity.onCreate() with currentActivity : uri = " + uri.toString());   
      UnityPlayer.UnitySendMessage("Fresca", "OnCustomURL", uri.toString()); 
      
     } else {
       Log.d("TEST", "PushProxyActivity.onCreate() without currentActivity : uri = " + uri.toString());   
       
       // Start a new player with uri
       try {
         Intent intent = new Intent(this, MainActivity.class);
         intent.putExtra("fresca_uri", uri.toString());
         intent.setFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP | Intent.FLAG_ACTIVITY_SINGLE_TOP);
         startActivity(intent);
       } catch (Exception e) {
         e.printStackTrace();
       }
    }
    
    finish();
  }
}
```
Finally, you should handle url from PushProxyActivity in your MainActivity.

```java
public class MainActivity extends UnityPlayerActivity {
  @Override
  public void onCreate(Bundle savedInstanceState) {
    ......
    // Handle custom uri from PushProxcyActivity
    String frescaURL = this.getIntent().getStringExtra("fresca_uri");
    if (frescaURL != null) {
      Log.d("TEST", "MainActivity.onCreate() with uri");  
      UnityPlayer.UnitySendMessage("Fresca", "OnCustomURL", frescaURI);
    }   
    ......
  }
  .........
}
```

#### Use custom url in iOS

For iOS, it is much easier to use custom url since there is only one event method for url handling.

1) Set your custom url schemes in Info.plst as follows

<img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png" />

2) In AppController.mm, implement handleOpenURL method. You should pass url string to your unity game object.

```mm
- (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url 
{  
  if ([url.scheme isEqualToString:@"myapp"])
  {
        NSString *frescaURL = [url absoluteString];
        UnitySendMessage("Fresca", "OnCustomURL", [frescaURL UTF8String]);
  }
  return YES;
}
```

#### Use custom url in Unity

Both example codes above have passed url strings to 'OnCustomURL' method of 'Fresca' game object. Now you can handle url values in Unity.

```java
public void OnCustomURL(string url)
{
  Debug.Log("OnCustomURL = " + url); 
  //ex) if myapp://com.adfresca.custom?item=abc is passed, parse 'item=abc' string to give item to users
}
```

* * *

### Cross Promotion Configuration

Using Incentivized CPI & CPA Campaign, your users in 'Media App' can get an incentive item when they install 'Adverting App' from the campaigns.

- Medial App: the media app which displays the promotion image and gives an incentive item to users
- AdvertisingApp: the promotion app which is displayed with an image in the media app's screen.

For more details of Incentivized campaigns and configuration guide in dashboard, please refer 'Understanding Cross-promotion (Korean)'  guide.

To integrate SDK with this feature, you should set URL Schema value for the adverting app and implement codes to give an incentive item to users in the media app.

#### Advertising App Configuration:

  1. Android

  For Android, SDK uses the package name to check if the advertising app is installed in the same device. 

  You can find the package name in AndroidManifest.xml

  ```xml
  <manifest package="com.adfresca.demo">
    ...
  </manifest>
  ```

  In this case, you should set CPI Identifier value of advertising app to "com.adfresca.demo" in our dashboard.

  2. iOS

  For iOS, SDK uses URL Schema value to check the advertising app's installation. 

  Open Info.plst in Xocde to check your value.

  <img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png"/>

  In this case, you should set CPI Identifier value of advertising app to "myapp://" in our dashboard. For iOS, url schema value may be duplicated with other apps, so be careful to choose unique value to run CPI Campaign..

  For Incentivized CPI Campaign, SDK Installation of the advertising app is not required. You only check the package name. 

  However, If you use Incentivized CPA Campaign, SDK installation is required and you should also implement 'Marketing Moment' feature to check a reward condition. For example, when you set the reward condition to check 'Tutorial Complete' event, you should call the marketing moment method to inform your user achieved the goal.

  ```cs
  void OnUserFinishTutorial() 
  {
    AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
    plugin.Load(MOMENT_INDEX_TUTORIAL);  
    plugin.Show();
  }
  ```

#### Media App Configuration:

  To give an incentive item to the media app's users, please refer to the [Give Reward](#give-reward) section.

* * *

### Proguard Configuration

If you use Proguard to protect your APK, you should add exception configurations for our Android SDK. Add following lines of codes to ignore our SDK, OpenUDID, and Google Gson. 

```java
-keep class com.adfresca.** {*;} 
-keep class com.google.gson.** {*;} 
-keep class org.openudid.** {*;} 
-keep class sun.misc.Unsafe { *; }
-keepattributes Signature 
```

* * *

## Troubleshooting

if our SDK can't show any message or raise errors, you can debug by following methods below.

**Android**

If you can write your own java codes in UnityPlayerActivity, you can implement setExceptionListener() to print logs, or send the error messages using UnitySendMessage();

```java
AdFresca.setExceptionListener(new AFExceptionListener(){
  @Override
  public void onExceptionCaught(AFException e) {
    Log.w("TAG", e.getLocalizedMessage());
  }
});
```

**iOS**

You can Implement AdFrescaViewDelegate in Xcode project to check the error messages.

```objective-c
// UnityAppController.h
@interface UnityAppController : NSObject<UIApplicationDelegate, AdFrescaViewDelegate>
{
  .....
}
```

```objective-c
// UnityAppController.mm

- (BOOL)application:(UIApplication*)application didFinishLaunchingWithOptions:(NSDictionary*)launchOptions
{
  ...
  [[AdFrescaView shardAdView] setDelegate:self];
}

- (void)fresca:(AdFrescaView *)fresca didFailToReceiveAdWithException:(AdException *)error {  
  NSLog(@"AdException message : %@", [error message]);
}
```

* * *

## Release Notes
- v2.1.6 _(1/10/2014 Updated)_ 
    - Added [Android SDK 2.3.2](https://github.com/adfresca/sdk-android-sample/blob/master/README.eng.md#release-notes)
    - for Unity 4.3.x for Android, 'ForwardNativeEventsToDalvik option has been required to enable touch event. Please refer to [Installation](#installation) section for detailed installation guide.
- v2.1.4 _(12/01/2013 Updated)_ 
    - Added [iOS SDK 1.3.4](https://adfresca.zendesk.com/entries/21796143#release-notes)
- v2.1.4 _(11/27/2013 Updated)_ 
    - Added [iOS SDK 1.3.3](https://adfresca.zendesk.com/entries/21796143#release-notes)
    - Added [Android SDK 2.3.1](https://github.com/adfresca/sdk-android-sample/blob/master/README.eng.md#release-notes)
- v2.1.3 _(10/01/2013 Updated)_ 
    - Added [Android SDK 2.2.3](https://github.com/adfresca/sdk-android-sample/blob/master/README.eng.md#release-notes)
- v2.1.2 _(08/19/2013 Updated)_ 
    - Added [iOS SDK 1.3.2](https://adfresca.zendesk.com/entries/21796143#release-notes)
- v2.1.1
    - Added [Android SDK 2.2.2](https://github.com/adfresca/sdk-android-sample/blob/master/README.eng.md#release-notes)
- v2.1.0 _(08/08/2013 Updated)_
    - Added [Android SDK 2.2.1](https://github.com/adfresca/sdk-android-sample/blob/master/README.eng.md#release-notes)
    - For Android, use PrintTestDeviceIdByLog() method to print test device id in log
- v2.0.1 _(07/26/2013 Updated)_
    - Fix a bug that a default GCMIntentService shows error message when push message was received in 'stopped' application state 
    - Removed some default argument codes of AndroidPlugin.cs 
    - Added Android SDK v2.1.3
- v2.0.0 _(07/10/2013 Updated)_
    - Added some API for _Incentivized CPI_ Campaign
