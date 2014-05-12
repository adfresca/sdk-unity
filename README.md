## Contents
- [Introduction](#introduction)
- [Quick Start](#quick-start)
    - [Installation](#installation)
    - [Code](#code)
- [Test Device ID](#test-device-id)
- [Custom Parameter](#custom-parameter)
- [Marketing Event](#marketing-event)
- [In-App-Purchase Count](#in-app-purchase-count)
- [Push Notification](#push-notification)
- [Custom URL](#custom-url)
- [In-App Purchase Tracking (Beta)](#in-app-purchase-tracking-beta)
- [CPI Identifier](#cpi-identifier)
- [Reward Item](#reward-item)
- [Advanced Features](#advanced-features)
    - [Timeout Interval](#timeout-interval)
- [Proguard Configuration](#proguard-configuration)
- [Trouble Shooting](#trouble-shooting)
- [Release Notes](#release-notes)

* * *

## Introduction

AD fresca는 게임 운영자나 마케터가 앱 내 사용자 특성을 실시간으로 파악하여  더 자주, 더 오래 플레이하고, 더 많이 결제하도록 유도하는 라이브 서비스 운영 툴을 제공합니다.

게임 운영자나 마케터는 [Dashboard](https://admin.adfresca.com) 사이트를 통해 실시간으로 타겟팅한 사용자에게 메시지를 전달할 수 있으며, 이를 실제 앱에 적용하기 위하여 게임 개발팀에서는 아래 제공되는 SDK를 손쉽게 설치하고 가이드에 따라 코드를 적용합니다.

Unity 엔진의 경우 기존의 Native SDK를 연동하여 사용할 수 있도록 공식 플러그인을 제공합니다.  

Unity Package 파일을 통해 모든 구성요소를 쉽게 설치할 수 있으며, 아래 적용 가이드에 따라 간단한 코드만으로  Unity 환경에서도 SDK 적용이 가능하게 되었습니다.

플러그인 제작에 사용된 모든 Wrapper 코드가 공개되며, 필요한 경우 각 게임에 알맞게 수정하여 적용할 수 있습니다. 

* * *

## Quick Start

### Installation

아래 링크를 통해 _Unity Plugin_을 다운로드 합니다.

[Unity Plugin v2.1.8 다운로드](https://s3-ap-northeast-1.amazonaws.com/file.adfresca.com/distribution/sdk-for-Unity.zip) (Android SDK v2.3.4, iOS SDK v1.3.5)

[Unity Plugin with IAP Tracking BETA v2.2.0-beta3 다운로드](https://s3-ap-northeast-1.amazonaws.com/file.adfresca.com/distribution/sdk-for-Unity-iap-beta.zip) (Android SDK v2.4.0-beta4, iOS SDK v1.3.5)

Unity 프로젝트를 열고 AdFrescaUnityPlugin.package 파일을 실행합니다.

아래의 구성 요소들이 Assets 폴더 아래에 복사되어야 합니다.

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

이제 각 플랫폼에 맞게 설치 작업을 진행합니다.

### Android

Android 플랫폼의 대부분의 설치 및 적용 작업이 플러그인에 이미 구현되어 있습니다.기본적으로 AndroidManifest.xml 수정 작업만 진행하여 주시면 됩니다.

#### AndroidManifest.xml 수정하기

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android" android:installLocation="preferExternal" package="com.MyCompany.ProductName" android:versionName="1.0" android:versionCode="1">	
	<application android:icon="@drawable/app_icon" android:label="@string/app_name">
		<activity android:name="com.unity3d.player.UnityPlayerProxyActivity" android:label="@string/app_name">
		  <intent-filter>
		    <action android:name="android.intent.action.MAIN" />
		    <category android:name="android.intent.category.LAUNCHER" />
		  </intent-filter>
		<!-- Enable ForwardNativeEventsToDalvik -->
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
		<receiver android:name="com.MyCompany.ProductName.CustomGCMReceiver" android:permission="com.google.android.c2dm.permission.SEND" >
		  <intent-filter>
		    <action android:name="com.google.android.c2dm.intent.RECEIVE" />
		    <action android:name="com.google.android.c2dm.intent.REGISTRATION" />
		    <category android:name="com.MyCompany.ProductName" />
		  </intent-filter>
		</receiver>
		<service android:name="com.MyCompany.ProductName.GCMIntentService" />  <!-- GCM 메시지를 처리하기 위하여 GCMReceiver, GCMIntentService 클래스를 직접 구현해야 합니다. (Push Notification 항목 참고) -->    	
	</application>
	
	<uses-feature android:glEsVersion="0x00020000" />
	<uses-sdk android:minSdkVersion="6" android:targetSdkVersion="16" />
	
	<!-- Permission 추가 -->
 	<uses-permission android:name="android.permission.INTERNET"/>
 	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
	
 	<!-- Push Notification 기능을 사용하기 위하여 아래 내용을 추가 -->
 	<permission android:name="com.MyCompany.ProductName.permission.C2D_MESSAGE" android:protectionLevel="signature" />
	<uses-permission android:name="com.MyCompany.ProductName.permission.C2D_MESSAGE" />
	<uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
	<uses-permission android:name="android.permission.GET_ACCOUNTS" />
	<uses-permission android:name="android.permission.WAKE_LOCK" />
	<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
	<uses-permission android:name="android.permission.READ_PHONE_STATE" /> 
	<uses-permission android:name="android.permission.VIBRATE" />
</manifest>
```

위와 같이 SDK 적용에 필요한 permission과 service를 등록합니다. 'com.MyCompany.ProductName'로 표기된 패키지명은 모두 알맞은 값으로 수정합니다.
GCMReceiver, GCMIntentService 클래스의 구현은 아래의 [Push Notification](#push-notification) 항목에서 진행합니다.

### iOS

iOS의 경우는 Native SDK와 동일한 설치 작업을 진행합니다.  모든 플러그인 구성 요소가 Import 되었는지 확인 후, Unity에서 Xcode 프로젝트를 빌드합니다. 

우선, iOS  SDK 설치 가이드의 ['Installation'](https://github.com/adfresca/sdk-ios/edit/master/README.md#installation) 항목을 따라서 설치 작업을 진행합니다.

그리고 아래의 내용을 AppController.mm 파일에 적용합니다.

```objective-c
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

AD fresca SDK 통해 사용자에게 메시지를 전달하기 위한 주요 코드를 적용합니다. 아래의 코드만으로도 게임 운영자 / 마케터가 지정한 캠페인의 콘텐츠를 화면에 표시할 수 있습니다.

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
    plugin.Load();
    plugin.Show();
	}
}
```

`plugin.Init(API_KEY);` API Key 를 설정합니다. API Key는 [Dashboard](https://admin.adfresca.com) 사이트에서 앱 추가 후 Overview 메뉴의 Settings - API Keys 버튼을 클릭하여 확인이 가능합니다.

`plugin.StartSession();` 세션이 시작됨을 서버에 알립니다. 어플리케이션이 시작될 때 **한 번만** 실행되도록 합니다.

`plugin.Load();` 서버로부터 매칭되는 캠페인의 콘텐츠를 내려받습니다. 

`plugin.Show();` 내려받은 콘텐츠를 화면에 표시합니다.

앱이 실행 되면 다음과 같은 화면이 보여집니다. 정상적으로 콘텐츠 뷰가 화면에 표시되고, 터치 시 앱스토어 페이지로 이동하는지 확인합니다.

<img src="https://adfresca.zendesk.com/attachments/token/2fv9e76ptp7yo3h/?name=android-sample-p.png" width="240" />
&nbsp;
<img src="https://adfresca.zendesk.com/attachments/token/phn4fcpvbi2damx/?name=device-2013-03-18-133443.png" height="240" />

* * *

### Test Device ID

AD fresca는 테스트 모드 기능을 지원하여 테스트를 원하는 디바이스에만 지정한 캠페인의 콘텐츠를 화면에 표시하고 푸시 메시지를 전송할 수 있습니다. 이로 인해 SDK가 적용된 앱이 이미 앱스토어에 출시된 경우, 게임 운영팀 혹은 개발팀에게만 새로운 메시지를 전달할 수 있도록 지원합니다.

테스트 기기 등록을 위한 아이디 값은 SDK를 통해 추출이 가능하며 2가지 방법을 지원 합니다.

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

2. SetPrintTestDeviceId() 메소드를 사용하여 화면에 Device ID를 표시하는 방법
 
```cs
AdFresca.Plugin plugin = AdFresca.Plugin.Instance
plugin.Init(API_KEY);
plugin.StartSession();
plugin.SetPrintTestDeviceId(true);
plugin.Load();
plugin.Show();
```

* * *

## Custom Parameter

커스텀 파라미터는 캠페인 진행 시, 타겟팅을 위해 사용할 사용자의 상태 값을 의미합니다.

AD fresca SDK는 기본적으로 '국가, 언어, 앱 버전, 실행 횟수 등'의 디바이스 고유 데이터를 수집하며, 동시에 각 앱 내에서 고유하게 사용되는 특수한 상태 값들(예: 캐릭터 레벨, 보유 포인트, 스테이지 등)을 커스텀 파라미터로 정의하고 수집하여 분석 및 타겟팅 기능을 제공합니다.

커스텀 파라미터 설정은 [Dashboard](https://admin.adfresca.com) 사이트를 접속하여 앱의 Overview 메뉴 -> Settings - Custom Parameters 버튼을 클릭하여 확인할 수 있습니다.

SDK 적용을 위해서는 Dashboard에서 지정된 각 커스텀 파라미터의 '인덱스' 값이 필요합니다. 인덱스 값은 1,2,3,4 와 같은 Integer 형태의 고유 값이며 소스코드에 Constant 형태로 지정하여 이용하는 것을 권장합니다.

Integer, Boolean 형태의 데이터를 상태 값으로 설정할 수 있으며, *SetCustomParameter** 메소드를 사용하여 각 인덱스 값에 맞게 상태 값을 설정합니다.

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

만약 불가피하게 StartSession() 호출 시에 커스텀 파라미터 값을 설정할 수 없는 경우, 앱을 최초로 실행한 사용자의 프로파일은 업데이트되지 않으며 해당 사용자의 2회째 앱 실행부터 SDK가 로컬에 캐싱해둔 값이 전달됩니다. 최초로 실행된 사용자의 프로파일까지 통계 및 타겟팅하기 위해서는 아래와 같이 초기 값 설정을 진행합니다. 또한, 사용자의 로그인 이벤트 이후 모든 커스텀 파라미터의 값을 설정할 수 있도록 구현합니다.

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

## Marketing Event

마케팅 이벤트는 유저에게 메세지를 전달하고자 하는 상황을 의미합니다. (예: 캐릭터 레벨 업, 퀘스트 달성, 스토어 페이지 진입)

마케팅 이벤트 기능을 사용하여 지정된 상황에 알맞는 캠페인이 노출되도록 할 수 있습니다.

마케팅 이벤트 설정은 [Dashboard](https://admin.adfresca.com) 사이트를 접속하여 앱의 Overview 메뉴 -> Settings - Marketing Events 버튼을 클릭하여 확인할 수 있습니다.

SDK 적용을 위해서는 Dashboard에서 지정된 각 마케팅 이벤트의 '인덱스' 값이 필요합니다. 인덱스 값은 1,2,3,4 와 같은 Integer 형태의 고유 값이며 소스코드에 Constant 형태로 지정하여 이용하는 것을 권장합니다.

각 이벤트 발생 시, Load() 메소드에 원하는 이벤트 인덱스 값을 인자로 넘겨주시면 간단히 적용이 완료됩니다.

(Load() 메소드에 인덱스를 설정하지 않은 경우, 인덱스 값은 '1' 값이 자동으로 지정됩니다.)

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

## In-App Purchase Count

앱에서 IAP 기능을 사용하는 경우, 현재까지 사용자가 구매한 누적 횟수를 SDK에 설정하여 분석 및 타겟팅에 이용할 수 있습니다. 

**setNumberOfInAppPurchases(int)** 메소드를 사용하여 현재까지 사용자가 구매한 누적 횟수 값을 SDK에 설정합니다. 커스텀 파라미터와 마찬가지로 앱 실행 혹은 사용자 로그인 이후에 값을 지정하고, IAP 결제가 일어난 직후에 갱신된 누적 구매 횟수 값을 설정합니다.

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
}
```

* * *

## Push Notification

AD fresca를 통해 Push Notification을 보내고 받을 수 있습니다.

#### Android

SDK를 적용하기 이전에 [Google API Console](https://cloud.google.com/console) 사이트에서 프로젝트를 생성하고, [Dashboard](https://admin.adfresca.com) 사이트에 설정할 GCM API Key 및 SDK 적용에 필요한 GCM_SENDER_ID (Project Number) 값을 얻어야 합니다.

'[Android Push Notification 설정 및 적용하기 (GCM)](https://adfresca.zendesk.com/entries/28526764)' 가이드를 참고하여 필요한 값들을 얻습니다.

이제 SDK 적용을 시작합니다.

1) AndroidManifest.xml 확인하기

```xml
<manifest>   
  <application>
      .........
      <activity android:name="com.adfresca.ads.AdFrescaPushActivity" />
      <receiver android:name="com.MyCompany.ProductName.CustomGCMReceiver"
        android:permission="com.google.android.c2dm.permission.SEND">  
        <intent-filter>
          <action android:name="com.google.android.c2dm.intent.RECEIVE" />
          <action android:name="com.google.android.c2dm.intent.REGISTRATION" />
          <category android:name="com.MyCompany.ProductName" />
         </intent-filter>
      </receiver>
      <service android:name="com.MyCompany.ProductName.GCMIntentService" />  
   </application>
    ..........
    <permission android:name="com.MyCompany.ProductName.permission.C2D_MESSAGE" android:protectionLevel="signature" />
    <uses-permission android:name="com.MyCompany.ProductName.permission.C2D_MESSAGE" />
    <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
    <uses-permission android:name="android.permission.GET_ACCOUNTS" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
    <uses-permission android:name="android.permission.READ_PHONE_STATE" /> 
    <uses-permission android:name="android.permission.VIBRATE" />
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
            	targetActivityClass = UnityPlayerProxyActivity.class; // or YourUnityPlayerProxyActivity.class
            }
            
            AdFresca.showNotification(context, intent, targetActivityClass, title, icon, when);
        }                
    }
    
    @Override
    protected void onError(Context context, String registrationId) {
    
    }
  }
```

showNotification() 메소드는 가장 기본적인 Notification 뷰를 이용하여 아무런 사운드 없이 메시지를 표시합니다. Notification에 알람이나 사운드 파일 설정 같은 커스터마이징 작업이 필요한 경우 Android SDK 가이드의 ["Custom Notification"](https://github.com/adfresca/sdk-android-sample/blob/master/README.md#custom-notification) 내용을 참고하여 주시기 바랍니다.

또한, Image Push Notification 기능 적용에 대한 내용은 Android SDK 가이드의 ["Image Notification"](https://github.com/adfresca/sdk-android-sample/blob/master/README.md#image-notification) 내용을 참고하여 주시기 바랍니다.

3) GCMReceiver 클래스 구현하기

```java
public class GCMReceiver extends GCMBroadcastReceiver { 
   	@Override
	protected String getGCMIntentServiceClassName(Context context) { 
		return "com.MyCompany.ProductName.GCMIntentService"; 
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

1) Push Notification 인증서 파일을 생성하고 Dashboard 사이트에 등록합니다.
  - [iOS Push Notification 인증서 설정 및 적용하기](https://adfresca.zendesk.com/entries/21714780) 가이드 를 따라 Production용 Push Notification Certificate를 생성하고 [Dashboard](https://admin.adfresca.com) 사이트에 등록합니다.

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
	<activity android:name="com.MyCompany.ProductName.MainActivity" android:label="@string/app_name">
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


## In-App Purchase Tracking (Beta)

_**(현재 In-App-Purchase Tracking 기능은 Unity Plugin 2.2.0-beta1 버전, Android OS에서만 지원됩니다.)**_

_In-App-Purchase Tracking_  기능을 통하여 현재 앱에서 발생하고 있는 모든 인-앱 결제를 분석하고 캠페인 타겟팅에 이용할 수 있습니다.

AD fresca의 In-App-Purchase Tracking은 2가지 유형이 있습니다.

1. 실제 화폐를 통해 결제되는 Actual Item Purchase Tracking (예: USD $1.99를 결제하여 Gold 100개 아이템을 구입)
2. 가상 화폐를 통해 결제되는 Virtual Item Purchase Tracking (예: Gold 10개를 이용하여 포션 아이템을 구입)

위 2가지 유형의 데이터를 모두 Tracking 함으로써 앱의 매출뿐만 아니라 인-앱 사용자들의 아이템 구매 추이 분석까지 가능합니다.

아이템 정보 등록을 위한 별도의 작업은 필요하지 않으며, 클라이언트에서 결제된 아이템 정보가 자동으로 대쉬보드에 등록되는 방식입니다. (아이템 리스트 확인은 대쉬보드 'Overview' 메뉴의 Settings - In App Items 페이지를 통해 확인할 수 있습니다.)

아래의 적용 예제를 참고하여 간단히 In-App-Purchase Tracking 기능을 적용합니다.

### Actual Item Tracking

Actual Item의 결제는 각 앱스토어별 인-앱 결제 라이브러리를 통해 이루어집니다. 각 결제 라이브러리에서 _'결제 성공'_ 이벤트가 발생 할 시에 Purchase 객체를 생성하고 LogPurchase(purchase) 메소드를 호출합니다.

적용 예제 1: 유니티 환경에서 결제 성공 이벤트 발생 시
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

적용 예제 2: Android 네이티브 환경에서 Google Play 결제 모듈을 직접 구현하는 경우
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

위 예제는 Google Play 결제 라이브러리를 기준으로 작성되었지만 아마존이나 티스토어 등 모든 결제 라이브러리에서도 Purchase 객체에 필요한 값을 얻어올 수 있습니다.

Actual Item을 위한 PurchaseBuilder의 보다 자세한 설명은 아래와 같습니다.

Method | Description
------------ | ------------- | ------------
WithItemId(string) | 결제한 아이템의 고유 식별 아이디를 설정합니다. 등록된 앱스토어에 상관 없이 앱내에서 고유한 식별 값을 이용하는 것을 권장합니다. AD fresca 대쉬보드에서 해당 값을 기준으로 아이템 목록이 생성됩니다. 
WithCurrencyCode(string) | ISO 4217 표준 코드를 설정합니다. Google Play의 경우 'Default price' 에 설정되는 Currency Code 값을 이용하며 타 결제 라이브러리의 경우는 보통 이용 가능한 Currency Code가 고정되어 있습니다 (예: 아마존은 USD, 티스토어는 KRW). 또는 자체 백엔드 서버에서 결제하는 아이템의 Currency Code를 내려받아 설정할 수 있습니다.
WithPrice(double) | 아이템의 가격을 설정합니다. 결제 라이브러리에서 주는 값을 이용하거나, 자체 백엔드 서버에서 가격을 내려받아 설정할 수 있습니다. 
WithPurchaseDate(datetime) | 결제된 시간을 DateTime 객체 형태로 설정합니다. 값이 설정되지 않은 경우 AD fresca 서비스에 기록되는 시간이 결제 시간으로 자동 설정됩니다.
WithReceipt(string, string, string) | 추후 Receipt Verficiation 기능을 위해 필요한 데이터를 설정합니다. 현재 버전의 SDK는 Google Play만 지원하며 타 결제 라이브러리의 경우는 값을 설정하지 않습니다.

### Virtual Item Tracking

Virtual Item의 결제는 앱 내의 가상 화폐로 아이템을 결제한 경우를 의미합니다. 앱 내에서 가상 화폐를 이용한 결제 이벤트가 성공한 경우 아래 예제와 같이 Purchase 객체를 생성하고 LogPurchase(purchase) 메소드를 호출합니다.

적용 예제: 
```cs
AdFresca.Purchase purchase = new AdFresca.PurchaseBuilder(AdFresca.Purchase.Type.VIRTUAL_ITEM)
	.WithItemId("long_sword")
	.WithCurrencyCode("gold") 
	.WithPrice(100)
	.WithPurchaseDate(purchaseDateTime);

AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
plugin.LogPurchase(purchase);
```

Virtual Item을 위한 PurchaseBuilder의 보다 자세한 설명은 아래와 같습니다.

Method | Description
------------ | ------------- | ------------
WithItemId(string) | 결제한 아이템의 고유 식별 아이디를 설정합니다. 등록된 앱스토어에 상관 없이 앱내에서 고유한 식별 값을 이용하는 것을 권장합니다. AD fresca 대쉬보드에서 해당 값을 기준으로 아이템 목록이 생성됩니다. 
WithCurrencyCode(string) | 결제에 사용한 가상화폐 고유 코드를 설정합니다. (예: gold)
WithPrice(double) | 가상 화폐로 결제한 가격 정보를 설정합니다. (예: gold 10개의 경우 10 값을 설정)
WithPurchaseDate(datetime) | 결제된 시간을 DateTime 객체 형태로 설정합니다. 값이 설정되지 않은 경우 AD fresca 서비스에 기록되는 시간이 결제 시간으로 자동 설정됩니다.

### IAP Trouble Shooting

LogPurchase() 메소드를 통해 기록된 Purchase 객체는 AD fresca 서비스에 업데이트되어 실시간으로 대쉬보드에 반영됩니다. 현재까지 등록된 아이템 리스트는 'Overview' 메뉴의 Settings - In App Items 페이지를 통해 확인할 수 있습니다.

만약 아이템 리스트가 새로 갱신되지 않는 경우, Android SDK의 AFPurchaseExceptionListener 구현하여 혹시 에러가 발생하고 있지 않은지 확인해야 합니다. Purchase 객체의 값이 제대로 설정되지 않은 경우, AFPurchaseExceptionListener 통하여 에러 메시지가 표시됩니다.

Unity Plugin의 경우는 이미 아래와 같은 코드가 적용되어 있습니다. 따라서 Purchase 객체가 제대로 생성되지 않은 경우 "AFPurchaseExceptionListener.onException = {error message}" 형태의 로그가 콘솔에 출력됩니다.

```java
......
AdFresca.getInstance(UnityPlayer.currentActivity).logPurchase(purchase, new AFPurchaseExceptionListener(){
	public void onException(AFPurchase purchase, AFException e) {
		Log.e("AdFresca", "AFPurchaseExceptionListener.onException = " + e.getMessage());
	}
});
```

* * *

## CPI Identifier

Incentivized CPI & CPA 캠페인 기능을 사용하여, 사용자가 Media App에서 Advertising App의 광고를 보고 앱을 설치하였을 때 보상으로 Media App의 아이템을 지급할 수 있습니다.

- Medial App: 다른 앱의 광고를 노출하고, 광고 대상의 앱을 설치한 사용자들에게 보상을 지급하는 앱
- Advertising: Media App에 광고가 노출되는 앱.

Incentivized CPI & CPA 캠페인에 대한 보다 자세한 설명 및 [Dashboard](https://admin.adfresca.com) 사이트에서의 설정 방법은 [크로스 프로모션 캠페인 이해하기](https://adfresca.zendesk.com/entries/22033960) 가이드를 참고하여 주시기 바랍니다.

SDK 적용을 위해서는 Advertising App에서의 URL Schema 설정 및 Media App에서의 Reward Item 지급 기능을 구현해야 합니다.

(현재 Incentivized CPI 캠페인을 진행할 경우, Advertising App의 SDK 설치는 필수가 아니며 URL Schema 설정만 진행되면 됩니다. 하지만 Incentivized CPA 캠페인을 진행할 경우 반드시 SDK 설치 및 [Marketing Event](#marketing-event) 기능이 적용되어야 합니다.)
  
#### Advertising App 설정하기:
  1. Android

  Android 플랫폼의 경우 앱의 패키지 이름을 이용하여 광고를 노출한 앱이 실제로 디바이스에 설치되었는지 검사하게 됩니다. 따라서 Advertising App 앱의 패키지 이름을 확인하고 CPI Identifier로 사용합니다.

  AndroidManifest.xml 파일을 열어 패키지 이름을 확인합니다.

  ```xml
  <?xml version="1.0" encoding="utf-8"?>

  <manifest xmlns:android="http://schemas.android.com/apk/res/android" package="com.adfresca.demo">
    <application>
    .....
    </application>
  </manifest>
  ```

  위 경우 [Dashboard](https://admin.adfresca.com) 사이트에서 Advertising App의 CPI Identifier 값을 'com.adfresca.demo' 으로 설정하게 됩니다. 

  2. iOS

  iOS 플랫폼의 경우 URL Schema 값을 이용하여 광고를 노출한 앱이 실제로 디바이스에 설치되었는지 검사하게 됩니다. 따라서 Advertising App 앱의 URL Schema을 설정하고 CPI Identifier로 사용합니다.

  Xcode 프로젝트의 Info.plst 파일을 열어 사용할 URL Schema 정보를 설정 합니다.

  <img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png"/>

  위 경우 [Dashboard](https://admin.adfresca.com) 사이트에서 Advertising App의 CPI Identifier 값을 'myapp://' 으로 설정하게 됩니다. 
  iOS 플랫폼의 경우 URL Schema 값이 다른 앱과 중복될 수 있습니다. 정상적인 캠페인 진행을 위해서는 최대한 Unique한 값을 선택해야 합니다.
  
  마지막으로, Incentivized CPA 캠페인을 진행할 경우는 보상 조건으로 지정한 마케팅 이벤트가 발생되어야 합니다. 사용자가 보상 조건을 완료한 이후 아래와 같이 유니티 코드로 지정한 마케팅 이벤트를 호출합니다.
  ```cs
  // 튜토리얼 완료 이벤트를 보상 조건으로 지정한 경우
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.Load(EVENT_INDEX_TUTORIAL);  
  plugin.Show(EVENT_INDEX_TUTORIAL);
  ```

#### Media App SDK 적용하기:

  Media App에서 보상 지급 여부를 확인하고, 사용자에게 아이템을 지급하기 위해서는 SDK 가이드의 [Reward Item](#reward-item) 항목의 내용을 구현합니다.

* * *

## Reward Item

Reward Item 기능을 적용하여 현재 사용자에게 지급 가능한 보상 아이템이 있는지 확인하고, 지정된 보상 아이템을 사용자에게 지급할 수 있습니다.

Annoucnement 캠페인의 'Reward Item' 항목을 설정했거나, Incentivized CPI & CPA 캠페인의 'Incentive Item' 을 설정한 경우 사용자에게 보상 아이템이 지급됩니다.

먼저 AndroidManifest.xml 내용을 확인합니다.

```xml
<manifest>   
  <application>
      .........
      <activity android:name="com.adfresca.sdk.reward.AFRewardActivity" />
      .........
   </application>
</manifest>
```

이제 구현을 위해서 아래 코드를 사용합니다.
- CheckRewardItems(): 현재 지급 가능한 보상 아이템이 있는지 검사합니다. 사용자가 앱을 실행할 호출하는 것을 권장합니다.
- SetAndroidRewardItemListener(): 아이템 지급 조건이 만족되면 onReward 이벤트가 발생하도록 설정할 수 있습니다. 이벤트를 발생시킬 게임 오브젝트와 이벤트 메소드 이름을 지정합니다. Android에 한해서 제공되며 iOS의 경우 아래와 같이 AFRewardItemDelegate를 네이티브 상에서 구현하여 유니티로 이벤트를 넘겨줍니다.

**iOS에서 AFRewardItemDelegate 구현하기**

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
  [AdFrescaView startSession:@"YOUR_API_KEY"];
  [[AdFrescaView shardAdView] setRewardDelegate:self];
}

- (void)itemRewarded:(AFRewardItem *)item 
{
  UnitySendMessage("YourGameObject", "OnReward", [[item JSON] UTF8String]);
}

```

**Unity 코드 적용하기**

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

캠페인 종류에 따라 onReward 이벤트의 발생 조건이 다릅니다.

- Annoucnement 캠페인: 캠페인이 앱 사용자에게 매칭되어 노출될 때 이벤트가 발생합니다
- Incentivized CPI 캠페인: 사용자의 Advertising App 설치가 확인된 후 이벤트가 발생합니다.
- Incentivized CPA 캠페인: 사용자의 Advertising App 설치가 확인되고 보상 조건으로 지정된 마케팅 이벤트가 호출된 후에 발생합니다.

만일 디바이스의 네트워크 단절이 발생한 경우 SDK는 데이터를 로컬에 보관하여 다음 앱 실행에서 아이템 지급이 가능하도록 구현되어 있기 때문에 항상 100% 지급을 보장합니다.

(기존의 GetAvailableRewardItems 메소드는 Deprecated 상태로 변경되었지만, 호환성을 보장하여 정상적으로 동작하고 있습니다.)

* * *

## Advanced Features

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

## Proguard Configuration

안드로이드에서 Proguard 툴을 이용하여 APK 파일을 보호하는 경우 몇 가지 예외 처리 작업을 진행해야 합니다. AD fresca SDK와 SDK에 포함된 OpenUDID 및 Google Gson에 대한 예외 처리를 아래와 같이 적용합니다.

```java
-keep class com.adfresca.** {*;} 
-keep class com.google.gson.** {*;} 
-keep class org.openudid.** {*;} 
-keep class sun.misc.Unsafe { *; }
-keepattributes Signature 
```

* * *

## Trouble Shooting

콘텐츠가 화면에 제대로 출력되지 않거나, 에러가 발생하는 경우 SDK에서 에러 정보를 확인할 수 있습니다. 현재 Unity 코드로 에러 정보를 출력하는 방법은 아직 지원되지 않고 있으며, 각 플랫폼 코드에서 직접 로그를 출력할 수 있습니다.

**Android**

UnityPlayerActivity 클래스를 오버라이드해서 사용하고 있거나, 다른 자바 플러그인을 이용하는 있는 경우 아래 코드를 적용하여 로그를 출력할 수 있습니다. (혹은 UnitySendMessage 메소드를 이용하여 유니티로 이벤트를 전달할 수 있습니다.)

```java
AdFresca.setExceptionListener(new AFExceptionListener(){
  @Override
  public void onExceptionCaught(AFException e) {
    Log.w("TAG", e.getCode() + ":" + e.getLocalizedMessage());
  }
});
```

**iOS**

Xcode 프로젝트에서 AdFrescaViewDelegate를 구현하여 로그를 출력할 수 있습니다. (혹은 UnitySendMessage 메소드를 이용하여 유니티로 이벤트를 전달할 수 있습니다.)

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
  [AdFrescaView startSession:@"YOUR_API_KEY"];
  [[AdFrescaView shardAdView] setDelegate:self];
}

- (void)fresca:(AdFrescaView *)fresca didFailToReceiveAdWithException:(AdException *)error {  
  NSLog(@"AdException message : %@", [error message]);
}
```

* * *

## Release Notes
- **v2.2.0-beta3 _(4/6/2014 Updated)_** 
    - iOS SDK 설치 과정에서 AdSupport framework 추가가 필수항목에서 제외됩니다. IFA 수집을 하지 않아도 SDK 이용이 가능하도록 수정되었습니다. 보다 자세한 내용은 [iOS SDK - Installation](https://github.com/adfresca/sdk-ios/edit/master/README.md#installation) 항목을 참고하여 주세요.
    - v2.1.8에서 적용된 'Announcement 캠페인을 통한 Reward Item 지급 기능'을 지원합니다.
    - v2.1.8에서 적용된 Incentivized CPA 캠페인 기능을 지원합니다. 자세한 내용은 [CPI Identifier](#cpi-identifier) 항목을 참고하여 주세요
    - v2.1.8에서 개선된 [Reward Item](#reward-item) 기능이 적용되었습니다. 
    - [Android SDK 2.4.0-beta4](https://github.com/adfresca/sdk-android-sample/blob/master/README.md#release-notes) 버전을 지원합니다.
    - [iOS SDK 1.3.5](https://github.com/adfresca/sdk-ios/edit/master/README.md#release-notes) 버전을 지원합니다.
- v2.2.0-beta2 _(1/31/2014 Updated)_ 
    - [Android SDK 2.4.0-beta3](https://github.com/adfresca/sdk-android-sample/blob/master/README.md#release-notes) 버전을 지원합니다.
- v2.2.0-beta1 _(1/14/2014 Updated)_ 
    - 앱 내에서 발생하는 In-App Purchase 데이터를 트랙킹할 수 있는 기능이 추가되었습니다. 자세한 내용은 [In-App Purchase Tracking (Beta)](#in-app-purchase-tracking-beta) 항목을 참고하여 주세요.
    - [Android SDK 2.4.0-beta2](https://github.com/adfresca/sdk-android-sample/blob/master/README.md#release-notes) 버전을 지원합니다.
- **v2.1.8 _(4/6/2014 Updated)_** 
   - iOS SDK 설치 과정에서 AdSupport framework 추가가 필수항목에서 제외됩니다. IFA 수집을 하지 않아도 SDK 이용이 가능하도록 수정되었습니다. 보다 자세한 내용은 [iOS SDK - Installation](https://github.com/adfresca/sdk-ios/edit/master/README.md#installation) 항목을 참고하여 주세요.
   - Announcement 캠페인을 통한 Reward Item 지급 기능을 지원합니다.
   - Incentivized CPA 캠페인 기능을 지원합니다. 자세한 내용은 [CPI Identifier](#cpi-identifier) 항목을 참고하여 주세요
   - SetAndroidRewardItemListener 구현 기능이 추가되어, 지급 가능한 아이템이 발생할 시에 자동으로 이벤트가 발생합니다. 보다 자세한 내용은 [Reward Item](#reward-item) 항목을 참고하여 주세요.
    - [Android SDK 2.3.4](https://github.com/adfresca/sdk-android-sample/blob/master/README.md#release-notes) 버전을 지원합니다.
    - [iOS SDK 1.3.5](https://github.com/adfresca/sdk-ios/edit/master/README.md#release-notes) 버전을 지원합니다.
- v2.1.7 _(1/31/2014 Updated)_
    - [Android SDK 2.3.3](https://github.com/adfresca/sdk-android-sample/blob/master/README.md#release-notes) 버전을 지원합니다.
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
