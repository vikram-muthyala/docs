---
title: Push notifications
excerpt: ''
hidden: false
next:
  pages:
    - voice-android-cloud-user-controller
---

## Receiving incoming calls via push notifications

The application may receive incoming calls only when:

- the application is in the foreground AND listening on active connection;
- the application is registered to receive incoming calls via the Push Notifications.

When an application is not running, or the `Active Connection` feature is not enabled, the user must be notified of an incoming call by a push notification.

By invoking `setSupportManagedPush(true)` the Sinch SDK will automatically register to _Firebase Cloud Messaging_ and the Sinch backend will initiate push messages to your application when needed. This feature requires _Google Play Services_ on the device. If you distribute your application through other channels than Google Play, push notifications will not be available on devices that do not have _Google Play Services_.

If using the Sinch backend and _Firebase Cloud Messaging_ is not viable in the application, please see [Active connection](https://developers.sinch.com/docs/voice-android-cloud-active-connection).

As a developer, you will be responsible for implementing the code that receives the FCM push message. For an example implementation, please see the sample app “Sinch Push” which is bundled with the SDK.

Sinch SDK moved from deprecated _Google Cloud Messaging_ (GCM) to its most up-to-date and Google-recommended version _Firebase Cloud Messaging_ (FCM), which requires client app to be modified in accordance with the Google’s official [GCM to FCM migration guide](https://developers.google.com/cloud-messaging/android/android-migrate-fcm)

The following sections cover how to support receiving calls and messages via push notifications.

## FCM configuration file required (google-services.json)

You can add Firebase to your app either semi-automatically using Android Studio, or manually [following this step-by-step official guide](https://firebase.google.com/docs/android/setup). In brief, to perform manual setup you first need to register your application in [firebase console](https://console.firebase.google.com/). If your project already uses GCM, the console will prompt you to import it as new Firebase Cloud Messaging project. Register your application using the console, and download relevant google-services.json into your project’s main folder.

Sample SDK projects _sinch-rtc-sample-push_ and _sinch-rtc-sample-video-push_ will require you to supply your own _google-services.json_ in order to be built. In the absence of this file, gradle will show relevant error with explanation and relevant links and stop the build. That _google-services.json_ file is the main mean of automatization of support of Firebase services to your app. Android Studio’s _‘com.google.gms.google-services’_ plugin parses and adds relevant resources and permissions to your applications manifest automatically.

## Permissions required

Unlike GCM setup, FCM application developer does not need to manually add any permission to application manifest. For relevant changes in you application’s manifest when migrating from GCM to FCM please consult official [GCM to FCM migration guide](https://developers.google.com/cloud-messaging/android/android-migrate-fcm)

## Enable push notifications

To enable push notifications, set the following capability before starting the Sinch client:

```java
sinchClient.setSupportManagedPush(true);
sinchClient.start();
```

> **Note**
>
> - You must catch the `MissingGCMException` if you distribute your app to devices without _Google Play Services_.
> - Using `setSupportManagedPush(true)` will register a token with Firebase Cloud Messaging using a Sender ID connected to Sinch, which will _NOT_ unregister your own token, so you _CAN_ use Firebase Cloud Messages for your own purpose filtering them in _onMessageReceived(RemoteMessage remoteMessage)_ method of your FCM Listening Service using Sinch helper API _SinchHelpers.isSinchPushPayload_.

```java
public class FcmListenerService extends FirebaseMessagingService {

@Override
public void onMessageReceived(RemoteMessage remoteMessage){
    if (SinchHelpers.isSinchPushPayload(remoteMessage.getData())) {
        // it's Sinch message - relay it to SinchClient
    } else {
        // it's NOT Sinch message - process yourself
    }
  }}
```

### Explicit push token registration

There are certain situations where it is either desirable to explicitly register push token and/or get assurance that the push token is indeed registered. These situations should be handled using new [UserController API](doc:voice-android-cloud-user-controller).

## Receive and forward push notifications to a Sinch client

For more details regarding how to implement receiving a FCM downstream message, please see the [Android developer site for FCM](https://firebase.google.com/docs/cloud-messaging/android/receive).

Once you have received the `RemoteMessage` in your `FirebaseMessagingService`, forward it to the Sinch client using the method `relayRemotePushNotificationPayload`.

```java
// make sure you have created a SinchClient
if (SinchHelpers.isSinchPushPayload(remoteMessage.getData())) {
    NotificationResult result = sinchClient.relayRemotePushNotificationPayload(remoteMessage.getData());
}
```

The returned `result` can be inspected to see whether the push was for an instant message or a call using `result.isMessage()` and `result.isCall()`.

### Incoming call

If the payload that was forwarded to the Sinch client was for a call, the `onIncomingCall` callback will automatically be triggered as for any other call. The `CallNotificationResult` object provides details about participants, whether the call timed out and whether the call offers video.

### Send and receive custom headers via Sinch managed push

The Sinch SDK supports adding custom headers in push notification messages when initiating a call, so developers do not need to implement their own push mechanism if they only need to deliver small pieces of information along the Sinch managed push between their app instances. The Sinch SDK allows up to _1024_ bytes of custom headers.

Setting custom headers on the sender side when initiating a call:

```java
// setting up custom headers
Map<String,String> headers = new HashMap<>();
headers.put("The first value is ", "@123");
headers.put("Custom value ", "two!");
Call call = callClient.callUser(userId, headers);
```

If custom headers were supplied by call initiator, they can be retrieved from notification result using `getHeaders()` API:

```java
// make sure you have created a SinchClient
if (SinchHelpers.isSinchPushPayload(remoteMessage.getData())) {
NotificationResult result = sinchClient.relayRemotePushNotificationPayload(remoteMessage.getData());
  if (result.isCall()) {
    CallNotificationResult callResult = result.getCallResult();
    Map<String, String> customHeaders = callResult.getHeaders());
  }
}
```

> **Note**
>
> It is possible to retrieve custom headers from the push message using _SinchHelpers.queryPushNotificationPayload_ without starting the client.

```java
// SinchClient is not needed to be created at all!
if (SinchHelpers.isSinchPushPayload(remoteMessage.getData())) {
  NotificationResult result = SinchHelpers.queryPushNotificationPayload(applicationContext, remoteMessage.getData());
  if (result.isCall()) {
    CallNotificationResult callResult = result.getCallResult();
    Map<String, String> customHeaders = callResult.getHeaders());
    // analyse headers, decide whether to process message/call and start Sinch client or ignore
    ...
  }
}
```

### Show local notifications for missed calls

When the push notification is enabled on a Sinch client, besides the incoming call notification, the Sinch SDK will also send a push notification for a canceled call when the caller cancels the call before it is answered. This gives developers a good opportunity to present local notifications for missed calls in their apps:

```java
NotificationResult result = sinchService.relayRemotePushNotificationPayload(payload);
// handle result, e.g. show a notification for a missed call:
if (result.isValid() && result.isCall()) {
    CallNotificationResult callResult = result.getCallResult();
    if (callResult.isCallCanceled()) {
        // user-defined method to show notification
        createNotification(callResult.getRemoteUserId());
    }
}
```

> **Note**
>
> If the message forwarded to the Sinch client happened to be _call cancel_ message, the client arranges the termination of the call automatically resulting in `CallListener.onCallEnded()` event being triggered, allowing UI to handle canceling the call.

## Unregister a device

If the user of the application logs out or performs a similar action, the push notification device token can be unregistered via `SinchClient.unregisterManagedPush()` to prevent further notifications to be sent to the device. Starting a client with `setSupportManagedPush(true)` will register the device again.

## Active Connection

If push notifications are not desired, the alternative is to use `setSupportActiveConnectionInBackground(true)` and then calling `startListeningOnActiveConnection()` to enable incoming calls. Don’t forget to call `stopListeningOnActiveConnection()` when the user is no longer available for calls (for example if the application is no longer active).
