## Contents
- [Basic Integration](#basic-integration)
    - [Installation](#installation)
    - [Start Session](#start-session)
    - [In-App Messaging](#in-app-messaging)
    - [Push Messaging](#push-messaging)
    - [Test Device Registration](#test-device-registration)
- [IAP, Reward and Sales Promotion](#iap-reward-and-sales-promotion)
  - [In-App Purchase Tracking](#in-app-purchase-tracking)
  - [Give Reward](#give-reward)
  - [Sales Promotion](#sales-promotion)
- [Dynamic Targeting](#dynamic-targeting)
  - [Custom Parameter](#custom-parameter)
  - [Marketing Moment](#marketing-moment)
- [Advanced](#advanced)
  - [Timeout Interval](#timeout-interval) 
- [Reference](#reference)
  - [Deep Link](#deep-link)
  - [Cross Promotion Configuration](#cross-promotion-configuration)
  - [Proguard Configuration](#proguard-configuration)
- [Troubleshooting](#troubleshooting)
- [Release Notes](#release-notes)

* * *

## Basic Integration

### Installation

Download _Unity Plugin_ at the following link.

[Unity Plugin Download](https://s3-ap-northeast-1.amazonaws.com/file.adfresca.com/distribution/sdk-for-Unity.zip) 

Open your Unity Project and run AdFrescaUnityPlugin.package. It will install the following components below.

Assets/

    LitJson.dll

Assets/AdFresca/

    Plugin.cs 
    AndroidPlugin.cs
    IOSPlugin.cs
    RewardItem.cs
    Purchase.cs

Assets/Plugins/Android/

    AdFresca.jar 
    AdFrescaPlugin.jar 
    gcm.jar 
    assets 

Assets/Plugins/iOS/

    AdFrescaPlugin.h
    AdFrescaPlugin.mm

Now, we will start installation for each platform.

#### Android

For Android, you also need to modify AndroidManifest.xml as shown below.

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

For iOS, Unity requires you to do the entire installation just like an iOS native project. After checking all the components from the package file, you need to build and export a Xcode project from Unity.

Install iOS SDK by referring to the following ['iOS SDK Installation'](https://github.com/adfresca/sdk-ios/blob/master/README.md#installation) section of our iOS SDK guide.

### Start Session

Now you need to start putting simple SDK codes in your app. You first need to call startSession() method with your API Key. To get your API Key, go to our [Dashboard](https://dashboard.nudge.do) and then click 'Settings - API Keys' button in your app's 'Overview' page.

#### Android

Put StartSession() method when users start the app. Please make sure that this method is called once, while your app is running.

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

With in-app messaging, you can deliver a message to your users in real-time. Simply put 'Load()' and 'Show()' methods where and when you want to deliver a message. The type of message can be an interstitial image, text, and iframe webpage. The message is only shown when users match the in-app messaging campaign's target logics. We will discuss more details of the dynamic targeting features in the [Dynamic Targeting](#dynamic-targeting) section.

```cs
void Start ()
{
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.Load();
  plugin.Show();
}
```

When you first call in-app messaging methods, you will see the test message as shown below. If you tap on the image, it will redirect users to the product page of the app on the app store. You can hide this test message by changing the test mode configuration later.

<img src="https://adfresca.zendesk.com/attachments/token/ans53bfy6mwq2e9/?name=4444.png" width="240" />
&nbsp;
<img src="https://adfresca.zendesk.com/attachments/token/ec7byt0qtj00qpb/?name=5555.png" height="240" />

### Push Messaging

You can also deliver your push messages anytime you want. Follow the steps below to configure the push notification settings in your app.

#### Android

Before you start, you need to have your GCM project number from Google API Console, and set GCM API Key value to our [Dashboard](https://dashboard.nudge.do). Please refer to '[Android Push Notification Guide](https://adfresca.zendesk.com/entries/82771087)'

1) Add codes to AndroidManifest.xml

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

- If you already have your own GCMReceiver and GCMIntentService classes, you only need to put codes here in your own GCMIntentService class.
- If you haven't implemented any GCM classes yet, you will need to write your own GCM classes in java code. Please use our 'Android Plugin Project' in our unity plugin folder. After importing the sample project in Eclipse ADT, you will need to rename packages in **/src** and **/gen** to your own package name. Also make sure you need to rename 'YOUR.PACKAGE.NAME' in AndroidManifest.xml as shown above.

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
  // Check Nudge notification
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

After updating the two classes as shown above, you need to export them as a single jar file into the unity project (/Assets/Plugins/Android/). Please make sure you only export these two classes, not the entire package.

<img src="https://s3-ap-northeast-1.amazonaws.com/file.adfresca.com/guide/sdk/unity/unity-android-gcm-export.png"/>

5) Unity Code

Now, you add a single line of Unity code to use GCM. Add 'SetGCMSenderId(GCM_PROJECT_ID)' method to set Google API Project number. Please note that the project number is not an API Key value.

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
  - You can export your .cer file to .p12 file using Keychain application. Please refer to [iOS Push Notification Certificate Guide](https://adfresca.zendesk.com/entries/82614238) to generate .p12 and upload to our [Dashboard](https://dashboard.nudge.do)

2. Check your provisioning
  - Nudge only supports APNS production environment. So you should build your app with App Store or Ad Hoc Provisioning file to enable production mode.

3. Add the following codes to AppController.mm in Xcode Project. 

```mm
#import <AdFresca/AdFrescaView.h>

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  ....
  if ([application respondsToSelector:@selector(registerUserNotificationSettings:)]) {
    UIUserNotificationType types = (UIUserNotificationTypeAlert | UIUserNotificationTypeBadge | UIUserNotificationTypeSound);
    UIUserNotificationSettings *notificationSettings = [UIUserNotificationSettings settingsForTypes:types categories:nil];
    [application registerUserNotificationSettings:notificationSettings];
    [application registerForRemoteNotifications];
  } else {
    [application registerForRemoteNotificationTypes:(UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeSound)];
  }

  NSDictionary* userInfo = [launchOptions objectForKey:UIApplicationLaunchOptionsRemoteNotificationKey];
  if (userInfo != nil) {
    [self application:application didReceiveRemoteNotification:userInfo];
  }
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

We are now done with the push messaging implementation!

### Test Device Registration

Nudge supports a test mode feature. With the test mode feature, you can deliver your test message to only registred test devices. 

To register your test device to our dashboard, you need your test device ID from our SDK. Our SDK provides two ways to show a test device ID.
 
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

2) Displaying a test device ID on your app screen using SetPrintTestDeviceId(bool) method.

```cs
AdFresca.Plugin plugin = AdFresca.Plugin.Instance
plugin.Init(API_KEY);
plugin.StartSession();
plugin.SetPrintTestDeviceId(true);
plugin.Load();
plugin.Show();
```

After you have your test device ID, you have to register it to [Dashboard](https://dashboard.nudge.do) under 'Test Device' menu.

* * *

## IAP, Reward and Sales Promotion

### In-App Purchase Tracking

_**(Unity plugin supports in-app-purchase tracking feature only with Android platform. iOS support will be available soon)**_

With in-app-purchase tracking, you can track all the in-app purchases, and use them for targeting specific user segments to display your campaigns.

There are two types of purchases you can track with our SDK.

1. **Hard Currency Item Purchase Tracking:**  purchases made with real money. For example, a user purchased a 'Gold 100' item with 'USD 1.99'.
2. **Soft Currency Item Purchase Tracking:** purchases made with virtual money. For example, a user purchased 'Rocket Launcher' item with 'Gold 10'. 

You don't need to manually write down an item list. Our SDK tracks all purchases and any items included in the purchases are automatically added to our dashboard. To see the list of items, go to 'Overview > Settings > In-App Items' page in our dashboard.

Let's get started by implementing SDK codes with the examples below. 

### Hard Currency Item Tracking

You will use app stores' billing library such as Google Play Billing for purchases of 'Hard Currency Item.' When users purchase an item successfully, simply create Purchase object and use LogPurchase() method. Also, call cancelPromotionPurchase() method when a user cancelled or failed to purchase.

Example 1: Implementing 'purchase succeeded' event in Unity 
```cs
private void OnHardItemPurchased() 
{
    AdFresca.Purchase purchase = new AdFresca.PurchaseBuilder(AdFresca.Purchase.Type.HARD_ITEM)
      .WithItemId("gold100")
      .WithCurrencyCode("USD") // The currencyCode must be specified in the ISO 4217 standard. (ex: USD, KRW, JPY)
      .WithPrice(0.99)
      .WithPurchaseDate(purchaseDateTime) // purchaseDateTime from In-app billing library
      .WithReceipt("google_play_order_id", "google_play_receipt_json", "google_play_signature"); // Optional
          
    AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
    plugin.LogPurchase(purchase);
}

private void OnPurchaseHardItemFailure() 
{
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.CancelPromotionPurchase();
}
```

Example 2: Implementing Google Billing Library in the native Android codes.

```java
// Callback for when a purchase is finished
IabHelper.OnIabPurchaseFinishedListener mPurchaseFinishedListener = new IabHelper.OnIabPurchaseFinishedListener() {
  public void onIabPurchaseFinished(IabResult result, Purchase purchase) {
    Log.d(TAG, "Purchase finished: " + result + ", purchase: " + purchase);

    if (mHelper == null || result.isFailure() || !verifyDeveloperPayload(purchase)) {
      ......
      AdFresca.getInstance(MainActivity.this).cancelPromotionPurchase();
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

          AFPurchase hardPurchase = new AFPurchase.Builder(AFPurchase.Type.HARD_ITEM)
                                .setItemId(itemId)
                                .setCurrencyCode(currencyCode)
                                .setPrice(price)
                                .setPurchaseDate(purhcaseDate)
                                .setReceipt(orderId, receiptData, signature)
                                .build();

          AdFresca.getInstance(UnityPlayer.currentActivity).logPurchase(hardPurchase);
        }
      });
    }
    
    ......
    }
};
```

The above example is written for Google Play. You can also get the required values from other billing libraries such as Amazon.

For more details of Purchase object with hard currency items, check the table below.

Method | Description
------------ | ------------- | ------------
WithItemId(string) | Set the unique identifier of your item. This value may not be different per os platform or app store. We recommend that you make this value unique for all platforms and stores. Our service distinguishes each item by this value.
WithCurrencyCode(string) | Set the current code of ISO 4217 standard. For Google Play, use the currency code of 'default price' in your account. For Amazon, set 'USD' value since Amazon only supports USD.
WithPrice(double) | Set the item price. You may use SkuDetails's value or manually set the value from your server.
WithPurchaseDate(date) | Set the date of purchase. You may use purchase.getPurchaseTime() value. If you set Null value, it will be automatically recorded by our SDK and our server. Please don't use the local time of a user's device.
WithReceipt(string, string, string) | Set the receipt property of purchase object (Google Play only). We will use it to verify the receipt in the future. 

### Soft Currency Item Tracking

When users purchase your soft currency item in the app, you can also create Purchase object and call LogPurchase() method. Also, call CancelPromotionPurchase() method when a user cancelled or failed to purchase.

```cs
private void OnSoftItemPurchased() 
{
    AdFresca.Purchase purchase = new AdFresca.PurchaseBuilder(AdFresca.Purchase.Type.SOFT_ITEM)
      .WithItemId("long_sword")
      .WithCurrencyCode("gold") 
      .WithPrice(100)
      .WithPurchaseDate(purchaseDateTime);
    
    AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
    plugin.LogPurchase(purchase);
}

private void OnPurchaseSoftItemFailure() 
{
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.CancelPromotionPurchase();
}
```

For more details of Purchase object with soft currency items, check the table below.

Method | Description
------------ | ------------- | ------------
WithItemId(string) | Set the unique identifier of your item. This value may not be different per os platform or app store. We recommend that you make this value unique for all platforms and stores. Our service distinguishes each item by this value.
WithCurrencyCode(string) | Set the item's soft currency code. (ex: 'gold', 'gas')
WithPrice(double) | Set the item price. You may get this value from your server. (ex: 100 of gold)
WithPurchaseDate(date) | Set the date of purchase. If you set Null value, it will be automatically recorded by our SDK and our server. Please don't use the local time of a user's device.

### IAP Trouble Shooting

After you call LogPurchase() method, the purchase data is updated to our dashboard in real-time. You can see the list of updated items in 'Overview > Settings > In-App Items' menu.

If you can't see any data in our dashboard, your Purchase object may be invalid. Check your Android (or iOS) logs if our plugin printed error messages. The log format will be "AFPurchaseExceptionListener.onException = {error message}".

* * *

### Give Reward

When you use 'Reward Item' in in-app messaging or 'Incentive item' in incentivized CPI & CPA campaign, you should implement this 'reward item' code to give a reward item to users.

With implementing reward item codes, you can check if users have any reward to receive, and then will be notified with a reward item information.

Use the two codes as shown below:
- CheckRewardItems(): This method checks if any item is available to receive. We recommend to put this code when the app becomes active. 
- SetAndroidRewardItemListener(): When the reward condition is met with the current user, onReward event is automatically called with an item value from our Android SDK. For iOS, you need to add codes in Xcode, as shown below. Then you can give an item to the user with RewardItem object.

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

  // give an item to the user using 'uniqueValue' of the item
  SendItemToUser(rewardItem.uniqueValue);
}
```

You need to implement your own 'SendItemToUser()' method. This method may send the current user info and item's uniqueValue to your server, which willl give the item to the user.

The onReward event is called when a campaign's reward condition is met.

- Reward Campaign: event is called when a user sees the campaign content.
- Incentivized CPI Campaign: event is called when our SDK confirms that the advertising app is installed.
- Incentivized CPA Campaign: event is called after our SDK confirms that the advertising app is installed and the user completes the intended action (calling the specified marketing moment.)

If users have any network connectivity issues, our SDK stores the reward data in the app's local storage, and then deliver the reward when users run the app again. This guarantees that users will always get the reward from our SDK.

#### implementing SendItemToUser()

You should give a reward item to your user using your own client code or back-end server api. Your client may send an api request with an unique vale of item, quantity and security token values to your server. Then the server application will add a reward item to the user's item inventory.

#### Hack Proof

Our SDK never calls itemRewarded event more than once per campaign. We always check it with device identifiers to avoid abuse. However, it is still possible for hackers to hijack your api request between your client and back-end server. To prevent this issue, we provide a security token value. A security token is an unique value per your campaign. You can generate the token while you're creating a reward campaign. You can hack proof by using the security token as noted below:

1. You will store a security token to your own database before starting a reward campaign. You should reject any reward requests with an invalid token value.
2. If some users are trying to request with the same token value more than once, you should reject those requests.
3. If you think your security token is exposed to hackers, you can always change the value in our dashboard.

* * *

### Sales Promotion

_**(Unity plugin supports promotion feature only with Android platform. iOS support will be available soon)**_

By using sales promotion campaigns, you can promote your in-app item to your users. When users tap on an action button of an image message, a purchase UI will appear to proceed with the user's purchase. Our SDK will automatically detect if users made a purchase or not, and then will update the campaign performance to our dashboard in real time.

To apply our promotion features, you should implement AFPromotionDelegate. onPromotion event is automatically called when users tap on an action button of an image message in a sales promotion campaign. You just need to show the purchase UI of the promotion item using 'promotionPurchase' object. 

For Hard Currency Items, you should use your in-app billing library codes to show the purchase UI. You can get the SKU value from ItemId property of promotionPurchase object.

For Soft Currency Items, you should use your own purchase UI which might be already implemented in your store page. Also there are discount options for soft currency item sales promotion campaigns. You can check the discount type using discountType property of promotionPurchase object

1. **Discount Price**: Users can buy a promotion item at a specific discounted price. You can get the price from Price property.

2. **Discount Rate**: Users can buy a promotion item at a discount rate. You will calculate the discounted price applying the discount rate which can be earned from DiscountRate property.

```cs
void Start ()
{
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.Init(API_KEY);
  plugin.StartSession();
  
  ....
  
  plugin.SetPromotionListener("YourGameObject", "OnPromotion");
}
  
public void OnPromotion(string json)
{
  Debug.Log("OnPromotion = " + json);   
  Purchase PromotionPurchase = LitJson.JsonMapper.ToObject<Purchase>(json);

  string ItemId = PromotionPurchase.ItemId;
  string LogMessage = "no logMessage";

  if (PromotionPurchase.PurchaseType == Purchase.Type.HARD_ITEM)
  {
    // Use In-app Billing Library  
    ShowHardItemPurchaseUI(ItemId);
    LogMessage = String.Format("on HARD_ITEM Promotion ({0})", ItemId);  
  }
  else if (PromotionPurchase.PurchaseType == Purchase.Type.SOFT_ITEM)
  {
    String CurrencyCode = PromotionPurchase.CurrencyCode;
    if (PromotionPurchase.PromotionDiscountType == Purchase.DiscountType.DISCOUNTED_TYPE_PRICE) 
    {
      // Use a discounted price
      double DiscountedPrice = PromotionPurchase.Price;
      ShowSoftItemPurchaseUIWithDiscountedPrice(ItemId, CurrencyCode, DiscountedPrice);
      LogMessage = String.Format("on SOFT_ITEM Promotion ({0}) with {1} {2}", ItemId, DiscountedPrice, CurrencyCode);
    }
    else if (PromotionPurchase.PromotionDiscountType == Purchase.DiscountType.DISCOUNT_TYPE_RATE)
    {
      // Use this rate to calculate a discounted price of item. discountedPrice = originalPrice - (originalPrice * discountRate)
      double DiscountRate = PromotionPurchase.PromotionDiscountRate;
      ShowSoftItemPurchaseUIWithDiscountRate(ItemId, CurrencyCode, DiscountRate);
      LogMessage = String.Format("on SOFT_ITEM Promotion ({0}) with {1}% discount", ItemId, DiscountRate * 100.0);
    }
  }

  Debug.Log(LogMessage);
}
```

Our SDK will detect if users made a purchase using our [In-App Purchase Tracking](#in-app-purchase-tracking) features. Therefore, you should implement it to complete this promotion feature. Please make sure that you implement 'CancelPromotionPurchase()' method when the users cancelled or failed to purchase items.

* * *

## Dynamic Targeting

### Custom Parameter

Our SDK can collect user-specific profiles such as level, stage, maximum score, etc. We can use them to deliver a personalized and targeted message in real-time to specific user segments that you define.

To implement codes, simply call SetCustomParameter method with parameter's index and value. You can get the custom parameter's index in our [Dashboard](https://dashboard.nudge.do): 1) Select an App 2) In 'Overview' menu, click 'Settings - Custom Parameters.'

You will call the method after the app is launched and when the value has changed. 

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

Marketing Moment means when and where you want to engage users. For example, you may need to deliver a message when a user completes a quest or enters into an item store. You will be able to use it with the [custom parameters](#custom-parameter) so you can deliver a personalized and targeted message at a specific moment in real-time.

To implement codes, simply call **Load(index)** method with marketing moment's index. You can get the marketing moment's index in our [Dashboard](https://dashboard.nudge.do): 1) Select an App 2) In 'Overview' menu, click 'Settings - Marketing Moment.' 

You will call the method after the moment happened in the app.

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

You can set a timeout interval for a messaging request. If the message is not loaded within this time interval, the message won't be displayed to users and our SDK will return control to your app.

Default is 5 seconds and you can set it from 1 to 5 seconds.

```cs
AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
plugin.SetTimeoutInterval(5);
plugin.Load();
plugin.Show();
```

* * *

## Reference

### Deep Link

You can set your own URL Schema as 'Deep Link' of the campaigns and you can redirect users to a specific page or run custom actions when usesr click the image message. 

#### Use a Custom URL for Android

In the native Android application, you can simply add schema information to AndroidManifest.xml to use custom url. However, unlike the native Android application that uses multiple activities for its pages, Unity engine uses only one player activity and implements its own paginations internally. So you can't add schema information and set schema on UnityPlayer activity.

To resolve this issue, you need to do the followings.

1) Override startActivity(intent) of UnityPlayer Activity to handle Custom URL for In-App Messaging Campaign.

Deep Link from In-App Messaging Campaign is always executed on in-game situation. It is never executed from outside of the game (i.e. a push notification.) Also our SDK uses startActivity() method to execute the url. Therefore, you can manually handle a url by overriding startActivity() of UnityPlayer activity. 

First, you should create a new Android Project in Eclipse and generate a class named MainActivity inherited from UnityPlayerActivity. Then modify AndroidMenefest.xml as follows.

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

You need to implement startActivity() method of MainActivity. If a custom url with 'myapp://" schema is received, it will pass a uri string value to your Unity game object.

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

2) Handle custom url form Push Messaging Campaign

When the app receives a push notification with a custom url, you can execute your own custom action. Typically a notification is received when a user is outside of the game. So we should handle the custom url with a different approach. 

First, create a new activity class named 'PushProxyActivity,' and register the activity in AndroidMenefest.xml as shown below.

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
You also need to create a custom url like 'myapp://com.adfresca.push?item=abc' in your Push Messaging Campaign. 

Then you should implement PushProxyActivity class. This class is a simple proxy-style activity which only handles a url from Android OS and then quits itself. 
However, there is an exception that a notification is received and your application is not running. When that happens, you can't handle a custom url in the game engine, so you should manually start your game and pass the url to MainActivity as shown below.

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
Finally, you should handle a url from PushProxyActivity in your MainActivity.

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

#### Use a Custom Url in iOS

For iOS, it is much easier to use a custom url since there is only one event method for url handling.

1) Set your custom url schemes in Info.plst as follows.

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

#### Use Custom Url in Unity

Both example codes shown above have passed a url string to 'OnCustomURL' method of 'Fresca' game object. Now you can handle url values in Unity.

```java
public void OnCustomURL(string url)
{
  Debug.Log("OnCustomURL = " + url); 
  //ex) if myapp://com.adfresca.custom?item=abc is passed, parse 'item=abc' string to give item to users
}
```

* * *

### Cross Promotion Configuration

Using Incentivized CPI & CPA Campaigns, users in 'Media App' can get an incentive item when they install 'Adverting App' from the campaigns.

- Media App: app which displays a promotion image to users and redirects users to Advertising App's install page in the app store and gives an incentive item to users when they complete the pre-defined actions in the Advetising App. 
- Advertising App: app which is the target of advertisment.

To integrate our SDK with this feature, you should set URL Schema value for the adverting app and implement codes to give an incentive item to users in the media app.

#### Advertising App Configuration:

  1. Android

  For Android, our SDK uses the package name to check if the advertising app is installed in the same device. 

  You can find the package name in AndroidManifest.xml

  ```xml
  <manifest package="com.adfresca.demo">
    ...
  </manifest>
  ```

  In this case, you should set CPI Identifier value of the advertising app to "com.adfresca.demo" in our dashboard.

  2. iOS

  For iOS, our SDK uses URL Schema value to check if the advertising app is installed.

  Open Info.plst in Xcode to check your value.

  <img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png"/>

  You need to set the CPI Identifier value of the advertising app to "myapp://" in our dashboard. For iOS, a url schema value may be redundant with other apps', so be careful to choose a unique value to run a CPI Campaign.

  For an Incentivized CPI Campaign, the advertising app does not need to have our SDK integrated. You only need to set the package name correct. 

  However, if you use an Incentivized CPA Campaign, you need to integrate our SDK in the advertising app and also implement a 'Marketing Moment' to use for a reward condition. For example, when you set a reward condition as'Tutorial Complete' marketing event, you should call the marketing moment method to inform that the user hit the condition (goal).

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

If you use Proguard to protect your APK, you need to add exception configurations for our Android SDK. Add the following lines of codes to ignore our SDK, OpenUDID, and Google Gson. 

```java
-keep class com.adfresca.** {*;} 
-keep class com.google.gson.** {*;} 
-keep class org.openudid.** {*;} 
-keep class sun.misc.Unsafe { *; }
-keepattributes Signature 
```

* * *

## Troubleshooting

If our SDK does not show any message or raise errors, you can debug by the following methods.

**Android**

If you can write your own java codes in UnityPlayerActivity, you can implement setExceptionListener() to print logs, or send error messages using UnitySendMessage();

```java
AdFresca.setExceptionListener(new AFExceptionListener(){
  @Override
  public void onExceptionCaught(AFException e) {
    Log.w("TAG", e.getLocalizedMessage());
  }
});
```

**iOS**

You can implement AdFrescaViewDelegate in a Xcode project to check error messages.

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
- **v2.2.3 _(2014/12/05 Updated)_**
  - HARD_ITEM and SOFT_ITEM enums are added to Purchase class to replace ACTUAL_ITEM and VIRTUAL_ITEM which will be deprecated. Please refer to [In-App Purchase Tracking](#in-app-purchase-tracking) section.
- **v2.2.2 (2014/09/30 Updated)**
    - Support A/B test feature for iOS platform. (No coding is required)
    - Added [iOS SDK 1.4.6](https://github.com/nudge-now/sdk-ios/blob/master/README.md#release-notes)
- v2.2.1 _(8/15/2014 Updated)_
    - Support sales promotion campaign. Please refer to [Sales Promotion](#sales-promotion) section.
    - Support security token of reward campaign's hack proof. Please refer to [Give Reward](#give-  reward) section.
    - Add cancelPromotionPurchase() method to [In-App Purchase Tracking](#in-app-purchase-tracking)
    - Support tap area feature.
    - SDK will match multiple campaigns and show multiple messages in one marketing moment request.
    - Added [Android SDK 2.4.2](https://github.com/adfresca/sdk-android-sample/blob/master/README.eng.md#release-notes)
- v2.2.0 _(1/14/2014 Updated)_ 
    - [In-App Purchase Tracking](#in-app-purchase-tracking) is added.
- v2.1.8 _(4/6/2014 Updated)_
    - SDK supports '[Reward Item](#reward-item)' feature of the In-App Messaging campaign.
    - SDK supports 'Incentivized CPA Campaign'. Please refer to 'CPI Identifier' section for detail. 
    - Added [iOS SDK 1.3.5](https://github.com/nudge-now/sdk-ios/blob/master/README.md#release-notes)
    - Added [Android SDK 2.3.4](https://github.com/adfresca/sdk-android-sample/blob/master/README.eng.md#release-notes)
- v2.1.7 _(1/31/2014 Updated)_
    - [Android SDK 2.3.3](https://github.com/adfresca/sdk-android-sample/blob/master/README.md#release-notes) 버전을 지원합니다.
- v2.1.6 _(1/10/2014 Updated)_ 
    - Added [Android SDK 2.3.2](https://github.com/adfresca/sdk-android-sample/blob/master/README.eng.md#release-notes)
    - for Unity 4.3.x for Android, 'ForwardNativeEventsToDalvik option is required to enable a touch event. Please refer to [Installation](#installation) section for detailed installation guide.
- v2.1.4 _(12/01/2013 Updated)_ 
    - Added [iOS SDK 1.3.4](https://github.com/nudge-now/sdk-ios/blob/master/README.md#release-notes)
- v2.1.4 _(11/27/2013 Updated)_ 
    - Added [iOS SDK 1.3.3](https://github.com/nudge-now/sdk-ios/blob/master/README.md#release-notes)
    - Added [Android SDK 2.3.1](https://github.com/adfresca/sdk-android-sample/blob/master/README.eng.md#release-notes)
- v2.1.3 _(10/01/2013 Updated)_ 
    - Added [Android SDK 2.2.3](https://github.com/adfresca/sdk-android-sample/blob/master/README.eng.md#release-notes)
- v2.1.2 _(08/19/2013 Updated)_ 
    - Added [iOS SDK 1.3.2](https://github.com/nudge-now/sdk-ios/blob/master/README.md#release-notes)
- v2.1.1
    - Added [Android SDK 2.2.2](https://github.com/adfresca/sdk-android-sample/blob/master/README.eng.md#release-notes)
- v2.1.0 _(08/08/2013 Updated)_
    - Added [Android SDK 2.2.1](https://github.com/adfresca/sdk-android-sample/blob/master/README.eng.md#release-notes)
    - For Android, use PrintTestDeviceIdByLog() method to print a test device id in log
- v2.0.1 _(07/26/2013 Updated)_
    - Fix a bug that a default GCMIntentService shows an error message when a push message is received in 'stopped' application state 
    - Removed some default argument codes of AndroidPlugin.cs 
    - Added Android SDK v2.1.3
- v2.0.0 _(07/10/2013 Updated)_
    - Added some API for _Incentivized CPI_ Campaign
