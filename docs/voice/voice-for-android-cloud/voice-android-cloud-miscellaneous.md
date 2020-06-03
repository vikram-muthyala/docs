---
title: Miscellaneous
excerpt: ''
hidden: false
---

## Minimum requirements

We officially support the 3 latest major Android versions. You can try older versions but there are no guarantees it will work as expected.

## Production environments

Sinch provides two environments:

> - Production - Used for applications deployed in production.

The environment is passed as the parameter _environmentHost_ when instantiating the Sinch client.

| Environment | EnvironmentHost parameter |
| ----------- | ------------------------- |
| Production  | ocra.api.sinch.com        |

## Restrictions on User IDs

User IDs can only contain characters in the _printable ASCII character set_. That is:

```text
    !"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefghijklmnopqrstuvwxyz{|}~
```

User IDs **must not** be longer than **40** characters.

## Encryption export regulations

Please check the Summary of U.S. Export Controls Applicable to Commercial Encryption Products and ensure that the application is registered for the Encryption Regulations, if applicable. It can be found under this [link](https://www.bis.doc.gov/index.php/policy-guidance/encryption).

## Statistics

The Sinch SDK client uploads statistics to the Sinch servers at the end of a call, a call failure, or similar event. The statistics are used for monitoring of network status, call quality, and other aspects regarding the general quality of the service.

Some of the information is not anonymous and may be associated with the User ID call participants.

The statistics upload is done by the client in the background.

## Push Notifications sent via your application server

In general we strongly recommend using _“managed push notifications”_, that is, when push notifications are sent directly from the Sinch cloud, which is described in the section [Push notifications](doc:voice-android-cloud-push-notifications). The following section describes the integration of push notifications, given that your application server maintains the connection with Google Cloud Messaging.

An application is considered offline in the following scenarios:

> - When the application is not running
> - When background mode has been disabled for the Sinch client, and the application is not in the foreground

For these two scenarios, push notifications must be implemented in the application to be able to receive incoming calls. The following sections cover how to support receiving calls and messages via push notifications.

The Sinch client relies on a push service to launch the application if it is not currently listening for incoming calls or messages due to the application being offline. Which push service to use is up to the developer, but for Android applications, the typical choice is to use Google Cloud Messaging (GCM). The examples that follow assume that Google Cloud Messaging is used to deliver push messages.

When offline, the recipient of a call or message receives a push notification containing a Sinch-specific payload that enables the Sinch Client to connect the incoming call or message. Acting on the push notification brings the application to the foreground allowing the user to answer the call or view the message.
![push-sequence-diagram_android.png](images\7600ed4-push-sequence-diagram_android.png)

The above figure describes the following sequence of events: Both users start their applications and Sinch clients. When A (the caller) calls B (the callee), B’s application is in a state where it is not considered online (that is reachable using an active socket connection). Sinch notices that B is not online, and tells A to send a push notification to B so that B can answer the call.

When the Sinch client on the caller’s (or sender’s) side observes that the destination client is offline, it notifies the application that it needs to trigger the sending of a push notification to the recipient device.

### Push notification data

On startup, each instance of the application is expected to register a device identifier. The identifier is referred to as _push notification data_ and should be provided to the Sinch client using the method `registerPushNotificationData`.

Push notifications can be addressed to that identifier in the event that the application goes offline.

The push notification data can be any byte sequence; it is up to you to define its structure and what it contains. However, the push notification data must not exceed 1024 bytes. It should contain enough information to allow a push service to send a push notification to a particular user of the application on a particular device. For example, an Android exclusive application would likely use the GCM registration id as its push notification data.

Multi-platform applications may use a mix of different push services. For instance, in an application running on both iOS and Android, the platform identifier in the push notification data can be used by the push server to determine whether APNS or GCM should be used.

The device-specific push notification data should be registered on start up of the application, or as soon as it’s available. If user B then turns off the application, and user A calls B, user A’s application would get the callback `CallListener.onShouldSendPushNotification`. One of the parameters in this callback, is a list of `PushPair`s that contain a payload and a push notification data. Each element in this list corresponds to each of B’s registered push notification data identifiers (a user can have multiple devices).

The push notification data can also be unregistered by calling the `SinchClient.unregisterPushNotificationData` method. This effectively disables incoming calls or messages using push notifications for the particular device.

### Enable push notifications

The following sections assumes that GCM is used, but the use pattern for other push services is similar.

The easiest way to enable offline calls or messages using GCM is to first call `SinchClient.setSupportPushNotifications(true)` and then register the device specific push notification data with `SinchClient.registerPushNotificationData`. In a simple example we can use the registration id received from Google when registering to GCM.

```java
// Register with the GCM service to get a device specific registrationId
// Should be done in a background job
GoogleCloudMessaging gcm = GoogleCloudMessaging.getInstance(context);
String regId = gcm.register("Your-Sender-ID");

...

sinchClient.setSupportPushNotifications(true);
sinchClient.start();
sinchClient.registerPushNotificationData(regId);
```

Please refer to Google’s [Google Cloud Messaging for Android](http://developer.android.com/google/gcm/index.html) for more information on how to use the GCM service.

> **Note**
>
> As described in the [Push Data Notification](#push-notification-data) section, the data that you register with the `registerPushNotificationData` method is defined by you. If using GCM, it must at a minimum include the registrationId from Google (so a GCM server can push to a particular device).

### Send and receive push notifications

To send push messages the application developer must have a server that is configured for sending push notifications to the Google Cloud Messaging Service. Please see the [Sinch REST API User Guide](doc:using-rest) for details on how to handle feedback from Google Cloud Messaging Service.

Also refer to Google’s [Google Cloud Messaging for Android](http://developer.android.com/google/gcm/index.html) for detailed information on how GCM works.

#### On the caller side

When the recipient’s application is offline and the app needs to notify the user using a push notification, the caller’s or sender’s application is notified using the callback method `CallListener.onShouldSendPushNotification`.

The callback includes a List of `PushPair`s. The pairs contain a payload that is Sinch- and call-specific. Moreover the pairs contain a push data byte array. The Sinch specific payload should be embedded in the push notification sent to the recipient’s device(s). The push data is the same push data that the recipient’s application registered earlier. There might be multiple registered devices for the recipient user (for example, the same user is using the application on both a phone and a tablet), which is why the callback includes a List of Push Pairs.

```java
public void onShouldSendPushNotification(Call call, List<PushPair> pushPairs) {
    // Send payload and push data to application server
    // which should communicate with GCM Service to send push notifications.
}
```

A push notification should be sent to each device, where each entry in the parameter `pushPairs` list corresponds to one device. Each push notification should include the Sinch-specific payload so it can be forwarded to the Sinch client running on the destination device.

The Sinch-specific payload should be embedded as custom payload data in the GCM Payload.

```java
{
  "registration_ids" : ["APA91bHun4MxP5egoKMwt2KZFBaFUH-1RYqx...", ...],
  "data" : {
    "Sinch" : <payload>,
  },
}
```

Please refer to Google’s [Google Cloud Messaging for Android](http://developer.android.com/google/gcm/index.html) for more information.

#### On the callee side

As a prerequisite, offline calling and messaging must be enabled on the receiver’s side (see [Push Notifications sent via your application server](#push-notifications-sent-via-your-application-server)).

When the application receives a push notification from the Google Cloud Messaging Service, the application should extract the Sinch-specific payload from the push notification, and forwarding it to the Sinch client using the method `relayRemotePushNotificationPayload`.

```java
protected void onMessage(final Context context, final Intent intent) {
    String sinchPayload = intent.getStringExtra("Sinch");

    sinchClient.relayRemotePushNotificationPayload(sinchPayload);
}
```

## Glossary

This glossary defines some of the domain specific terms used throughout this document.

| Term                 | Explanation                                                                                                                                                       |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Application          | The mobile application running on iOS and/or Android. A partner can have more than one application.                                                               |
| Application Instance | One installation of the application on a single device.                                                                                                           |
| Application Key      | A key generated by Sinch. The key is unique to the application. A key looks like `196087a1-e815-4bc4-8984-60d8d8a 43f1d` (lowercase hexadecimal formatted GUID).  |
| Application Secret   | A string generated by Sinch. The secret is used to verify the application. A secret looks like `oYdgGRXoxEuJhGDY2KQ/HQ==` (Base64-encoded string representation). |
| Callee               | The person receiving a call.                                                                                                                                      |
| Caller               | The person making a call.                                                                                                                                         |
| User                 | A user of the mobile application. The actual person holding the mobile device.                                                                                    |
| User Identity        | Identity of a user in the application domain. Can be any string, for instance a user name, user id, phone number or email address.                                |
| Active Connection    | A socket connection for signaling purposes where incoming calls are received.                                                                                     |

## Third party libraries and copyright notices

All Third Party Libraries and Copyright notices can be found under this [link](http://www.sinch.com/legal/third-party-licenses/).
