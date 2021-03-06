# React Native Appsflyer Plugin Guides

<img src="https://massets.appsflyer.com/wp-content/uploads/2016/06/26122512/banner-img-ziv.png"  width="150">

## Table of content

- [Init SDK](#init-sdk)
- [Deep Linking](#deeplinking)
    - [Deferred Deep Linking (Get Conversion Data)](#deferred-deep-linking)
    - [Direct Deep Linking](#direct-deep-linking)
    - [iOS Deeplink Setup](#iosdeeplinks)
- [Uninstall](#track-app-uninstalls)
    - [iOS Uninstall Setup](#track-app-uninstalls-ios)
    - [Android Uninstall Setup](#track-app-uninstalls-android)


##  <a id="init-sdk"> Init SDK
  *Example:*

```javascript
appsFlyer.initSdk(
  {
    devKey: 'K2***********99',
    isDebug: false,
    appId: '41*****44',
  },
  (result) => {
    console.log(result);
  },
  (error) => {
    console.error(error);
  }
);
```

*With Promise:*

```javascript
try {
  var result = await appsFlyer.initSdk(options);
} catch (error) {}
```


##  <a id="deeplinking"> Deep Linking
    
<img src="https://massets.appsflyer.com/wp-content/uploads/2018/03/21101417/app-installed-Recovered.png"  width="300">



#### The 2 Deep Linking Types:
Since users may or may not have the mobile app installed, there are 2 types of deep linking:

1. Deferred Deep Linking - Serving personalized content to new or former users, directly after the installation. 
2. Direct Deep Linking - Directly serving personalized content to existing users, which already have the mobile app installed.

For more info please check out the [OneLink™ Deep Linking Guide](https://support.appsflyer.com/hc/en-us/articles/208874366-OneLink-Deep-Linking-Guide#Intro).

###  <a id="deferred-deep-linking"> 1. Deferred Deep Linking (Get Conversion Data)

Check out the deferred deep linking guide from the AppFlyer knowledge base [here](https://support.appsflyer.com/hc/en-us/articles/207032096-Accessing-AppsFlyer-Attribution-Conversion-Data-from-the-SDK-Deferred-Deeplinking-#Introduction).

Code Sample to handle the conversion data:


```javascript
this.onInstallConversionDataCanceller = appsFlyer.onInstallConversionData(
  (res) => {
    if (JSON.parse(res.data.is_first_launch) == true) {
      if (res.data.af_status === 'Non-organic') {
        var media_source = res.data.media_source;
        var campaign = res.data.campaign;
        alert('This is first launch and a Non-Organic install. Media source: ' + media_source + ' Campaign: ' + campaign);
      } else if (res.data.af_status === 'Organic') {
        alert('This is first launch and a Organic Install');
      }
    } else {
      alert('This is not first launch');
    }
  }
);

appsFlyer.initSdk(/*...*/);
```
**Note:** The code implementation for `onInstallConversionData` must be made **prior to the initialization** code of the SDK.

###  <a id="direct-deep-linking"> 2. Direct Deep Linking
    
When a deep link is clicked on the device the AppsFlyer SDK will return the link in the [onAppOpenAttribution](https://support.appsflyer.com/hc/en-us/articles/208874366-OneLink-Deep-Linking-Guide#deep-linking-data-the-onappopenattribution-method-) method.

```javascript
this.onAppOpenAttributionCanceller = appsFlyer.onAppOpenAttribution((res) => {
  console.log(res);
});

appsFlyer.initSdk(/*...*/);

```
**Note:** The code implementation for `onAppOpenAttribution` must be made **prior to the initialization** code of the SDK.

<hr/>

**Important**

The `appsFlyer.onInstallConversionData` returns function to  unregister this event listener. Actually it calls `NativeAppEventEmitter.remove()`. It is **required** on iOS to call the `onInstallConversionDataCanceller` and `onAppOpenAttributionCanceller`.

*Example:*

```javascript
  state = {
    appState: AppState.currentState,
  };

  componentDidMount() {
    AppState.addEventListener('change', this._handleAppStateChange);
  }

  componentWillUnmount() {
    AppState.removeEventListener('change', this._handleAppStateChange);
  }

  _handleAppStateChange = (nextAppState) => {
    if (this.state.appState.match(/active|foreground/) && nextAppState === 'background') {
      if (this.onInstallConversionDataCanceller) {
        this.onInstallConversionDataCanceller();
        console.log('unregister onInstallConversionDataCanceller');
      }
      if (this.onAppOpenAttributionCanceller) {
        this.onAppOpenAttributionCanceller();
        console.log('unregister onAppOpenAttributionCanceller');
      }
    }

    if (this.state.appState.match(/inactive|background/) && nextAppState === 'active') {
      if (Platform.OS === 'ios') {
        appsFlyer.trackAppLaunch();
      }
    }

    this.setState({appState: nextAppState});
  };
```






### <a id="iosdeeplinks"> iOS Deep Links - Universal Links and URL Schemes

In order to track retargeting and use the onAppOpenAttribution callbacks in iOS,  the developer needs to pass the User Activity / URL to our SDK, via the following methods in the **AppDelegate.m** file:

#### Universal Links (iOS 9 +)
```objectivec
- (BOOL)application:(UIApplication *)application continueUserActivity:(nonnull NSUserActivity *)userActivity restorationHandler:(nonnull void (^)(NSArray<id<UIUserActivityRestoring>> * _Nullable))restorationHandler {
  [[AppsFlyerTracker sharedTracker] continueUserActivity:userActivity restorationHandler:restorationHandler];
  return YES;
}
```

#### URL Schemes
```objectivec
// Reports app open from deep link from apps which do not support Universal Links (Twitter) and for iOS8 and below
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString*)sourceApplication annotation:(id)annotation {
[[AppsFlyerTracker sharedTracker] handleOpenURL:url sourceApplication:sourceApplication withAnnotation:annotation];
return YES;
}

// Reports app open from URL Scheme deep link for iOS 10
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url
options:(NSDictionary *) options {
[[AppsFlyerTracker sharedTracker] handleOpenUrl:url options:options];
return YES;
}
```
---



### <a id="track-app-uninstalls"> Track App Uninstalls

#### <a id="track-app-uninstalls-ios"> iOS

AppsFlyer enables you to track app uninstalls. To handle notifications it requires  to modify your `AppDelegate.m`. Use [didRegisterForRemoteNotificationsWithDeviceToken](https://developer.apple.com/reference/uikit/uiapplicationdelegate) to register to the uninstall feature.

*Example:*

```objective-c
@import AppsFlyerLib;

...

- (void)application:(UIApplication ​*)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *​)deviceToken {
// notify AppsFlyerTracker
[[AppsFlyerTracker sharedTracker] registerUninstall:deviceToken];
}
```

Read more about Uninstall register: [Appsflyer SDK support site](https://support.appsflyer.com/hc/en-us/articles/207032066-AppsFlyer-SDK-Integration-iOS)

#### <a id="track-app-uninstalls-android"> Android


Updates Firebase device token so it can be sent to AppsFlyer

*Example:*

```javascript
appsFlyer.updateServerUninstallToken(newFirebaseToken, (success) => {
  //...
});
```

Read more about Android  Uninstall Tracking: [Appsflyer SDK support site](https://support.appsflyer.com/hc/en-us/articles/208004986-Android-Uninstall-Tracking)

