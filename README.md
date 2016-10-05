# UnityDeeplinks
A set of tools for Unity to allow handling deeplink activation from within Unity scripts, for Android and iOS, including iOS Universal Links.

### Disclaimer
This is NOT a TROPHiT SDK - this repo is an open-source contribution to developers for handling deeplink activations in a unified way for both iOS and Android. It can be used independently and regardless of TROPHiT services in order to intercept deeplinks for whatever purpose. If you are looking for info about TROPHiT integration modules, visit the [TROPHiT Help Center](https://trophit.zendesk.com/hc/en-us/articles/200865062-How-do-I-integrate-TROPHiT-)

# Usage
#### Example: Track Deeplinks with Adjust
* Tested with [Adjust Unity SDK](https://github.com/adjust/unity_sdk) v4.10.0
* Also enables Adjust's SDK to handle iOS Universal Links
* Assuming you already integrated the Adjust SDK, just implement `onDeeplink` in *UnityDeeplinks.cs* as follows:
```cs
public void onDeeplink(string deeplink) {
    AdjustEvent adjustEvent = new AdjustEvent("abc123");
    adjustEvent.addCallbackParameter("deeplink", deeplink); // optional, for callback support
    Adjust.trackEvent(adjustEvent);
}
```
* Add the following code marked `add this` to *Assets/UnitDeeplinks/iOS/UnityDeeplinks.mm*:
```objc
#import "Adjust.h"  // <==== add this
...

- (void)onNotification:(NSNotification*)notification {
    if (![kUnityOnOpenURL isEqualToString:notification.name]) return;
    ...
    [Adjust appWillOpenUrl:url]; // <==== add this right before UnityDeeplinks_dispatch
    UnityDeeplinks_dispatch([url absoluteString]);
}

```

#### Example: Track Deeplinks with Tune
* Tested with [Tune Unity Plugin](https://developers.tune.com/sdk/unity-quick-start/) v4.3.1
* Also enables Tune's Plugin to handle iOS Universal Links
* Assuming you already integrated the Tune Plugin, just implement `onDeeplink` in *UnityDeeplinks.cs* as follows:
```cs
public void onDeeplink(string deeplink) {
   TuneEvent event = new TuneEvent("deeplink");
   event.attribute1 = deeplink;
   Tune.MeasureEvent(event);
}
```

#### Example: Track Deeplinks with Kochava
* Also enables Kochava's SDK to handle iOS Universal Links
* Assuming you already integrated the [Kochava Unity SDK](http://support.kochava.com/sdk-integration/unity-sdk-integration), just implement `onDeeplink` in *UnityDeeplinks.cs* as follows:
```cs
public void onDeeplink(string deeplink) {
   Kochava.DeeplinkEvent(deeplink, null);
}
```

#### Example: Track Deeplinks with AppsFlyer
* Tested with [AppsFlyer Unity SDK](https://support.appsflyer.com/hc/en-us/articles/213766183-Unity) v4.10.1
* Also enables AppsFlyer's SDK to handle iOS Universal Links (for AppsFlyer Unity SDK 4.10 or earlier)
* Assuming you already integrated the [AppsFlyer Unity SDK](https://support.appsflyer.com/hc/en-us/articles/213766183-Unity), just implement `onAppOpenAttribution` in *AppsFlyerTrackerCallbacks.cs* as follows:
```cs
public void onAppOpenAttribution(string validateResult) {
	print("AppsFlyerTrackerCallbacks:: got onAppOpenAttribution  = " + validateResult);
	System.Collections.Generic.Dictionary<string, string> values =
		new System.Collections.Generic.Dictionary<string, string>();
	values.Add("data", validateResult);
	AppsFlyer.trackRichEvent("deeplink", values);
}
```

# Integration
* Clone/download the repository
* Copy the entire UnityDeeplinks folder into your Unity project Assets folder
* **If you are using AppsFlyer, skip the next steps to this [section](#appsflyer). Otherwise, continue**
* Attach the *Assets/UnityDeeplinks/UnityDeeplinks.cs* script to an empty *UnityDeeplinks* game object

## Android
There are two alternatives to handle a deeplink by a Unity app, depending on how your Unity project is currently built. It's up to you to decide which way to go.

### Alternative 1: Subclassing UnityPlayerActivity
In this approach, you use a subclass of the default *UnityPlayerActivity*, which contains deeplink-handling code that marshals deeplinks into your Unity script. This is the recommended approach, as it is the least complex. If you have no previous plans to subsclass UnityPlayerActivity, it's a good approach because then the subclassed code would not conflict with anything. If you do have plans to subclass UnityPlayerActivity, however, it would usually also be a good approach as the code changes you need to make are minimal. You will have to use the provided *MyUnityPlayerActivity* subclass as follows:

* Replace the default UnityPlayerActivity in your Assets/Plugins/Android/AndroidManifest.xml with com.trophit.MyUnityPlayerActivity:

```xml
<!--
<activity android:name="com.unity3d.player.UnityPlayerActivity" ...
-->
<activity android:name="com.trophit.MyUnityPlayerActivity" ...
```

* Add the following inside the <activity> tag, assuming your deeplink URL scheme is myapp://
```xml
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="myapp" />
    </intent-filter>
```

* Optional: by default, *MyUnityPlayerActivity* calls a Unity script method `onDeeplink` on a game object called *UnityDeeplinks*. If you wish to change the name of the object or method, you should edit *Assets/UnityDeeplinks/Android/MyUnityPlayerActivity.java*, change the values of the `gameObject` and/or `deeplinkMethod` static properties and rebuild the *UnityDeeplinks.jar* file as instructed below

### Alternative 2: Adding a Deeplink Activity
In this approach, a second activity with deeplink-handling code is added to the Unity project, without subclassing the default activity. Use this is case where option #1 is not acceptable (code is too complex, not under your control, cannot be subclassed, etc)

* Add the following activity to your Assets/Plugins/Android/AndroidManifest.xml, assuming your deeplink URL scheme is myapp://
```xml
<activity android:name="com.trophit.DeeplinkActivity" android:exported="true">
	<intent-filter>
		<action android:name="android.intent.action.VIEW" />
		<category android:name="android.intent.category.DEFAULT" />
		<category android:name="android.intent.category.BROWSABLE" />
		<data android:scheme="myapp" />
 	</intent-filter>
</activity>
```

* Optional: by default, *DeeplinkActivity* calls a Unity script method `onDeeplink` on a game object called *UnityDeeplinks*. If you wish to change the name of the object or method, you should edit *Assets/UnityDeeplinks/Android/DeeplinkActivity.java*, change the values of the `gameObject` and/or `deeplinkMethod` static properties and rebuild the *UnityDeeplinks.jar* file as instructed below

### Building the UnityDeeplinks.jar file
Only perform this step if you made changes to any .java file under *Assets/UnityDeeplinks/Android/* or would like to rebuild it using an updated version of Unity classes, Android SDK, JDK and so on.

#### Prerequisites
* Go to your Unity => Preferences => External tools menu and find your Android SDK and JDK home folders
* Edit *UnityDeeplinks/Android/bulid_jar.sh*
* Ensure ANDROID_SDK_ROOT points to your Android SDK root folder
* Ensure JDK_HOME points to your JDK root folder
* Ensure UNITY_LIBS points to the Unity classes.jar file in your development environment

#### Build instructions
* Run the build script:
```
cd MY_UNITY_PROJECT_ROOT/Assets/UnityDeeplinks/Android
./build_jar.sh
```
Example output:
```
Compiling ...
/Library/Java/JavaVirtualMachines/jdk1.7.0_51.jdk/Contents/Home/jre/lib
/Applications/Unity/PlaybackEngines/AndroidPlayer/Variations/mono/Release/Classes/classes.jar:/kankado/dev/tools/android-sdk-macosx/platforms/android-23/android.jar
Creating jar file...
adding: com/(in = 0) (out= 0)(stored 0%)
adding: com/trophit/(in = 0) (out= 0)(stored 0%)
adding: com/trophit/DeeplinkActivity.class(in = 2368) (out= 1237)(deflated 47%)
adding: com/trophit/MyUnityPlayerActivity.class(in = 1504) (out= 789)(deflated 47%)
```
This creates/updates a *UnityDeeplinks.jar* file under your Unity project's Assets/UnityDeeplinks folder

* Continue to build and test your Unity project as usual in order for any jar changes to take effect

## iOS
UnityDeeplinks implements a native plugin for iOS, initialized by *Assets/UnityDeeplinks/UnityDeeplinks.cs*. the plugin listens for URL/Univeral Link activations and relayes them to the Unity script for processing.

* Ensure your XCode project's Info.plist file contains a custom URL scheme definiton or [Universal Links setup](https://developer.apple.com/library/content/documentation/General/Conceptual/AppSearch/UniversalLinks.html). Here is an example of a custom URL scheme *myapp://* for the bundle ID *com.mycompany.myapp*:
```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLName</key>
        <string>com.mycompany.myapp</string>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>myapp</string>
        </array>
    </dict>
</array>
```
*Note:* Custom URL scheme settings may be removed during Unity rebuilds. Ensure they are recreated if needed, by a post-build script, for example. Universal Link settings are not removed during Unity rebuilds.

## Testing

* Prepare a dummy web page that is accessible by your mobile device:
```xml
<body>
<a href="myapp://?a=b">deeplink test</a>
</body>
```

* Open the web page on the device browser and click the deeplink
* The Unity app should open and the onDeeplink Unity script method should be invoked, performing whatever it is you designed it to perform

# Special Cases

## AppsFlyer
AppsFlyer already provides some implementation for iOS and Android to handle deeplinks. However, it does not behave consistently for iOS and Android when a deeplink is activated:
* For iOS, it triggers the *onAppOpenAttribution* method in the *AppsFlyerTrackerCallbacks* script
* For Android, it does not trigger any Unity script method

Fortunately, AppsFlyer provides an implementation similar to [Alternative #2](#alternative-2-adding-a-deeplink-activity) above for Android, so in order to make AppsFlyer behave consistently for Android, we simply need to add some code to their class and rebuild their native .jar file using tools they provide:
* First, ensure you have the [AppsFlyer Unity SDK](https://support.appsflyer.com/hc/en-us/articles/213766183-Unity) integrated including the deeplinking configuration
* Ensure you have your URL schemes or Universal Links set up
* Next, ensure you call `AppsFlyer.getConversionData();` in your AppsFlyer iOS startup script, right after `setAppId`:
```cs
#if UNITY_IOS
AppsFlyer.setAppID ("123456789");
AppsFlyer.getConversionData();
// ...
```
* Add the following to *Assets/Plugins/Android/src/GetDeepLinkingActivity.java* inside `onCreate` right after `this.starActivity(newIntent)` and right before `finish`:
```java
// this.startActivity(newIntent);
String deeplink = getIntent().getDataString();
if (deeplink != null) {
    try {
        org.json.JSONObject jo = new org.json.JSONObject();
        jo.put("link", getIntent().getDataString());
        com.unity3d.player.UnityPlayer.UnitySendMessage("AppsFlyerTrackerCallbacks", "onAppOpenAttribution", jo.toString());
    } catch (org.json.JSONException ex) {
        Log.e(TAG, "Unable to send deeplink to Unity", ex);
    }
}
// finish()
```
* Edit *Assets/Plugins/Android/src/build_plugin_jar.sh*
* Ensure, [like with UnityDeeplink's *build_jar.sh*](#building-the-unitydeeplinksjar-file) that all paths are set correctly
* Run the build script, which should rebuild *Assets/Plugins/Android/AppsFlyerAndroidPlugin.jar*
`./build_plugin_jar.sh`

* Finally, implement your `AppsFlyerTrackerCallbacks.onAppOpenAttribution` method as needed. Upon deeplink activation on iOS or Android, it receives a JSON string in the format:
`{"link":"deeplink url comes here"}`
* Proceed to [testing](#testing) as usual
