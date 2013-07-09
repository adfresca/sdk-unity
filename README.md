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
    - [Custom Notification](#custom-notification)
- [Reward Item](#reward-item)
- [Advanced Features](#advanced-features)
    - [Test Device ID](#test-device-id)
    - [Timeout Interval](#timeout-interval)
- [Release Notes](#release-notes)

* * *

## Introduction

Unity 엔진에서 _AD fresca SDK_를 사용할 수 있도록 공식 플러그인을 제공합니다.  

Unity Package 파일을 통해 모든 구성요소를 쉽게 설치할 수 있으며, 아래 적용 가이드에 따라 간단한 코드만으로  Unity 환경에서도 SDK 적용이 가능하게 되었습니다.

플러그인 제작에 사용된 모든 Wrapper 코드가 공개되며, 필요한 경우 각 게임에 알맞게 수정하여 적용할 수 있습니다. 

* * *

## Quick Start

### Installation

아래 링크를 통해 _Unity Plugin_을 다운로드 합니다.

[AD fresca Unity Plugin v2.0.0  다운로드](https://s3-ap-northeast-1.amazonaws.com/file.adfresca.com/distribution/sdk-for-Unity.zip) (Android SDK v2.1.2, iOS SDK v1.2.1)

Unity 프로젝트를 열고 AdFrescaUnityPlugin.package 파일을 실행합니다.

아래의 구성 요소들이 Assets 폴더 아래에 복사되어야 합니다.

 

/AdFresca

    Plugin.cs 
    AndroidPlugin.cs
    IOSPlugin.cs
    Reward.cs

/Plugins/Android

    AdFresca.jar 
    AdFrescaPlugin.jar 
    gcm.jar 
    assets 
    AndroidManifest.xml 

/Plugins/iOS

    AdFrescaPlugin.h
    AdFrescaPlugin.mm

 

이제 각 플랫폼에 맞게 설치 작업을 진행합니다.

### Android

Android 플랫폼의 대부분의 설치 및 적용 작업이 플러그인에 이미 구현되어 있습니다.기본적으로 AndroidManifest.xml 수정 작업만 진행하여 주시면 됩니다.

#### AndroidManifest.xml 수정하기

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
		
		<service android:name="org.openudid.OpenUDID_service">
	        <intent-filter>
	          <action android:name="org.openudid.GETUDID" />
	        </intent-filter>
		</service>
		
		<activity android:name="com.adfresca.sdk.reward.AFRewardActivity" />
		
		<!-- If you need a push notification feature, add following codes -->
		<activity android:name="com.adfresca.ads.AdFrescaPushActivity" />
		<receiver android:name="com.adfresca.unity.GCMReceiver" android:permission="com.google.android.c2dm.permission.SEND" >
		  <intent-filter>
		    <action android:name="com.google.android.c2dm.intent.RECEIVE" />
		    <action android:name="com.google.android.c2dm.intent.REGISTRATION" />
		    <category android:name="com.Company.ProductName" />
		  </intent-filter>
		</receiver>
		<service android:name="com.adfresca.unity.GCMIntentService" />  <!-- You must create your own GCMIntentService class to handle GCM messages  -->	    	
	</application>
	
	<uses-feature android:glEsVersion="0x00020000" />
	<uses-sdk android:minSdkVersion="6" android:targetSdkVersion="16" />
	
 	<uses-permission android:name="android.permission.INTERNET"/>
 	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
	
 	<!-- If you need a push notification feature, add following permissions -->
 	<permission android:name="com.Company.ProductName.permission.C2D_MESSAGE" android:protectionLevel="signature" />
	<uses-permission android:name="com.Company.ProductName.permission.C2D_MESSAGE" />
	<uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
	<uses-permission android:name="android.permission.GET_ACCOUNTS" />
	<uses-permission android:name="android.permission.WAKE_LOCK" />
</manifest>
```

기본적으로 Android SDK 가이드에 따라 필요한 permission과 service를 적용합니다. 'com.Company.ProductName'로 표기된 패키지명을 모두 알맞은 값으로 수정합니다.

 
#### 별도의 GCM GCMReceiver 및 GCMIntentService 적용하기 (Advanced)


Android 용 플러그인에는 GCMReceiver, GCMIntentService 파일이 이미 포함이 되어 있습니다.

만약 GCM 등록이나 메시지 핸들링을 직접 하고 싶은 경우, 별도로 클래스 파일을 생성하여 AndroidManifest.xml를 수정합니다.

더 자세한 내용은 _Android SDK_ 가이드의 [Push Notification 설정하기- 4) GCMIntentService 클래스 구현하기](https://github.com/adfresca/sdk-android-sample#push-notification)를 참고하여 적용합니다.

### iOS

iOS의 경우는 Native SDK와 동일한 설치 작업을 거칩니다.  모든 플러그인 구성 요소가 Import 되었는지 확인 후, Unity에서 Xcode 프로젝트를 빌드합니다. 

우선, iOS  SDK 설치 가이드의 '1. SDK 설치' 항목을 따라서 설치 작업을 진행합니다.

그리고 아래의 내용을 AppController.mm 파일에 적용합니다.

```mm
#import <AdFresca/AdFrescaView.h>

  - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    [AdFrescaView startSession:@"YOUR_API_KEY"];

    [[UIApplication sharedApplication] registerForRemoteNotificationTypes:UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge |UIRemoteNotificationTypeSound];   // Push Notification 기능을 이용할 경우 등록.      

  } 

  

  // Push Notification 기능을 사용할 경우 아래 코드를 삽입합니다. 자세한 내용은 '8. Push Notification 설정하기' 항목을 참고해 주세요.

  - (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {

    [AdFrescaView registerDeviceToken:deviceToken];

  }

 

  - (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {

    if ([AdFrescaView isFrescaNotification:userInfo] && [application applicationState] != UIApplicationStateActive) {

      [AdFrescaView handlePushNotification:userInfo];

    }

  } 
```

iOS 플러그인 설치가 완료 되었습니다.

이후에 진행되는 플러그인 적용 작업부터는 더이상 Native 코드 없이 모두 Unity 코드만으로 진행됩니다.

### Code

_Plugin_은 아래 코드 처럼 단 몇줄의 코드만으로도 캠페인을 시작할 수 있습니다.

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
		Plugin plugin = Plugin.Instance;
		plugin.Init(API_KEY);
		plugin.StartSession();
	}

	void OnGUI ()
	{
		if (GUI.Button (new Rect (100, 100, 150, 150), "Load & Show")) {
			Plugin plugin = Plugin.Instance;
			plugin.Load();
			plugin.Show();
		}
	}
}
```

`plugin.Init(API_KEY);` API Key 를 설정합니다.

`plugin.StartSession();` 세션이 시작됨을 서버에 알립니다. 어플리케이션이 시작될 때 **한번만** 실행되도록 합니다.

`plugin.Load();` 서버로부터 컨텐츠를 내려받습니다.

`plugin.Show();` 내려받은 컨텐츠를 보여줍니다.

정상적으로 실행이 되면 다음과 같은 화면이 보여집니다.

<img src="https://adfresca.zendesk.com/attachments/token/zngvftbmcccyajk/?name=device-2013-03-18-133517.png" width="240" />
&nbsp;
<img src="https://adfresca.zendesk.com/attachments/token/phn4fcpvbi2damx/?name=device-2013-03-18-133443.png" height="240" />
* * *

## Basic Features

### In-App Purchase Count

사용자가 In-App Purchase를 구매한 경우, _AD fresca_에 해당 정보를 기록하여 관리하실 수 있습니다.

사용자가 In-App Purchase 를 구매한 횟수를  `AdFresca.Plugin` 객체의 `SetNumberOfInAppPurchases(int)` 메소드를 사용하여  설정해 주시면 간단히 적용이 가능합니다.

```cs
Plugin plugin = Plugin.Instance;
plugin.Init(API_KEY);
plugin.SetNumberOfInAppPurchases(user.GetInAppPurchaseCount());
plugin.StartSession();
plugin.Load();
plugin.Show();
```

위와 같은 방식으로 호출이 가능합니다.

정확한 기록을 위해 반드시 앱이 실행되었다고 판단되는 시점에서 호출하는 것을 권장합니다.

추후 보다 편하게 해당 기능을 이용하실 수 있도록 지원해드릴 예정입니다.

### Custom Parameter

_AD fresca_는 앱 사용자의 특수한 정보들 (레벨, 스테이지, 성별, 나이 등)을 입력 받아 타겟팅 및 분석 기능을 제공 합니다.

_Unity Plugin_에서는 `SetCustomParameter` 메소드를 사용하여 각 커스텀 파라미터의 인덱스 번호에 맞게 값을 설정하면 됩니다.

(각 파라미터의 정보는 Admin 사이트를 접속하여 앱의 Overview 메뉴 -> 각 앱스토어의 Details 버튼을 눌러 설정 및 확인이 가능합니다.)

```cs
Plugin plugin = Plugin.Instance;
plugin.Init(API_KEY);
plugin.SetCustomParameter(CUSTOM_PARAM_INDEX_LEVEL, User.level);
plugin.SetCustomParameter(CUSTOM_PARAM_INDEX_AGE, User.age);
plugin.SetCustomParameter(CUSTOM_PARAM_INDEX_HAS_FB_ACCOUNT, User.hasFacebookAccount);
plugin.StartSession();
plugin.Load();
plugin.Show();
```

### Event

Event 기능을 사용하면 앱에서 일어나는 다양한 사용자들의 활동, 페이지 이동 등에 Event를 설정한 후 그러한 Event 발생 시에 그에 적합한 공지사항, 컨텐츠 등을 노출할 수 있습니다.

Event 설정은 Admin 을 통해 가능하며 '[Event 설정하기](https://adfresca.zendesk.com/entries/23359141)' 가이드를 참고해주시기 바랍니다.

Event 설정하신 후, _Plugin_ 적용을 위해서는 각 Event 'Index' 값이 필요합니다. Index 값은 1,2,3,4 와 같은 Integer 형태의 고유 값이며 소스코드에 Constant 형태로 미리 입력하여 이용하시는 것을 권장합니다.

각 Event 발생 시, load() 메소드에 원하는 Event의  Index 값을 인자로 넘겨주시면 간단히 적용이 완료됩니다.

_(기존의 ['AD Slot 지정하기](https://adfresca.zendesk.com/entries/23359131)' 기능은 Deprecated 되어 현재 Event로 대체 되었습니다. 자세한 내용은 SDK Changed Log를 확인하여 주세요. )_


**Example**:  사용자가 메인 페이지로 이동할 시에 설정한 컨텐츠를 노출

```cs
Plugin plugin = Plugin.Instance;
plugin.Load(EVENT_INDEX_MAIN_PAGE);  // 메인 페이지에 설정한  컨텐츠를 노출
plugin.Show();
```

**Example**: 사용자의 게임 캐릭터가 레벨업을 했을 때 설정한 컨텐츠를 노출

```cs
Plugin plugin = Plugin.Instance;
plugin.SetCustomParameter(CUSTOM_PARAM_INDEX_LEVEL, level); // 사용자 level 정보를 가장 최신으로 업데이트
plugin.Load(EVENT_INDEX_LEVEL_UP); // 레벨업 이벤트에 설정한 컨텐츠를 노출
plugin.Show();
```

* * *

## Push Notification

_AD fresca_를 통해 Push Notification을 보내고 받을 수 있습니다.

SDK를 적용하기 이전에 구글의 ["GCM: Getting Started" ](http://developer.android.com/google/gcm/gs.html)가이드 문서를 읽어보시길 권장합니다.

1) GCM Helper Library 설치하기.
    - 구글에서 제공하는 [GCM Helper Library](http://code.google.com/p/gcm/source/browse/) 를 다운로드 받습니다. (Download zip 혹은 git clone을 이용)
    - /gcm-client/dist/**gcm.jar** 파일을 프로젝트에 복사하여 설치합니다.
    
2) AndroidManifest.xml 확인하기.

```xml
<manifest>   
  <application>
      .........
      <activity android:name="com.adfresca.ads.AdFrescaPushActivity" />
      <receiver android:name="com.google.android.gcm.GCMBroadcastReceiver"
        android:permission="com.google.android.c2dm.permission.SEND">  
        <intent-filter>
          <action android:name="com.google.android.c2dm.intent.RECEIVE" />
          <action android:name="com.google.android.c2dm.intent.REGISTRATION" />
          <category android:name="your_app_package" />
         </intent-filter>
      </receiver>
      <service android:name=".GCMIntentService" />  <!-- GCM 메시지를 처리하기 위하여 GCMIntentService 클래스를 구현해야 합니다 -->
   </application>
    ..........
    <permission android:name="your_app_pakcage.permission.C2D_MESSAGE" android:protectionLevel="signature" />
    <uses-permission android:name="your_app_package.permission.C2D_MESSAGE" />
    <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
    <uses-permission android:name="android.permission.GET_ACCOUNTS" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    ..........
</manifest>
```

3) Registration ID를 등록하고 SDK에 셋팅하기.

```java
    /*
    * GCM_SENDER_ID는 Google API project number를 의미 합니다.
    * https://code.google.com/apis/console/#project:1234567890
    */
   final String GCM_SENDER_ID = "1234567890";
   
   GCMRegistrar.checkDevice(this);
   GCMRegistrar.checkManifest(this);
   
   final String gcmDeviceId = GCMRegistrar.getRegistrationId(this);  
   
   if (regId.equals("")) {
     GCMRegistrar.register(this, GCM_SENDER_ID);          
   }
   
  AdFresca adfresca = AdFresca.getInstance(this);
  adfresca.setPushRegistrationId(gcmDeviceId);
  adfresca.startSession();
```

4) GCMIntentService 클래스 구현하기

```java
  public class GCMIntentService extends GCMBaseIntentService {

    /*
    * GCM_SENDER_ID는 Google API project number를 의미 합니다.
    * https://code.google.com/apis/console/#project:1234567890
    */
    private static final String GCM_SENDER_ID = "1234567890";

    public GCMIntentService() {
      super(GCM_SENDER_ID);
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

      // AD fresca를 통해서 수신한 notification인지 확입합니다.
      if (AdFresca.isFrescaNotification(intent)) { 
        String title = context.getString(R.string.app_name);
        int icon = R.drawable.icon;
        long when = System.currentTimeMillis();

        // 수신 받은 notification을 status bar에 표시합니다.
        // notification에 URI Schema가 설정된 경우, 해당 URI를 실행하며 기본적으로는 targetClass에 설정한 액티비티를 실행하여 줍니다. 
        AdFresca.showNotification(context, intent, MainActivity.class, title, icon, when);

      } 

    }

    @Override
    protected void onError(Context context, String registrationId) {

    }
  }
```

### Custom Notification

아래 코드는 Push Notification 을 만들고 직접 notify 하는 방법입니다.

```java
public class GCMIntentService extends GCMBaseIntentService {
	@Override
	protected void onMessage(Context context, Intent intent) {
		if (AdFresca.isFrescaNotification(intent)) {
			String title = context.getString(R.string.app_name);
			int icon = R.drawable.icon;
			long when = System.currentTimeMillis();
			Notification notification = AdFresca.generateNotification(context, intent, DemoIntroActivity.class, title, icon, when);
			notification.sound = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION);
			NotificationManager notificationManager = (NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);
			notificationManager.notify(0, notification);
		}
	}
}
```

* * *

## Reward Item

_Incentivized Campaign_을 사용하여 , 사용자가 _Media App_에서 _Advertising App_의 광고를 보고 앱을 설치하였을 때 보상으로 _Media App_의 아이템을 지급할 수 있습니다.

(현재 CPI  캠페인을 진행할 경우, _Advertising App_의 SDK 설치는 필수가 아닙니다. 하지만 이후 지원할 CPA 캠페인을 위해서 미리 SDK를 설치하는 것을 권장합니다.)

Media App에 SDK 적용하기:

- AndroidManifest.xml 확인하기

```xml
<manifest>   
  <application>
      .........
      <activity android:name="com.adfresca.sdk.reward.AFRewardActivity" />
      .........
   </application>
</manifest>
```

- 아이템 지급을 원하는 위치에서 `getAvailableRewardItems()` 메소드를 통해 아이템 리스트를 받습니다. 

- Array에는 `AFRewardItem` 객체들이 포함되어 있으며 name, quantity, uniqueValue 프로퍼티 값을 가지고 있습니다.

- `getAvailableRewardItems()` 메소드는 현재 지급이 가능한 아이템 리스트를 리턴하며, 아직 검사가 끝나지 않은 경우 이후 메소드 호출 시에 반영됩니다.

```java
@Override
public void onStart() {
  super.onStart();

  AdFresca adfresca = AdFresca.getInstance(DemoIntroActivity.this);
  List<AFRewardItem> items = adfresca.getAvailableRewardItems();

  if (items.size() > 0) {
    for (AFRewardItem item : items) {
      Log.d(TAG,String.format("Get AFRewardItem: name=%s, quantity=%d, uniqueVlaue=%s", item.getName(), item.getQuantity(), item.getUniqueValue())));
      // do something with tem.getName(), item.getQuantity(), item.getUniqueValue()
    }
    String itemNames = joinNameStringsByComma(items);
    String alertMessage = String.format("You got the reward item(s)! (%s)", itemNames);
    showAlert(alertMessage);
  }
}
```

###(Advanced) 더욱 빠르게 아이템 지급하기:

`getAvailableRewardItems()` 메소드는 현재 지급 가능한 아이템 리스트를 리턴한 이후, 새롭게 지급 가능한 아이템들이 있는지 백그라운드로 검사를 진행하게 됩니다. 만약 앱 시작시에 미리 검사를 수동으로 진행하고 원하는 위치에서 `getAvailableRewardItems()` 메소드를 호출한다면, 사용자들에게 더욱 빠른 아이템 지급이 가능해집니다.

```java
@Override
public void onStart() {
  super.onStart();

  AdFresca adfresca = AdFresca.getInstance(DemoIntroActivity.this);
  adfresca.checkRewardItems();
}

@Override
public void onClick(View view) {
  List<AFRewardItem> items = adfresca.getAvailableRewardItems();

  if (items.size() > 0) {
    for (AFRewardItem item : items) {
      Log.d(TAG,String.format("Get AFRewardItem: name=%s, quantity=%d, uniqueVlaue=%s", item.getName(), item.getQuantity(), item.getUniqueValue())));
      // do something with tem.getName(), item.getQuantity(), item.getUniqueValue()
    }
    String itemNames = joinNameStringsByComma(items);
    String alertMessage = String.format("You got the reward item(s)! (%s)", itemNames);
    showAlert(alertMessage);
  }
}
```

`checkRewardItems(synchronized)` 메소드를 Synchronized 모드로 실행하면,, SDK가 모든 검사를 완료할 때 까지 기다린 후 바로 아이템을 지급할 수도 있습니다.

```java
new AsyncTask<Void, Void, Void>() {
  protected Void doInBackground(Void... params) {
    AdFresca adfresca = AdFresca.getInstance(DemoIntroActivity.this);
    adfresca.checkRewardItems(true);
    return null;
  }

  protected void onPostExecute(Void param) {
    AdFresca adfresca = AdFresca.getInstance(DemoIntroActivity.this);
    List<AFRewardItem> items = adfresca.getAvailableRewardItems();

    if (items.size() > 0) {
      for (AFRewardItem item : items) {
      Log.d(TAG,String.format("Get AFRewardItem: name=%s, quantity=%d, uniqueVlaue=%s", item.getName(), item.getQuantity(), item.getUniqueValue())));
      // do something with tem.getName(), item.getQuantity(), item.getUniqueValue()
      }
      String itemNames = joinNameStringsByComma(items);
      String alertMessage = String.format("You got the reward item(s)! (%s)", itemNames);
      showAlert(alertMessage);
    }
  }
}.execute();
```

* * *

## Advanced Features

### Test Device ID

_AD fresca_는 테스트 모드 기능을 지원하며 테스트에 사용할 수 있는 기기를 별도로 등록하여 관리할 수 있습니다.

테스트 기기 ID는 SDK를 통해 추출이 가능하며 2가지 방법을 지원 합니다.


1. testDeviceId를 얻어와서 원하는 곳에 출력하는 방법

```java
AdFresca adfresca = AdFresca.getInstance(this);
String deviceId = adfresca.getTestDeviceId();
textView.setText(deviceId);
```

2. printTestDeviceId Property를 설정하여 화면에 Device ID를 표시하는 방법
 
```java
  AdFresca adfresca = AdFresca.getInstance(this);
  Log.d(TAG, "AD fresca Test Device ID is = " + adfresca.getTestDeviceId());
  adfresca.setPrintTestDeviceId(true);
  adfresca.load();
  adfresca.show();
```

### Timeout Interval

컨텐츠의 최대 로딩 시간을 직접 지정하실 수 있습니다. 지정된 시간 내에 컨텐츠가 로딩되지 못한 경우, 사용자에게 컨텐츠를 노출하지 않습니다.

최소 1초 이상 지정이 가능하며, 지정하지 않을 시 기본 값으로 5초가 지정 됩니다.

```java
  AdFresca adfresca = AdFresca.getInstance(this);
  AdFresca.setTimeoutInterval(5) // # 5 seconds
  adfresca.load();
  adfresca.show();
```

* * *

## Release Notes
- v2.0.0 _(07/10/2013 Updated)_
    - _Incentivized CPI_캠페인을 위한 API 가 추가되었습니다.
