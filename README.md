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

Unity 엔진에서 _AD fresca SDK_를 사용할 수 있도록 공식 플러그인을 제공합니다.  

Unity Package 파일을 통해 모든 구성요소를 쉽게 설치할 수 있으며, 아래 적용 가이드에 따라 간단한 코드만으로  Unity 환경에서도 SDK 적용이 가능하게 되었습니다.

플러그인 제작에 사용된 모든 Wrapper 코드가 공개되며, 필요한 경우 각 게임에 알맞게 수정하여 적용할 수 있습니다. 

* * *

## Quick Start

### Installation

아래 링크를 통해 _Unity Plugin_을 다운로드 합니다.

[AD fresca Unity Plugin v2.1.6 다운로드](https://s3-ap-northeast-1.amazonaws.com/file.adfresca.com/distribution/sdk-for-Unity.zip) (Android SDK v2.3.2, iOS SDK v1.3.4)

Unity 프로젝트를 열고 AdFrescaUnityPlugin.package 파일을 실행합니다.

아래의 구성 요소들이 Assets 폴더 아래에 복사되어야 합니다.

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
		<meta-data android:name="unityplayer.ForwardNativeEventsToDalvik" android:value="true" />
		</activity>
		
		<!-- OpenUDID 서비스 등록 -->
		<service android:name="org.openudid.OpenUDID_service">
	          <intent-filter>
	            <action android:name="org.openudid.GETUDID" />
	          </intent-filter>
		</service>
		
		<!-- Google Refererer Tracking 을 위한 Boradcast Receiver-->
		<receiver android:name="com.adfresca.sdk.referer.AFRefererReciever" android:exported="true">
    		  <intent-filter>
      	 	    <action android:name="com.android.vending.INSTALL_REFERRER" />
     		  </intent-filter>
		</receiver>
		
		<!-- Incentivized Campaign 을 위한 액티비티-->
		<activity android:name="com.adfresca.sdk.reward.AFRewardActivity" />
		
		<!-- Push Notification 기능을 사용하기 위하여 아래 내용을 추가 -->
		<activity android:name="com.adfresca.ads.AdFrescaPushActivity" />
		<receiver android:name="com.Company.ProductName.CustomGCMReceiver" android:permission="com.google.android.c2dm.permission.SEND" >
		  <intent-filter>
		    <action android:name="com.google.android.c2dm.intent.RECEIVE" />
		    <action android:name="com.google.android.c2dm.intent.REGISTRATION" />
		    <category android:name="com.Company.ProductName" />
		  </intent-filter>
		</receiver>
		<service android:name="com.Company.ProductName.GCMIntentService" />  <!-- GCM 메시지를 처리하기 위하여 GCMReceiver, GCMIntentService 클래스를 직접 구현해야 합니다.  -->    	
	</application>
	
	<uses-feature android:glEsVersion="0x00020000" />
	<uses-sdk android:minSdkVersion="6" android:targetSdkVersion="16" />
	
 	<uses-permission android:name="android.permission.INTERNET"/>
 	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
	
 	<!-- Push Notification 기능을 사용하기 위하여 아래 내용을 추가 -->
 	<permission android:name="com.Company.ProductName.permission.C2D_MESSAGE" android:protectionLevel="signature" />
	<uses-permission android:name="com.Company.ProductName.permission.C2D_MESSAGE" />
	<uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
	<uses-permission android:name="android.permission.GET_ACCOUNTS" />
	<uses-permission android:name="android.permission.WAKE_LOCK" />
</manifest>
```

위와 같이 SDK 적용에 필요한 permission과 service를 등록합니다. 'com.Company.ProductName'로 표기된 패키지명은 모두 알맞은 값으로 수정합니다.
GCMReceiver, GCMIntentService 클래스의 구현은 아래의 [Push Notification](#push-notification) 항목에서 진행합니다.

### iOS

iOS의 경우는 Native SDK와 동일한 설치 작업을 진행합니다.  모든 플러그인 구성 요소가 Import 되었는지 확인 후, Unity에서 Xcode 프로젝트를 빌드합니다. 

우선, iOS  SDK 설치 가이드의 ['1. SDK 설치'](https://adfresca.zendesk.com/entries/21346861#installation) 항목을 따라서 설치 작업을 진행합니다.

그리고 아래의 내용을 AppController.mm 파일에 적용합니다.

```mm
#import <AdFresca/AdFrescaView.h>

  - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [AdFrescaView startSession:@"YOUR_API_KEY"];
    [[UIApplication sharedApplication] registerForRemoteNotificationTypes:UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge |UIRemoteNotificationTypeSound];   // Push Notification 기능을 위하여 등록
  } 

  // Push Notification 기능을 위하여 아래 내용을 추가합니다.

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

### Code

기본적으로 앱의 세션을 기록하고, 매칭되는 캠페인의 컨텐츠를 노출하기 위해서는 아래와 같이 코드를 적용합니다.

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

`plugin.Init(API_KEY);` API Key 를 설정합니다.

`plugin.StartSession();` 세션이 시작됨을 서버에 알립니다. 어플리케이션이 시작될 때 **한 번만** 실행되도록 합니다.

`plugin.Load();` 서버로부터 컨텐츠를 내려받습니다.

`plugin.Show();` 내려받은 컨텐츠를 보여줍니다.

정상적으로 실행이 되면 다음과 같은 화면이 보여집니다.

<img src="https://adfresca.zendesk.com/attachments/token/2fv9e76ptp7yo3h/?name=android-sample-p.png" width="240" />
&nbsp;
<img src="https://adfresca.zendesk.com/attachments/token/phn4fcpvbi2damx/?name=device-2013-03-18-133443.png" height="240" />
* * *

## Basic Features

### In-App Purchase Count

사용자가 In-App Purchase를 구매한 경우, _AD fresca_에 해당 정보를 기록하여 관리하실 수 있습니다.

사용자가 In-App Purchase 를 구매한 횟수를  `AdFresca.Plugin` 객체의 `SetNumberOfInAppPurchases(int)` 메소드를 사용하여  설정해 주시면 간단히 적용이 가능합니다.

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

**주의:** SetNumberOfInAppPurchases() 메소드는 StartSession(), Load() 메소드 이전에 호출이 되어야 합니다. 만약 StartSession() 이전에 값이 설정되지 않은 경우, 사용자의 최초 앱 실행 시에는 값이 업데이트 되지 않으며 2회째 실행부터 SDK가 로컬 캐싱해둔 값으로 서버에 전달됩니다. (SDK 로컬 캐시 기능은 Android SDK 2.2.0, iOS SDK 1.3.1 버전부터 지원됩니다.)

### Custom Parameter

_AD fresca_는 앱 사용자의 특수한 정보들 (레벨, 스테이지, 성별, 나이 등)을 입력 받아 타겟팅 및 분석 기능을 제공 합니다.

_Unity Plugin_에서는 `SetCustomParameter` 메소드를 사용하여 각 커스텀 파라미터의 인덱스 번호에 맞게 값을 설정하면 됩니다.

(각 파라미터의 정보는 Admin 사이트를 접속하여 앱의 Overview 메뉴 -> 각 앱스토어의 Details 버튼을 눌러 설정 및 확인이 가능합니다.)

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
**주의**_ SetCustomParameter() 메소드는 StartSession(), Load() 메소드 이전에 호출이 되어야 합니다. 특히 StartSession() 이전에는 반드시 모든 커스텀 파리미터 값들을 설정하고, 이후 변경되는 값들에 한하여 각 위치에 커스텀 파라미터를 설정합니다.

만약 불가피하게 StartSession() 호출 시에 커스텀 파라미터 값을 설정할 수 없는 경우, 앱을 최초로 실행한 사용자의 프로파일은 업데이트되지 않으며 해당 사용자의 2회째 앱 실행부터 SDK가 로컬에 캐싱해둔 값이 전달됩니다. 최초로 실행된 사용자의 프로파일까지 통계 및 타겟팅하기 위해서는 아래와 같이 초기 값 설정을 진행해야 합니다. (SDK의 로컬 캐싱 기능은 Android SDK 2.2.0, iOS SDK 1.3.1 버전부터 지원합니다.)

```cs
void Start() {
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.Init(API_KEY);
  if (isFirstRunUser)
  {
    plugin.SetCustomParameter(CUSTOM_PARAM_INDEX_LEVEL, defaultLevel);
    plugin.SetCustomParameter(CUSTOM_PARAM_INDEX_STAGE, defaultStage);
    plugin.SetCustomParameter(CUSTOM_PARAM_INDEX_HAS_FB_ACCOUNT, defaultFacebookFlag);
  }
  plugin.StartSession();
}

.....

void onUserSignedIn() {
  User.level = level
  
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.SetCustomParameter(CUSTOM_PARAM_INDEX_LEVEL, User.level);
  plugin.SetCustomParameter(CUSTOM_PARAM_INDEX_AGE, User.age);
  plugin.SetCustomParameter(CUSTOM_PARAM_INDEX_HAS_FB_ACCOUNT, User.hasFacebookAccount);
}
```

### Event

Event 기능을 사용하면 앱에서 일어나는 다양한 사용자들의 활동, 페이지 이동 등에 Event를 설정한 후 그러한 Event 발생 시에 그에 적합한 공지사항, 컨텐츠 등을 노출할 수 있습니다.

Event 설정은 Admin 을 통해 가능하며 '[Event 설정하기](https://adfresca.zendesk.com/entries/23359141)' 가이드를 참고해주시기 바랍니다.

Event 설정하신 후, _Plugin_ 적용을 위해서는 각 Event 'Index' 값이 필요합니다. Index 값은 1,2,3,4 와 같은 Integer 형태의 고유 값이며 소스코드에 Constant 형태로 미리 입력하여 이용하시는 것을 권장합니다.

각 Event 발생 시, load() 메소드에 원하는 Event의  Index 값을 인자로 넘겨주시면 간단히 적용이 완료됩니다.

_(기존의 ['AD Slot 지정하기](https://adfresca.zendesk.com/entries/23359131)' 기능은 Deprecated 되어 현재 Event로 대체 되었습니다. 자세한 내용은 SDK Changed Log를 확인하여 주세요. )_


**Example**:  사용자가 메인 페이지로 이동할 시에 설정한 컨텐츠를 노출

```cs
AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
plugin.Load(EVENT_INDEX_MAIN_PAGE);  // 메인 페이지에 설정한  컨텐츠를 노출
plugin.Show();
```

**Example**: 사용자의 게임 캐릭터가 레벨업을 했을 때 설정한 컨텐츠를 노출

```cs
AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
plugin.SetCustomParameter(CUSTOM_PARAM_INDEX_LEVEL, level); // 사용자 level 정보를 가장 최신으로 업데이트
plugin.Load(EVENT_INDEX_LEVEL_UP); // 레벨업 이벤트에 설정한 컨텐츠를 노출
plugin.Show();
```

* * *

## Push Notification

AD fresca를 통해 Push Notification을 보내고 받을 수 있습니다.

#### Android

SDK를 적용하기 이전에 구글의 ["GCM: Getting Started"](http://developer.android.com/google/gcm/gs.html) 가이드 문서를 읽어보시길 권장합니다.

1) AndroidManifest.xml 확인하기

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

2) GCMIntentService 클래스 구현하기

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
            String title = AdFrescaPlugin.getAppName(context);
            int icon = R.drawable.app_icon;
            long when = System.currentTimeMillis();
            Class<?> targetActivityClass = null;
            
            if (UnityPlayer.currentActivity != null) {
            	targetActivityClass = UnityPlayer.currentActivity.getClass();
            } else {
            	targetActivityClass = YourMainActivity.class; // or UnityPlayer.class
            }
            
            AdFresca.showNotification(context, intent, targetActivityClass, title, icon, when);
        }                
    }
    
    @Override
    protected void onError(Context context, String registrationId) {
    
    }
  }
```

showNotification() 메소드는 가장 기본적인 Notification 뷰를 이용하여 아무런 사운드 없이 메시지를 표시합니다. Notification에 사운드를 설정하거나, Big View와 같은 커스터마이징 작업이 필요한 경우 Android SDK 가이드의 ["Custom Notification"](https://github.com/adfresca/sdk-android-sample/blob/master/README.md#custom-notification) 내용을 참고하여 주시기 바랍니다.

3) GCMReceiver 클래스 구현하기

```java
public class GCMReceiver extends GCMBroadcastReceiver { 
   	@Override
	protected String getGCMIntentServiceClassName(Context context) { 
		return "com.Company.ProductName.GCMIntentService"; 
	} 
}
```

4) Unity 환경에서 GCM 적용하기

이제 유니티 클래스에서 GCM Sender ID 값을 플러그인에 적용합니다. Sender ID 값은 API Key 값과 다른 Project Number 값 입니다. 

```cs
#if UNITY_ANDROID
private static string GCM_SENDER_ID = "12345678"; // Google API Proejct Number (ex: 12345678)
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

SDK를 적용하기 이전에 애플의 ["Local and Push Notification Programming Guide"](https://developer.apple.com/library/mac/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Introduction.html) 가이드 문서를 읽어보시길 권장합니다. 

(현재 AD fresca의 iOS Push Notification 서비스는 APNS의 Production 환경만을 지원하며, 추후 업데이트를 통해 Development 환경을 추가로 지원할 예정입니다.)

1)  Push Notification 인증서 파일을 생성하고 Admin 사이트에 등록합니다.
- ["iOS Push Notification 인증서 설정 및 적용하기 가이드"](https://adfresca.zendesk.com/entries/21714780) 를 따라 Production용 Push Notification Certificate를 생성하고 설치합니다.
- Keychain을 통해 export된 p12 파일을 AD fresca Admin 사이트에 등록 합니다. (해당 앱의 Overview -> iOS App Store Edit -> Push Authentication)

2) Info.plast 확인하기 / Provision 확인하기
- Info.plst 파일의 'aps-environment' 값을 'production' 으로 설정합니다. 
- App Store / Ad Hoc release에 사용하는 Provision 인증서를 사용하여 빌드해야 합니다.

3) AppController.mm 파일의 이벤트 확인하기

```mm
#import <AdFresca/AdFrescaView.h>

  - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [AdFrescaView startSession:@"YOUR_API_KEY"];
    [[UIApplication sharedApplication] registerForRemoteNotificationTypes:UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge |UIRemoteNotificationTypeSound];   // Push Notification 기능을 위하여 등록
  } 

  - (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
    [AdFrescaView registerDeviceToken:deviceToken];
  }
  
  - (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
    // AD fresca를 통해 전송된 메시지 여부를 확인하고 앱이 이미 실행 중인 경우에는 동작하지 않도록 합니다.
    if ([AdFrescaView isFrescaNotification:userInfo] && [application applicationState] != UIApplicationStateActive) {
      [AdFrescaView handlePushNotification:userInfo];
    }
  } 
```

iOS는 별도로 유니티에서 설정할 작업이 없습니다.

이로써 Push Notification 기능을 위한 적용 작업이 모두 완료되었습니다.

* * *

## Custom URL

Announcement 캠페인의 Click URL, Push Notification 캠페인의 URL Schema 설정 시에 자신의 앱 URL Schema를 사용할 수 있습니다. 이를 통해 사용자가 콘텐츠를 클릭할 경우, 자신이 원하는 특정 앱 페이지로 이동하는 등의 액션을 지정할 수 있습니다.

#### Android 환경에서 Custom URL 적용하기

네이티브 애플리케이션 개발 환경에서는 AndroidManifest.xml 파일을 수정하여 원하는 액티비티에 scheme 정보를 추가하는 방식으로 적용이 됩니다.

하지만 액티비티를 페이지 개념으로 사용하는 네이티브 환경과 달리, Unity 엔진을 사용하여 안드로이드 애플리케이션을 개발하는 경우 단 하나의 UnityPlayer 액티비티만을 사용하며 엔진 내부적으로 페이지를 처리합니다.

때문에 schema를 지정할 수 있는 액티비티의 제약이 생깁니다. MAIN 으로 지정된 UnityPlayer 액티비티는 url schema를 적용할 수 없습니다. 그래서 아래와 같은 방법들을 사용하여 Custom URL을 처리합니다.

1) UnityPlayer 액티비티의 startActivity(intent) 메소드를 오버라이딩하여 Custom URL 처리하기 (Annoucnement 캠페인)

Annoucnement 캠페인을 통해 전달되는 Click URL은 항상 인게임 상황에서 전달되며, SDK가 내부적으로 startActivity() 메소드를 이용하여 호출하고 있습니다. 이러한 조건에서는 게임이 실행되고 있는 UnityPlyaer 액티비티의 startActivity() 메소드를 직접 구현함으로써 Custom URL 처리가 가능합니다.

먼저 Eclipse에서 Android Project를 생성하여 UnityPlayerActivity를 상속받은 'Main Actvity' 클래스를 생성합니다. 그리고 AndroidMenefest.xml 파일을 아래와 같이 수정합니다.

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
startActivity() 메소드를 아래와 같이 구현합니다. 'myapp://' 으로 적용된 Custom URL이 전달된다면 새로 액티비티를 호출하지 않고 미리 설정한 게임 오브젝트로 값을 전달합니다.

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

2) Push Notification을 통해 넘어오는 Custom URL 처리하기 (Push Notificiaton 캠페인)

Custom URL이 설정된 Push Notification을 수신한 경우, Notification을 터치 시 원하는 액션을 지정할 수 있습니다. 단, 이 경우는 인게임 상황이 아니기 때문에 조금 다른 방법을 사용합니다.

먼저 PushProxyActivity 라는 이름의 액티비티 클래스를 하나 생성합니다. 그리고 AndroidMenefest.xml 내용을 아래와 같이 추가합니다. 

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
위와 같이 설정한 경우 Push Notificaiton 캠페인에서는 myapp://com.adfresca.push?item=abc 와 같은 형식의 url을 입력해야 합니다.

다음은 PushProxyActivity 클래스의 내용을 구현해야 합니다. PushProxyActivity 클래스는 Android OS로 부터 수신하는 Custom URL 정보를 받아 처리하고 바로 자신을 종료하는 단순한 프록시 형태의 액티비티입니다. 만약 현재 UnityPlayer가 실행 중이 아니라면 Custom URL을 처리할 수 없으므로 새로 UnityPlayer를 실행하여 uri 값을 넘겨야 합니다.

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
마지막으로 PushProxyActivity를 통해 게임이 실행된 경우 넘어오는 uri 값을 처리합니다. Main 액티비티에 아래와 같은 내용을 추가합니다.

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

Android 플랫폼 환경에서 Custom URL을 처리할 수 있는 모든 방법을 구현하였습니다.

#### iOS 환경에서 Custom URL 적용하기

iOS의 경우 1개의 이벤트체서 모든 URL 처리가 가능하기 때문에 비교적 간단하게 적용할 수 있습니다.

1) Info.plst 파일을 열어 사용할 URL Schema 정보를 설정 합니다.

<img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png" />

2) AppController.mm 파일을 열어 handleOpenURL 메소드를 구현합니다. 호출되는 URL 값을 유니티 게임 오브젝트에 전달합니다. 

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

### Unity에서 url 값 확인 및 처리하기

위 예제에서는 모두 'Fresca' 게임 오브젝트의 'OnCustomURL' 이벤트 메소드를 호출하여 값을 전달하였습니다. 이제 Unity 게임 상에서 그 값을 직접 받아 확인하고 처리할 수 있습니다.

```java
public void OnCustomURL(string url)
{
	Debug.Log("OnCustomURL = " + url); 
	//ex) myapp://com.adfresca.custom?item=abc 값이 전달된 경우 item=abc 값을 파싱하여 아이템 지급
}
```

* * *

## Reward Item

_Incentivized Campaign_을 사용하여 , 사용자가 _Media App_에서 _Advertising App_의 광고를 보고 앱을 설치하였을 때 보상으로 _Media App_의 아이템을 지급할 수 있습니다.

(현재 CPI  캠페인을 진행할 경우, _Advertising App_의 SDK 설치는 필수가 아닙니다. 하지만 이후 지원할 CPA 캠페인을 위해서 미리 SDK를 설치하는 것을 권장합니다.)

**Media App에 SDK 적용하기**

- /Plugins/Android/AndroidManifest.xml 확인하기

```xml
<manifest>   
  <application>
      .........
      <activity android:name="com.adfresca.sdk.reward.AFRewardActivity" />
      .........
   </application>
</manifest>
```

**Advertising App 설정하기(iOS)**

Info.plst 파일을 열어 사용할 URL Schema 정보를 설정 합니다.

<img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png"/>

위 경우 어드민 Dashboard에서 해당 앱의 CPI Identifier 값을 'myapp://' 으로 설정하게 됩니다. 

iOS 플랫폼의 경우 URL Schema 값이 다른 앱과 중복될 수 있습니다. 정상적인 CPI 캠페인을 위해서는 최대한 Unique한 값을 선택해야 합니다.

**Code**

- 아이템 지급을 원하는 위치에서 `GetAvailableRewardItems()` 메소드를 통해 아이템 리스트를 받습니다. 

- Array에는 `AdFresca.RewardItem` 객체들이 포함되어 있으며 name, quantity, uniqueValue 프로퍼티 값을 가지고 있습니다.

- `GetAvailableRewardItems()` 메소드는 현재 지급이 가능한 아이템 리스트를 리턴하며, 아직 검사가 끝나지 않은 경우 이후 메소드 호출 시에 반영됩니다.

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

###(Advanced) 더욱 빠르게 아이템 지급하기:

`GetAvailableRewardItems()` 메소드는 현재 지급 가능한 아이템 리스트를 리턴한 이후, 새롭게 지급 가능한 아이템들이 있는지 백그라운드로 검사를 진행하게 됩니다. 만약 앱 시작시에 미리 검사를 수동으로 진행하고 원하는 위치에서 `GetAvailableRewardItems()` 메소드를 호출한다면, 사용자들에게 더욱 빠른 아이템 지급이 가능해집니다.

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

**Tip:** `CheckRewardItems(bool synchronized)` 메소드를 `synchronized=true` 로 실행하면, _Plugin_이 모든 검사를 완료할 때 까지 기다린 후 바로 아이템을 지급할 수도 있습니다.

* * *

## Advanced Features

### Test Device ID

_AD fresca_는 테스트 모드 기능을 지원하며 테스트에 사용할 수 있는 기기를 별도로 등록하여 관리할 수 있습니다.

테스트 기기 ID는 SDK를 통해 추출이 가능하며 2가지 방법을 지원 합니다.


1. testDeviceId 값을 직접 얻어와서 로그로 출력하는 방법

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
2. printTestDeviceId Property를 설정하여 화면에 Device ID를 표시하는 방법
 
```cs
AdFresca.Plugin plugin = AdFresca.Plugin.Instance
plugin.Init(API_KEY);
plugin.StartSession();
plugin.SetPrintTestDeviceId(true);
plugin.Load();
plugin.Show();
```

### Timeout Interval

컨텐츠의 최대 로딩 시간을 직접 지정하실 수 있습니다. 지정된 시간 내에 컨텐츠가 로딩되지 못한 경우, 사용자에게 컨텐츠를 노출하지 않습니다.

최소 1초 이상 지정이 가능하며, 지정하지 않을 시 기본 값으로 5초가 지정 됩니다.

```cs
AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
plugin.Init(API_KEY);
plugin.SetTimeoutInterval(5);
plugin.Load();
plugin.Show();
```

* * *

## Release Notes
- v2.1.6 _(1/10/2014 Updated)_ 
    - [Android SDK 2.3.2](https://github.com/adfresca/sdk-android-sample/blob/master/README.md#release-notes) 버전을 지원합니다.
    - Unity 4.3.x for Android 버전에서 ForwardNativeEventsToDalvik 옵션이 설정되지 않은 경우 터치 이벤트가 동작하지 않습니다. 이를 해결하기 위한 자세한 적용 방법은 [Installation](#installation) 항목을 참고하여 주세요.
- v2.1.5 _(12/01/2013 Updated)_ 
    - [iOS SDK 1.3.4](https://adfresca.zendesk.com/entries/21346861#release-notes) 버전을 지원합니다.
- v2.1.4 _(11/27/2013 Updated)_ 
    - [iOS SDK 1.3.3](https://adfresca.zendesk.com/entries/21346861#release-notes) 버전을 지원합니다.
    - [Android SDK 2.3.1](https://github.com/adfresca/sdk-android-sample/blob/master/README.md#release-notes) 버전을 지원합니다.
- v2.1.3 _(10/01/2013 Updated)_ 
    - [Android SDK 2.2.3](https://github.com/adfresca/sdk-android-sample/blob/master/README.md#release-notes) 버전을 지원합니다.
- v2.1.2 _(08/19/2013 Updated)_ 
    - [iOS SDK 1.3.2](https://adfresca.zendesk.com/entries/21346861#release-notes) 버전을 지원합니다.
- v2.1.1
    - [Android SDK 2.2.2](https://github.com/adfresca/sdk-android-sample/blob/master/README.md#release-notes) 버전을 지원합니다.
- v2.1.0 _(08/08/2013 Updated)_
    - [Android SDK 2.2.1](https://github.com/adfresca/sdk-android-sample/blob/master/README.md#release-notes) 버전을 지원합니다.
    - Android Platform 에서는 TestDeviceId() 메소드 대신 PrintTestDeviceIdByLog() 메소드를 사용하여 연결된 디바이스의 아이디를 확인하도록 변경 되었습니다.
- v2.0.1 _(07/26/2013 Updated)_
    - Plugin에 포함된 GCMIntentService 클래스를 이용하는 경우, 앱이 완전히 종료된 상황에서 푸시 메시지 수신 시 에러 메시지가 발생하는 버그를 수정하였습니다.
    - AndroidPlugin.cs 파일의 기본 매개변수 설정을 삭제하였습니다. 
    - 포함된 Android SDK를 2.1.3 버전으로 업데이트하였습니다.
- v2.0.0 _(07/10/2013 Updated)_
    - _Incentivized CPI_캠페인을 위한 API 가 추가되었습니다.
