# Incoming Call Version 1.0.3

> React Native module to display custom incoming call activity, best result when using with firebase background messaging. Only for Android since iOS we have VoIP.

Yes I heard you could use **self managed ConnectionService** thing. But since I'm not an Android expert, this is a solution I found acceptable.

You could also wait for [this feature request](https://github.com/react-native-webrtc/react-native-callkeep/issues/43) from `react-native-callkeep` to be resolved and have an easier life.

<img width="400" src="https://cbe.themaestro.in/Bharath/incoming-call.jpg">

## Getting started

`$ npm install react-native-incoming-call-android --save`

### Addition installation step

In `AndroidManifest.xml`:

- Add `<activity android:name="com.incomingcall.UnlockScreenActivity" />` line between `<application>` tag.

- Add `<uses-permission android:name="android.permission.VIBRATE" />` permission.

- Also, it's recommend to put `android:launchMode="singleInstance"` in `<activity android:name=".MainActivity"...` tag to prevent duplicate activities.

For RN >= 0.60, it's done. Otherwise:

`$ react-native link react-native-incoming-call-android`

## Usage

In `App.js`:

```javascript
import {useEffect} from 'react';
import {DeviceEventEmitter, Platform} from 'react-native';
import IncomingCall from 'react-native-incoming-call-android';

// Listen to cancel and answer call events
useEffect(() => {
  if (Platform.OS === "android") {
    /**
     * App open from killed state (headless mode)
    */
    const payload = await IncomingCall.getExtrasFromHeadlessMode();
    console.log('launchParameters', payload);
    if (payload) {
      // Start call action here. You probably want to navigate to some CallRoom screen with the payload.uuid.
    }

    /**
     * App in foreground / background: listen to call events and determine what to do next
    */
    DeviceEventEmitter.addListener("endCall", payload => {
      // End call action here
    });
    DeviceEventEmitter.addListener("answerCall", payload => {
      // Start call action here. You probably want to navigate to some CallRoom screen with the payload.uuid.
    });
  }
}, []);
```

In `index.js` or anywhere firebase background handler lies: 

```javascript
import messaging from '@react-native-firebase/messaging';
import {DeviceEventEmitter} from 'react-native';
import IncomingCall from 'react-native-incoming-call-android';

messaging().setBackgroundMessageHandler(async remoteMessage => {
  // Receive remote message
  if (remoteMessage?.notification?.title === 'Incoming Call') {
    // Display incoming call activity.
    IncomingCall.display(
      'callUUIDv4', // Call UUID v4
      'Quocs', // Username
      'https://avatars3.githubusercontent.com/u/16166195', // Avatar URL
      'Incomming Call', // Info text
      20000 // Timeout for end call after 20s
    );
  } else if (remoteMessage?.notification?.title === 'Missed Call') {
    // Terminate incoming activity. Should be called when call expired.
    IncomingCall.dismiss();
  }

  // Listen to headless action events
  DeviceEventEmitter.addListener("endCall", payload => {
    // End call action here
  });
  DeviceEventEmitter.addListener("answerCall", (payload) => {
    console.log('answerCall', payload);
    if (payload.isHeadless) {
      // Called from killed state
      IncomingCall.openAppFromHeadlessMode(payload.uuid);
    } else {
      // Called from background state
      IncomingCall.backToForeground();
    }
  });
});
```
