# jitsi_meet

Jitsi Meet Plugin for Flutter. Supports Android and iOS platforms.

"Jitsi Meet is an open-source (Apache) WebRTC JavaScript application that uses Jitsi Videobridge to provide high quality, secure and scalable video conferences." 

Find more information about Jitsi Meet [here](https://github.com/jitsi/jitsi-meet)

## Table of Contents
* [ Configuration](#configuration)
  * [ Android](#android)
* [ Join A Meeting](#join-a-meeting)
* [ JitsiMeetingOptions](#jitsimeetingoptions)
* [ JitsiMeetingResponse](#jitsimeetingresponse)
* [ Listening to Meeting Events](#listening-to-meeting-events)
* [ Contributing](#contributing)

<a name="configuration"></a>
## Configuration

<a name="android"></a>
### Android

#### Gradle
Set dependencies of build tools gradle to minimum 3.6.3:
```xml
dependencies {
    classpath 'com.android.tools.build:gradle:3.6.3' <!-- Upgrade this -->
    classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
}
```

Set distribution gradle wrapper to minimum 5.6.4.
```xml
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-5.6.4-all.zip <!-- Upgrade this -->
```

Add Java 1.8 compatibility support to your project by adding the following lines into your build.gradle file:
```xml
compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
}
```

#### AndroidManifest.xml
Jitsi Meet's SDK AndroidManifest.xml will conflict with your project, namely 
the application:label field. To counter that, go into 
`android/app/src/main/AndroidManifest.xml` and add the tools library
and `tools:replace="android:label"` to the application tag.

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="yourpackage.com"
    xmlns:tools="http://schemas.android.com/tools"> <!-- Add this -->
    <application 
        tools:replace="android:label"  
        android:name="your.application.name"
        android:label="My Application"
        android:icon="@mipmap/ic_launcher">
        ...
    </application>
...
</manifest>
```

#### Minimum SDK Version 21
Update your minimum sdk version to 21 in android/app/build.gradle
```groovy
defaultConfig {
    // TODO: Specify your own unique Application ID (https://developer.android.com/studio/build/application-id.html).
    applicationId "com.gunschu.jitsi_meet_example"
    minSdkVersion 21 //Required for Jitsi
    targetSdkVersion 28
    versionCode flutterVersionCode.toInteger()
    versionName flutterVersionName
}
```

#### Proguard

Jitsi's SDK enables proguard, but without a proguard-rules.pro file, your release 
apk build will be missing the Flutter Wrapper as well as react-native code. 
In your Flutter project's android/app/build.gradle file, add proguard support

```groovy
buildTypes {
    release {
        // TODO: Add your own signing config for the release build.
        // Signing with the debug keys for now, so `flutter run --release` works.
        signingConfig signingConfigs.debug
        
        // Add below 3 lines for proguard
        minifyEnabled true
        useProguard true
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
}
```

Then add a file in the same directory called proguard-rules.pro. See the example 
app's [proguard-rules.pro](example/android/app/proguard-rules.pro) file to know what to paste in.

*Note*  
If you do not create the proguard-rules.pro file, then your app will 
crash when you try to join a meeting or the meeting screen tries to open
but closes immediately. You will see one of the below errors in logcat.

```
## App crashes ##
java.lang.RuntimeException: Parcel android.os.Parcel@8530c57: Unmarshalling unknown type code 7536745 at offset 104
    at android.os.Parcel.readValue(Parcel.java:2747)
    at android.os.Parcel.readSparseArrayInternal(Parcel.java:3118)
    at android.os.Parcel.readSparseArray(Parcel.java:2351)
    .....
```

```
## Meeting won't open and you go to previous screen ##
W/unknown:ViewManagerPropertyUpdater: Could not find generated setter for class com.BV.LinearGradient.LinearGradientManager
W/unknown:ViewManagerPropertyUpdater: Could not find generated setter for class com.facebook.react.uimanager.g
W/unknown:ViewManagerPropertyUpdater: Could not find generated setter for class com.facebook.react.views.art.ARTGroupViewManager
W/unknown:ViewManagerPropertyUpdater: Could not find generated setter for class com.facebook.react.views.art.a
.....
```

<a name="join-a-meeting"></a>
### Join A Meeting

```dart
_joinMeeting() async {
    try {
      var options = JitsiMeetingOptions()
        ..room = "myroom" // Required, spaces will be trimmed
        ..serverURL = "https://someHost.com"
        ..subject = "Meeting with Gunschu"
        ..userDisplayName = "My Name"
        ..userEmail = "myemail@email.com"
        ..audioOnly = true
        ..audioMuted = true
        ..videoMuted = true;

      await JitsiMeet.joinMeeting(options);
    } catch (error) {
      debugPrint("error: $error");
    }
  }
```

<a name="jitsimeetingoptions"></a>
### JitsiMeetingOptions

| Field           | Required  | Default   | Description |
| --------------- | --------- | --------- | ----------- |
| room            | Yes       | N/A              | Unique room name that will be appended to serverURL. Valid characters: alphanumeric, dashes, and underscores. |
| subject         | No        | $room            | Meeting name displayed at the top of the meeting.  If null, defaults to room name where dashes and underscores are replaced with spaces and first characters are capitalized. |
| userDisplayName | No        | "Fellow Jitster" | User's display name |
| userEmail       | No        | none             | User's email address |
| audioOnly       | No        | false            | Start meeting without video. Can be turned on in meeting. |
| audioMuted      | No        | false            | Start meeting with audio muted. Can be turned on in meeting. |
| videoMuted      | No        | false            | Start meeting with video muted. Can be turned on in meeting. |
| serverURL       | No        | meet.jitsi.si    | Specify your own hosted server. Must be a valid absolute URL of the format `<scheme>://<host>[/path]`, i.e. https://someHost.com. Defaults to Jitsi Meet's servers. |
| userAvatarURL   | N/A       | none             | *Not yet implemented*. User's avatar URL. |
| token           | N/A       | none             | *Not yet implemented*. JWT token used for authentication. |

<a name="jitsimeetingresponse"></a>
### JitsiMeetingResponse

| Field           | Type    | Description |
| --------------- | ------- | ----------- |
| isSuccess       | bool    | Success indicator. |
| message         | String  | Success message or error as a String. |
| error           | dynamic | Optional, only exists if isSuccess is false. The error object. |

<a name="listening-to-meeting-events"></a>
### Listening to Meeting Events

Events supported

| Name                   | Description  |
| :--------------------- | :----------- |
| onConferenceWillJoin   | Meeting is loading. |
| onConferenceJoined     | User has joined meeting. |
| onConferenceTerminated | User has exited the conference. |
| onError                | Error has occurred with listening to meeting events. |

#### Per Meeting Events
To listen to meeting events per meeting, pass in a JitsiMeetingListener
in joinMeeting. The listener will automatically be removed when an  
onConferenceTerminated event is fired.

```
await JitsiMeet.joinMeeting(options,
  listener: JitsiMeetingListener(onConferenceWillJoin: ({message}) {
    debugPrint("${options.room} will join with message: $message");
  }, onConferenceJoined: ({message}) {
    debugPrint("${options.room} joined with message: $message");
  }, onConferenceTerminated: ({message}) {
    debugPrint("${options.room} terminated with message: $message");
  }));
```

#### Global Meeting Events
To listen to global meeting events, simply add a JitsiMeetListener with  
`JitsiMeet.addListener(myListener)`. You can remove listeners using  
`JitsiMeet.removeListener(listener)` or `JitsiMeet.removeAllListeners()`.

```dart
@override
void initState() {
  super.initState();
  JitsiMeet.addListener(JitsiMeetingListener(
    onConferenceWillJoin: _onConferenceWillJoin,
    onConferenceJoined: _onConferenceJoined,
    onConferenceTerminated: _onConferenceTerminated,
    onError: _onError));
}

@override
void dispose() {
  super.dispose();
  JitsiMeet.removeAllListeners();
}

_onConferenceWillJoin({message}) {
  debugPrint("_onConferenceWillJoin broadcasted");
}

_onConferenceJoined({message}) {
  debugPrint("_onConferenceJoined broadcasted");
}

_onConferenceTerminated({message}) {
  debugPrint("_onConferenceTerminated broadcasted");
}

_onError(error) {
  debugPrint("_onError broadcasted");
}
```

### Title bar
When Jitsi Meet is opening, the title bar will reflect: 
* For Android: the `android:label` tag in the AndroidManifest.xml in <application>
* For iOS: the `Bundle name` in Info.plist
For IOS - Support added for Xcode 11.4
Ensure in your Podfile  you have an entry like below declaring platform of 11.0 or above.
platform :ios, '11.0'
Also in Info.plist add the following
   <key>NSCameraUsageDescription</key>
    <string>$(PRODUCT_NAME) Camera use</string>
    <key>NSMicrophoneUsageDescription</key>
    <string>$(PRODUCT_NAME) Microphone use</string>
This will ensure the right permissions are configured in your project. 

<a name="contributing"></a>
## Contributing
Send a pull request with as much information as possible clearly 
describing the issue or feature. Try to keep changes small and 
for one issue at a time.
