##What's Countly?
[Countly](http://count.ly) is an innovative, real-time, open source mobile analytics application. 
It collects data from mobile devices, and visualizes this information to analyze mobile application 
usage and end-user behavior. There are two parts of Countly: the server that collects and analyzes data, 
and mobile SDK that sends this data. Both parts are open source with different licensing terms.

This repository includes the SDK for Android.

##Installing Android SDK

Installing Android SDK requires two very easy steps. Countly Android SDK uses OpenUDID (which comes ready with the zip file). First step is about OpenUDID requirement and second step is integrating Countly SDK to your project:

###1. Add Countly SDK to your project

#### Gradle users:
Add Maven Central repository:
<pre class="prettyprint">
repositories {
    mavenCentral()
}
</pre>

Add Countly SDK dependency:
<pre class="prettyprint">
dependencies {
    compile 'ly.count:sdk-android:+'
}
</pre>


#### Maven users:
<pre class="prettyprint">
&lt;dependency&gt;
    &lt;groupId&gt;ly.count&lt;/groupId&gt;
    &lt;artifactId&gt;sdk-android&lt;/artifactId&gt;
    &lt;version&gt;14.11&lt;/version&gt;
&lt;/dependency&gt;
</pre>

#### Eclipse users:
Download [Latest JAR](https://github.com/Countly/countly-sdk-android/releases/latest) and put it into your lib folder.



###2. Set up Android SDK

Call `Countly.sharedInstance().init(this, "https://YOUR_SERVER", "YOUR_APP_KEY", "OPTIONAL_DEVICE_ID")` in your main activity onCreate, which requires your App key and the URL of your Countly server (use `https://cloud.count.ly` for Countly Cloud). You can either specify your own Device ID, or omit this parameter and add OpenUDID service (it will generate unique device ID automatically) to your `AndroidManifest.xml`:

<pre class="prettyprint">
&lt;service android:name="org.openudid.OpenUDID_service">
    &lt;intent-filter>
        &lt;action android:name="org.openudid.GETUDID" />
    &lt;/intent-filter>
&lt;/service>
</pre>

* Call `Countly.sharedInstance().onStart()` in onStart.
* Call `Countly.sharedInstance().onStop()` in onStop.


Additionally, make sure that *INTERNET* permission is set if there's none in your manifest file.

**Note:** Make sure you use App Key (found under Management -> Applications) and not API Key. Entering API Key will not work. 

**Note:** Call `Countly.sharedInstance().init(...)` only once during onCreate of main activity. After that, call `Countly.sharedInstance().onStart()` and `Countly.sharedInstance().onStop()` in each of your activities' `onStart()` and `onStop()` methods. 

###3. Using Countly Push Notifications

Countly can send messages to your users too! To enable it, go to Google API Console and turn GCM on for your app. Then, instead of `ly.count:sdk-android:+` dependency or `sdk-android-14.11.jar`, use `ly.count:sdk-android-messaging:+` and `sdk-android-messaging-14.11.jar` respectively. Additionally, you'll need to enable GCM itself (see `countly-android-example-messaging` folder for example app built with Android Studio):

**Add extra lines in `AndroidManifest.xml`**

Make sure your app requests these permissions (replace `ly.count.android.example.messaging` with your app package):
<pre class="prettyprint">
&lt;uses-permission android:name="android.permission.INTERNET"/&gt;
&lt;uses-permission android:name="com.google.android.c2dm.permission.RECEIVE"/&gt;
&lt;uses-permission android:name="android.permission.GET_ACCOUNTS"/&gt;
&lt;uses-permission android:name="android.permission.WAKE_LOCK"/&gt;
&lt;uses-permission android:name="ly.count.android.api.permission.C2D_MESSAGE"/&gt;
&lt;permission android:name="ly.count.android.example.messaging.permission.C2D_MESSAGE" android:protectionLevel="signature" /&gt;
&lt;uses-permission android:name="ly.count.android.example.messaging.permission.C2D_MESSAGE" /&gt;
</pre>

Add GMS version to the `<application>` element:
``<meta-data android:name="com.google.android.gms.version" android:value="@integer/google_play_services_version" />``

Then specify `ProxyActivity` (used to display `Dialog`'s and handle user actions), `CountlyMessaging` (it's our `WakefulBroadcastReceiver`) and `CountlyMessagingService` (`IntentService` which processes incoming notifications):
<pre class="prettyprint">
&lt;activity
    android:name="ly.count.android.api.messaging.ProxyActivity"
    android:label="@string/app_name" android:theme="@android:style/Theme.Translucent" android:noHistory="true"/&gt;
&lt;receiver
    android:name="ly.count.android.api.messaging.CountlyMessaging"
    android:permission="com.google.android.c2dm.permission.SEND" &gt;
    &lt;intent-filter&gt;
        &lt;action android:name="com.google.android.c2dm.intent.RECEIVE" /&gt;
        &lt;category android:name="ly.count.android.api" /&gt;
    &lt;/intent-filter&gt;
&lt;/receiver&gt;
&lt;service android:name="ly.count.android.api.messaging.CountlyMessagingService" &gt;
    &lt;meta-data android:name="broadcast_action" android:value="ly.count.android.api.broadcast" /&gt;
&lt;/service&gt;

&lt;service android:name="org.OpenUDID.OpenUDID_service"&gt;
    &lt;intent-filter&gt;
        &lt;action android:name="org.OpenUDID.GETUDID" /&gt;
    &lt;/intent-filter&gt;
&lt;/service&gt;
</pre>

**Add Support & Play Services dependencies**
Depending on what IDE you use you'll need to include corresponding library projects, or just add a couple of new lines into your `build.gradle`:
<pre class="prettyprint">
dependencies {
    compile 'com.android.support:appcompat-v7:20.0.0'
    compile 'com.google.android.gms:play-services:5.2.08'
}
</pre>
 
 **And, finally, change the way you init Countly**
 <pre class="prettyprint">
Countly.sharedInstance()
    .init(this, "YOUR_SERVER", "APP_KEY")
    .initMessaging(this, CountlyActivity.class, "PROJECT_ID", Countly.CountlyMessagingMode.TEST);
</pre>
Where `PROJECT_ID` is ID of your project from Google API Console. Note, that you can use one of two `CountlyMessagingMode`'s: `TEST` and `PRODUCTION`. This feature is very handy when you want to send your message just to your beta testers. 


**There are two extra features:**
To localize `Dialog`s Countly show whenever text message is received you can supply additional array of strings containing localized version of `new String[]{"Open", "Review"}`.

To get notified whenever Countly receives a message, register your `BroadcastReceiver` like this:
<pre class="prettyprint">
IntentFilter filter = new IntentFilter();
filter.addAction(CountlyMessaging.getBroadcastAction(getApplicationContext()));
registerReceiver(new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        Message message = intent.getParcelableExtra(CountlyMessaging.BROADCAST_RECEIVER_ACTION_MESSAGE);
        Log.i("CountlyActivity", "Got a message with data: " + message.getData());
    }
}, filter);
</pre>


###4. Use some extra features
**How can I make sure that requests to Countly are sent correctly?**

Enable logging: `Countly.sharedInstance().setLoggingEnabled(true)`

**I'm already using OpenUDID in my project, how can I remove it from build?**

There is `custom_rules.xml` ant build file, check it out. Or you can just use sources instead of jars.

###5. Other

Check Countly Server source code here: 

- [Countly Server (countly-server)](https://github.com/Countly/countly-server)

There are also other Countly SDK repositories below:

- [Countly iOS SDK](https://github.com/Countly/countly-sdk-ios)
- [Countly Android SDK](https://github.com/Countly/countly-sdk-android)
- [Countly Windows Phone SDK](https://github.com/Countly/countly-sdk-windows-phone)
- [Countly Blackberry Webworks SDK](https://github.com/Countly/countly-sdk-blackberry-webworks)
- [Countly Blackberry Cascades SDK](https://github.com/craigmj/countly-sdk-blackberry10-cascades) (Community supported)
- [Countly Mac OS X SDK](https://github.com/mrballoon/countly-sdk-osx) (Community supported)
- [Countly Appcelerator Titanium SDK](https://github.com/euforic/Titanium-Count.ly) (Community supported)
- [Countly Unity3D SDK](https://github.com/Countly/countly-sdk-unity) (Community supported)

##How can I help you with your efforts?
Glad you asked. We need ideas, feedbacks and constructive comments. All your suggestions will be taken care with upmost importance. 

We are on [Twitter](http://twitter.com/gocountly) and [Facebook](http://www.facebook.com/Countly) if you would like to keep up with our fast progress!

For community support page, see [http://support.count.ly](http://support.count.ly "Countly Support").
