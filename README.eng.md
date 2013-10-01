## Contents
- [Introduction](#introduction)
- [Quick Start](#quick-start)
    - [Installation](#installation)
    - [Code](#code)
- [Basic Features](#basic-features)
    - [In-App-Purchase Count](#in-app-purchase-count)
    - [Custom Parameter](#custom-parameter)
    - [Event](#event)
- [Push Notification](#push-notification)
- [Custom URL](#custom-url)
- [Reward Item](#reward-item)
- [Advanced Features](#advanced-features)
    - [Test Device ID](#test-device-id)
    - [Timeout Interval](#timeout-interval)
- [Release Notes](#release-notes)

* * *

## Introduction

To use _AD fresca SDK_ in Unity Engine, we provide a simple unity plugin.

You can easily install the components of plugin using Unity package file, and use SDK with following simple codes.

All the codes of plugin are open so you can always check and modify as you want


* * *

## Quick Start

### Installation

Download _Unity Plugin_ at the following link.

[AD fresca Unity Plugin v2.1.3  Download](https://s3-ap-northeast-1.amazonaws.com/file.adfresca.com/distribution/sdk-for-Unity.zip) (Android SDK v2.2.3, iOS SDK v1.3.2)

Open your Unity Project and run AdFrescaUnityPlugin.package. It will install following components below

Assets/

    LitJson.dll

Assets/AdFresca/

    Plugin.cs 
    AndroidPlugin.cs
    IOSPlugin.cs
    RewardItem.cs

Assets/Plugins/Android/

    AdFresca.jar 
    AdFrescaPlugin.jar 
    gcm.jar 
    assets 
    AndroidManifest.xml 

Assets/Plugins/iOS/

    AdFrescaPlugin.h
    AdFrescaPlugin.mm

Now, we start some installation works for each platform

### Android

For Android, we have already done with most of installation from package file. You just need to modify AndroidManifest.xml as below

#### Modify AndroidManifest.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android" android:installLocation="preferExternal" package="com.test.android" android:versionName="1.0" android:versionCode="1">	
	<application android:icon="@drawable/app_icon" android:label="@string/app_name" android:debuggable="true">
		<activity android:name="com.unity3d.player.UnityPlayerActivity" android:label="@string/app_name">
		  <intent-filter>
		    <action android:name="android.intent.action.MAIN" />
		    <category android:name="android.intent.category.LAUNCHER" />
		  </intent-filter>
		</activity>
		
    <!-- Service for OpenUDID -->
    <service android:name="org.openudid.OpenUDID_service">
      <intent-filter>
        <action android:name="org.openudid.GETUDID" />
      </intent-filter>
    </service>

    <!-- Activity for Incentivized Campaign -->
    <activity android:name="com.adfresca.sdk.reward.AFRewardActivity" />
      
   <!-- Boradcast Receiver for Google Referrer Tracking-->
    <receiver android:name="com.adfresca.sdk.referer.AFRefererReciever" android:exported="true">
    	<intent-filter>
          <action android:name="com.android.vending.INSTALL_REFERRER" />
   	</intent-filter>	
    </receiver>
            
   <!-- Add following codes if you need a push notification feature -->
    <activity android:name="com.adfresca.ads.AdFrescaPushActivity" />
    <receiver android:name="com.google.android.gcm.GCMBroadcastReceiver" android:permission="com.google.android.c2dm.permission.SEND">  
      <intent-filter>
        <action android:name="com.google.android.c2dm.intent.RECEIVE" />
        <action android:name="com.google.android.c2dm.intent.REGISTRATION" />
        <category android:name="your_app_package" />
      </intent-filter>
    </receiver>
    <service android:name=".GCMIntentService" />  <!-- To handle GCM messages, you need to implement GCMIntentService class (See 'Push Notification' for detail  -->
  </application>
  <uses-feature android:glEsVersion="0x00020000" />
	
  <uses-permission android:name="android.permission.INTERNET"/>
  <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>

  <!-- Add following codes if you need a push notification feature -->
  <permission android:name="your_app_pakcage.permission.C2D_MESSAGE" android:protectionLevel="signature" />
  <uses-permission android:name="your_app_package.permission.C2D_MESSAGE" />
  <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
  <uses-permission android:name="android.permission.GET_ACCOUNTS" />
  <uses-permission android:name="android.permission.WAKE_LOCK" />
</manifest>
```

Make sure you have changed all 'com.Company.ProductName' package names to your owns. 
For GCMReceiver and GCMIntentService class, check [Push Notification](#push-notification) section later.

### iOS

For iOS, Unity requires to do the same installation work as a native project. After checking all the components from package file, build and export Xcode project form Unity.

Start to Install iOS SDK with following ['1. SDK Installation'](https://adfresca.zendesk.com/entries/21796143#installation) section of our iOS SDK guide.

Then add following codes in AppController.mm

```mm
#import <AdFresca/AdFrescaView.h>

  - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [AdFrescaView startSession:@"YOUR_API_KEY"];
    [[UIApplication sharedApplication] registerForRemoteNotificationTypes:UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge |UIRemoteNotificationTypeSound];   // for Push Notification 
  } 

  // for Push Notification, add following lines

  - (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
    [AdFrescaView registerDeviceToken:deviceToken];
  }
  
  - (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
    if ([AdFrescaView isFrescaNotification:userInfo] && [application applicationState] != UIApplicationStateActive) {
      [AdFrescaView handlePushNotification:userInfo];
    }
  } 
```

Now, we are done with Plugin installation! 

### Code

You need only several lines of code to show contents of campaigns like the following.

```cs
public class AdFrescaUnitySample : MonoBehaviour
{
	#if UNITY_ANDROID
	private static string API_KEY = "YOUR_ANDROID_API_KEY";
	private static string GCM_SENDER_ID = "YOUR_GCM_SENDER_ID"; // Google API Proejct ID (ex: 12345678)
	#elif UNITY_IPHONE
	private static string API_KEY = "YOUR_IOS_API_KEY";
	private static string GCM_SENDER_ID = null;
	#else
	private static string API_KEY = null;
	private static string GCM_SENDER_ID = null;
	#endif

	
	void Start ()
	{
		AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
		plugin.Init(API_KEY);
		plugin.StartSession();
	}

	void OnGUI ()
	{
		if (GUI.Button (new Rect (100, 100, 150, 150), "Load & Show")) {
			AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
			plugin.Load();
			plugin.Show();
		}
	}
}
```

`plugin.Init(API_KEY);` Set API Key

`plugin.StartSession();` Start requesting the session logging. Please make sure that it is called once while your app is running.

`plugin.Load();` Load a matched content.

`plugin.Show();` Show the loaded content.

You can see the following if everything is correct.

<img src="https://adfresca.zendesk.com/attachments/token/2fv9e76ptp7yo3h/?name=android-sample-p.png" width="240" />
&nbsp;
<img src="https://adfresca.zendesk.com/attachments/token/phn4fcpvbi2damx/?name=device-2013-03-18-133443.png" height="240" />
* * *

## Basic Features

### In-App Purchase Count

If user purchases in-app items, you can save this information on our service to use use targeting and analytics features.

You can simply set a number of in-app purchases for current user, using SetNumberOfInAppPurchases(int) method.

```cs
void Start() {
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.Init(API_KEY);
  plugin.SetNumberOfInAppPurchases(user.GetInAppPurchaseCount());
  plugin.StartSession();
}

.....

void OnUserPurchasedItem() {
  User.inAppPurchaseCount++;

  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.SetNumberOfInAppPurchases(User.GetInAppPurchaseCount());
  plugin.Load();
  plugin.Show();
}
```

**Caution:** SetNumberOfInAppPurchases() method should be called earlier than StartSession() and Load() as an example above. If this method is not initially called earlier than StartSession(), the value of in-app purchase count is not updated to our service on user's first session, but the cached value is updated since user's second session (SDK caches the value on the local storage since Android v2.2.0 and iOS v1.3.1)

### Custom Parameter

AD fresca can save user specific information such as level, stage, gender and etc to use targeting and analytics features.

In SDK, you can just set custom parameters using SetCustomParameter method with passing parameter's index and value.

(You can set and see the custom parameter's index in our Admin website: 1) Select App 2) In 'Overview' menu, click 'Details' button for each app store)

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

void onUserLevelChanged(int level) {
  User.level = level
  
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.SetCustomParameter(CUSTOM_PARAM_INDEX_LEVEL, User.level);
  plugin.Load(EVENT_INDEX_LEVEL_UP);
  plugin.Show();
}
```

**Caution:** SetCustomParameter() method should be called earlier than StartSession() and Load(). Especially, you should Initialize custom parameter values before calling StartSession(), and then set the changed values at later events as an example above. However, some apps may not able to Initialize values earlier than StartSession(). In this case, the values of custom parameters are not updated to our service on user's first session, but the cached values are updated since user's second session  (SDK caches the value on the local storage since Android v2.2.0 and iOS v1.3.1)

### Event

Using 'Event' feature, you can define your in-app events such as user's behavior, page navigation, and etc. Then you can target your campaigns for each event. 

For defining events on Dashboard, see ['Event Guide (Korean)'](https://adfresca.zendesk.com/entries/23359141)  

After you finished defining events, you need 'Event Index' value for each event. The index value is an integer value like '1,2,3,4'. We recommend to manage these values with Constant or Enum types in your source code.

To simply apply codes,  just pass event index into `plugin.Load(int eventIndex)` method when the event occurs.

(If you don't specify event index on `plugin.Load(int eventIndex)`, a default index is set to 1)

**Example**:   When user entered a main page

```cs
AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
plugin.Load(EVENT_INDEX_MAIN_PAGE);  // Request contents for main page event
plugin.Show();
```

**Example**: When user's character level increased

```cs
AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
plugin.SetCustomParameter(CUSTOM_PARAM_INDEX_LEVEL, level); // Update the latest user level
plugin.Load(EVENT_INDEX_LEVEL_UP); // Request contents for level up event
plugin.Show();
```
* * *

## Push Notification

You can send a push notification and see the result of how users respond a notification by AD fresca

#### Android

Before you start, we recommend reading ["GCM: Getting Started" ](http://developer.android.com/google/gcm/gs.html) from Google.

1) Check AndroidManifest.xml

```xml
<manifest>   
  <application>
      .........
      <activity android:name="com.adfresca.ads.AdFrescaPushActivity" />
      <receiver android:name="com.Company.ProductName.CustomGCMReceiver"
        android:permission="com.google.android.c2dm.permission.SEND">  
        <intent-filter>
          <action android:name="com.google.android.c2dm.intent.RECEIVE" />
          <action android:name="com.google.android.c2dm.intent.REGISTRATION" />
          <category android:name="com.Company.ProductName" />
         </intent-filter>
      </receiver>
      <service android:name="com.Company.ProductName.GCMIntentService" />  
   </application>
    ..........
    <permission android:name="com.Company.ProductName.permission.C2D_MESSAGE" android:protectionLevel="signature" />
    <uses-permission android:name="com.Company.ProductName.permission.C2D_MESSAGE" />
    <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
    <uses-permission android:name="android.permission.GET_ACCOUNTS" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    ..........
</manifest>
```

2) Implement GCMIntentService class

```java
  public class GCMIntentService extends GCMBaseIntentService {

    public GCMIntentService() {
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
            String appName = AdFrescaPlugin.getAppName(context);
            int icon = R.drawable.app_icon;
            long when = System.currentTimeMillis();
            Class<?> targetActivityClass = null;
            
            if (UnityPlayer.currentActivity != null) {
            	targetActivityClass = UnityPlayer.currentActivity.getClass();
            } else {
            	targetActivityClass = YourMainActivity.class; // or UnityPlayer.class
            }
            
            AdFresca.showNotification(context, intent, targetActivityClass, appName, icon, when);
        }                
    }
    
    @Override
    protected void onError(Context context, String registrationId) {
    
    }
  }
```

showNotification() method will show a simple notification view with message, but it has no sound. If you want to do some customization of view such as adding sound or using big view style ui, please check ["Custom Notification"](https://github.com/adfresca/sdk-android-sample/blob/master/README.eng.md#custom-notification) section of Android SDK guide.

3) Implement GCMReceiver class

```java
public class GCMReceiver extends GCMBroadcastReceiver { 
   	@Override
	protected String getGCMIntentServiceClassName(Context context) { 
		return "com.Company.ProductName.GCMIntentService"; 
	} 
}
```

4) Unity Code

Now, you add a single line of unity code to use GCM. Add 'SetGCMSenderId(GCM_SENDER_ID)' method to set Google API Project number. Be carful that project number is not API Key value.

```cs
#if UNITY_ANDROID
private static string GCM_SENDER_ID = "12345678"; // Google API Project Number (ex: 12345678)
#endif

void Start ()
{
	AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
	plugin.Init(API_KEY);
	
	plugin.SetGCMSenderId(GCM_SENDER_ID);
	plugin.StartSession();
}
```

#### iOS

Before you start, we recommend reading  ["Local and Push Notification Programming Guide"](https://developer.apple.com/library/mac/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Introduction.html) from Apple.
(Currently, AD fresca only provides production environment of APNS. We wil support development environment in the near future)  

1) Creating Apple Push Notification Certificate (.p12) file and uploading on AD fresca Admin
- Follow  ["iOS Push Notification Certificate Guide - KOR"](https://adfresca.zendesk.com/entries/21714780) to create push notification certificate.
- After you export .p12 file from Keychain, you need to upload this file to our Admin site (Select App -> Overview -> iOS App Store Edit -> Push Authentication) 

2) Check Info.plast / Check Provisioning
- Check if a value of  'aps-environment' is set to 'production' in Info.plst
- You should build your app with App Store or Ad Hoc Provisioning file to enable production mode

3) Add event codes to AppController 
- In your AppController.mm, check following methods and codes

```mm
#import <AdFresca/AdFrescaView.h>

  - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [AdFrescaView startSession:@"YOUR_API_KEY"];
    [[UIApplication sharedApplication] registerForRemoteNotificationTypes:UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge |UIRemoteNotificationTypeSound];  // for Push Notification
  } 

  - (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
    // Register user's push device token to our SDK
    [AdFrescaView registerDeviceToken:deviceToken];
  }
  
  - (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
    // Check a push notification is form AD fresca. Also, Ignore a notification received when app is already running  
    if ([AdFrescaView isFrescaNotification:userInfo] && [application applicationState] != UIApplicationStateActive) {
      [AdFrescaView handlePushNotification:userInfo];
    }
  } 
```

For iOS, you don't need to add any codes of Unity
So, we are now done with Push Notification implementation!

* * *

## Custom URL

You can set your own URL Schema for click url of Announcement Campaign and url schema of Push Notification Campaign. 

So, you can navigate your users to the specific page or do some custom actions when user click the content view.

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

### Use custom url in Unity

Both example codes above have passed url strings to 'OnCustomURL' method of 'Fresca' game object. Now you can handle url values in Unity.

```java
public void OnCustomURL(string url)
{
	Debug.Log("OnCustomURL = " + url); 
	//ex) if myapp://com.adfresca.custom?item=abc is passed, parse 'item=abc' string to give item to users
}
```

* * *

## Reward Item

_Incentivzed Campaign_ makes it possible to give a reward to users who see an ad of _Advertising App_ in _Media App_ and install _AdVertising App_ by the ad.

(It is highly recommended to install SDK in _Advertising App_ for _CPA Campaign_ that is coming soon although it is not required at this time.)

**Install SDK for Media App**

- Check /Plugins/Android/AndroidManifest.xml

```xml
<manifest>   
  <application>
      .........
      <activity android:name="com.adfresca.sdk.reward.AFRewardActivity" />
      .........
   </application>
</manifest>
```

**Set Advertising App (only for iOS)**

Set your own url schema to check app install in Info.plst as follows

<img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png"/>

In this case, you should set CPI Identifier value of advertising app to "myapp://" in our dashboard.

For iOS, url schema value may be duplicated with other apps, so be careful to choose unique value to run CPI Campaign..

**Code**

- Call GetAvailableRewardItems() method when you want to get an item list.

- Array returned from GetAvailableRewardItems() contains objects of 'AdFresca.RewardItem' which has name, quantity and uniqueValue properties. You should use uniqueValue property of RewardItem object to give an item to your users. (ex: you may send an item request to your game server with user id and uniqueValue)

- 'GetAvailableRewardItems()' method only return when all the validation work are done for current user. it may return items at later if reward is still in validation process.

```cs
AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
IList<AdFresca.RewardItem> rewardItemList = plugin.GetAvailableRewardItems();
Debug.Log ("GetAvailableRewardItems = " + rewardItemList.Count);
foreach(AdFresca.RewardItem rewardItem in rewardItemList)
{
    // do something with rewardItem.name, rewardItem.quantity, rewardItem.uniqueValue
    Debug.Log ("name: " + rewardItem.name + ", quantity: " + rewardItem.quantity + ", uniqueValue: " + rewardItem.uniqueValue);
}
```

###(Advanced) Giving a reward with a faster way:

- Call `CheckRewardItems()` to check at the time of starting app or whenever you want.
- Call `GetAvailableRewardItems()` to retrieve available reward items to give it to users.

```cs
void Start ()
{
    AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
    plugin.Init(API_KEY);
    plugin.StartSession();
    plugin.CheckRewardItems();
}

void OnGUI ()
{
    if (GUI.Button (new Rect (100, 100, 150, 150), "Reward")) {
        AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
        IList<AdFresca.RewardItem> rewardItemList = plugin.GetAvailableRewardItems();
        Debug.Log ("GetAvailableRewardItems = " + rewardItemList.Count);
        foreach(AdFresca.RewardItem rewardItem in rewardItemList)
        {
            // do something with rewardItem.name, rewardItem.quantity, rewardItem.uniqueValue
            Debug.Log ("name: " + rewardItem.name + ", quantity: " + rewardItem.quantity + ", uniqueValue: " + rewardItem.uniqueValue);
        }
    }
}
```

**Tip:** `If you use CheckRewardItems((bool synchronized) with synchronized=true option, _Plugin_ will wait until checking is done, and then you can give an item immediately

* * *

## Advanced Features

### Test Device ID

AD fresca supports a test mode feature. you can easily register test devices and manage them.

To check test device id, we provide two methods on our SDK.

1. Using getTestDeviceId() Method

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
2. Displaying test device id on your app screen using setPrintTestDeviceId method
 
```cs
AdFresca.Plugin plugin = AdFresca.Plugin.Instance
plugin.Init(API_KEY);
plugin.StartSession();
plugin.SetPrintTestDeviceId(true);
plugin.Load();
plugin.Show();
```

### Timeout Interval

You can set a timeout interval for content request. If content is not loaded within this time interval, Content won't be displayed to users and SDK will return the control to your app.

Default is 5 seconds and you can set from 1 seconds to 5 seconds.

```cs
AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
plugin.Init(API_KEY);
plugin.SetTimeoutInterval(5);
plugin.Load();
plugin.Show();
```

* * *

## Release Notes
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
