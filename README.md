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

[AD fresca Unity Plugin v2.1.0  다운로드](https://s3-ap-northeast-1.amazonaws.com/file.adfresca.com/distribution/sdk-for-Unity.zip) (Android SDK v2.2.1, iOS SDK v1.3.0)

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
		</activity>
		
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
		
		<!-- Push Notification 기능을 사용할 경우, 아래 내용을 추가합니다. -->
		<activity android:name="com.adfresca.ads.AdFrescaPushActivity" />
		<receiver android:name="com.Company.ProductName.CustomGCMReceiver" android:permission="com.google.android.c2dm.permission.SEND" >
		  <intent-filter>
		    <action android:name="com.google.android.c2dm.intent.RECEIVE" />
		    <action android:name="com.google.android.c2dm.intent.REGISTRATION" />
		    <category android:name="com.Company.ProductName" />
		  </intent-filter>
		</receiver>
		<service android:name="com.Company.ProductName.CustomGCMIntentService" />  <!-- GCM 메시지를 처리하기 위하여 CustomGCMReceiver, CustomGCMIntentService 클래스를 구현해야 합니다.  -->    	
	</application>
	
	<uses-feature android:glEsVersion="0x00020000" />
	<uses-sdk android:minSdkVersion="6" android:targetSdkVersion="16" />
	
 	<uses-permission android:name="android.permission.INTERNET"/>
 	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
	
 	<!-- Push Notification 기능을 사용할 경우, 아래 내용을 추가합니다. -->
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

주의: SetNumberOfInAppPurchases() 메소드는 StartSession(), Load() 메소드 이전에 호출이 되어야 합니다.

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
_주의:_ SetCustomParameter() 메소드는 StartSession(), Load() 메소드 이전에 호출이 되어야 합니다. 특히 startSession() 이전에는 반드시 모든 커스텀 파리미터 값들을 설정하고, 이후 변경되는 값들에 한하여 각 위치에 커스텀 파라미터를 설정합니다.

만약 불가피하게 StartSession() 호출 시에 커스텀 파라미터 값을 설정할 수 없는 경우, 최초 실행한 사용자의 프로파일은 업데이트되지 않으며 해당 사용자의 2회째 앱 실행부터 SDK가 로컬에 캐싱해둔 값이 전달됩니다. 최초로 실행된 사용자의 프로파일까지 통계 및 타겟팅하기 위해서는 아래와 같이 초기 값 설정을 진행해야 합니다.
(SDK의 로컬 캐싱 기능은 Android SDK 2.2.0, iOS SDK 1.3.1 버전부터 지원합니다.)

```cs
void Start() {
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.Init(API_KEY);
  if (isFirstRun)
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

Push Notification 은 플랫폼 별로 세팅해야 하는 사항들이 있습니다. 자세한 사항은 각 플랫폼 별 가이드([Android](https://github.com/adfresca/sdk-android-sample#push-notification), [iOS](https://adfresca.zendesk.com/entries/21346861/#push-notification))를 참고해 주시기 바랍니다.

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
- v2.1.0 _(08/08/2013 Updated)_
    - [Android SDK 2.2.1](https://github.com/adfresca/sdk-android-sample/blob/master/README.md#release-notes) 버전을 지원합니다.
    - Android Platform 에서는 TestDeviceId() 메소드 대신 PrintTestDeviceIdByLog() 메소드를 사용하여 연결된 디바이스의 아이디를 확인하도록 변경 되었습니다.
- v2.0.1 _(07/26/2013 Updated)_
    - Plugin에 포함된 GCMIntentService 클래스를 이용하는 경우, 앱이 완전히 종료된 상황에서 푸시 메시지 수신 시 에러 메시지가 발생하는 버그를 수정하였습니다.
    - AndroidPlugin.cs 파일의 기본 매개변수 설정을 삭제하였습니다. 
    - 포함된 Android SDK를 2.1.3 버전으로 업데이트하였습니다.
- v2.0.0 _(07/10/2013 Updated)_
    - _Incentivized CPI_캠페인을 위한 API 가 추가되었습니다.
