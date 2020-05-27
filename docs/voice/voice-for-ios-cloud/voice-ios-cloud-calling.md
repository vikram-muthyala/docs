---
title: Voice Calling
excerpt: >-
  Set up voice calling with the Sinch Voice and Video SDK. The SDK supports four
  types of calls: _app-to-app_ (audio or video), _app-to-phone_, _app-to-sip_ and
  _conference_ calls.
hidden: false
next:
  pages:
    - voice-ios-cloud-video-calling
---

The Sinch SDK supports four types of calls: _app-to-app_ (audio or video), _app-to-phone_, _app-to-sip_ and _conference_ calls. The `SINCallClient` is the entry point for the calling functionality of the Sinch SDK.

Calls are placed using `SINCallClient` and events are received via `SINCallClientDelegate`. The call client (`SINCallClient`) is owned by the _Sinch client_ (`SINClient`) and accessed using `-[SINClient callClient]`.

## Setting Up an _app-to-app_ Call

Use the call client to start the call using the method `callUserWithId:`, passing the user identifier of the callee (the destination user) as an argument.

```objectivec
id<SINCallClient> callClient = [sinchClient callClient];
id<SINCall> call = [callClient callUserWithId:@"<remote user id>"];
call.delegate = self; // Assign delegate to observe call state changes
// Or for video call: id<SINCall> call = [callClient callUserVideoWithId:@"<remote user id>"];
```

A call object is returned, containing details about the participants in the call, call details such as start time, call state, possible errors, and so on.

Assuming the callee’s device is available and responsive, the delegate method `callDidProgress:` will be called. It notifies the application that the outbound call is progressing. If a progress tone should be played, this is where it should be started. We recommend that you use the available functionality provided by the Sinch SDK to play sounds such as ringtones (`SINAudioController`), see [Playing Ringtones](doc:voice-ios-cloud-playing-ringtones).

When the other party answers, the delegate method `callDidEstablish:` method will be called. Now, the users can start talking. If a progress tone has previously been initiated, it should be stopped now, in the delegate callback method.

## Setting Up an _app-to-phone_ Call

An _app-to-phone_ call is a call that is made to a phone on the regular telephone network. Setting up an _app-to-phone_ call is not much different from setting up an _app-to-app_ call.

Instead of invoking the `callUserWithId:` method, invoke the `callPhoneNumber:` method on the `SINCallClient` object. Sufficient funds must be available on the Sinch account and a valid phone number specified for the call to connect successfully. The phone number should be specified according to the E.164 number formatting (<http://en.wikipedia.org/wiki/E.164>) recommendation and should be prefixed with a ‘+’. E.g. to call the US phone number 415 555 0101, the phone number should be specified as “+14155550101”. The ‘+’ is the required prefix and the US country code ‘1’ prepended to the local subscriber number.

Placing an _app-to-phone_ call requires an account with credits; topping up credits can be done on the Account page. Credits are used each time an _app-to-phone_ call is placed and the balance history is updated after each call.

_App-to-phone_ calls can be tested by calling the following test number: _+46000000000_. When placing a call to this number, you will hear a voice prompt stating that the call has been connected, and shortly after that the call will automatically be ended.

## Setting Up an _app-to-sip_ Call

An _app-to-sip_ call is a call that is made to a SIP server. Setting up an _app-to-sip_ call is not much different from setting up an _app-to-app_ call. Instead of invoking the `callUserWithId:` method, invoke the `callSIP:` method on the `SINCallClient` object. The SIP identity follows the form of email addresses (<user@domain>), for example <Alice@SipProviderA.com>.

```objectivec
id<SINCallClient> callClient = [sinchClient callClient];
id<SINCall> call = [callClient callSIP:@"<SIP Identity>"];
```

When customized SIP headers are passed as a parameter, the headers should be prefixed with ‘x-’. If the SIP server reported any errors, the `SINCallDetails` object will provide an error with _SINErrorDomainSIP_.

## Setting Up a _conference_ Call

A _conference_ call can be made to connect a user to a conference room where multiple users can be connected at the same time. The identifier for a conference room may not be longer than 64 characters.

```objectivec
id<SINCallClient> callClient = [sinchClient callClient];
id<SINCall> call = [callClient callConferenceWithId:@"<conferenceId>"];
```

It is also possible to connect users to a conference call via the [Sinch REST API](doc:voice-rest-api-onprem-calling-api#text-to-speech).

## Handling Incoming Calls

To act on the incoming calls, implement the protocol _SINCallClientDelegate_ and assign a delegate to the call client. The call client delegate is notified using the delegate method `didReceiveIncomingCall:` as calls come in to the device.

When the delegate method is executed, the call can either be connected automatically without any user action, or it can wait for the user to press the answer or the hangup button. We recommend that ringtones are played from within the delegate callback method. See \[Playing Ringtones\]\[\] for details.

```objectivec
- (void)client:(id<SINCallClient>)client didReceiveIncomingCall:(id<SINCall>)call {
    // Start playing ringing tone
    ...

    // Assign delegate
    call.delegate = self;
```

To get events related to the call, set the call delegate. The call object contains details about participants, start time, potential error codes, and error messages.

If [VoIP push notifications](doc:voice-ios-cloud-local-and-remote-push-notifications#enabling-voip-push-notifications) is enabled, add logic for presenting a local notification if the app is in the background when receiving the call:

```objectivec
- (SINLocalNotification *)client:(id<SINClient>)client
  localNotificationForIncomingCall:(id<SINCall>)call {
    SINLocalNotification *notification = [[SINLocalNotification alloc] init];
    notification.alertAction = @"Answer";
    notification.alertBody = @"Incoming call";
    return notification;
```

> **WARNING: Important**
>
> See \[Local and Remote Push Notifications\]\[\] for further details on how to present and handle a user notification for an incoming call when the application is in the background.

### Incoming Video Call

When incoming call is a video call, the `didReceiveIncomingCall` delegate method will be executed, just like for the incoming audio call. The `SINCallDetails` object provides a `isVideoOffered` property to check whether the call offers a video track. See the \[Video calling\] section for details on how to add video views.

### Answering an Incoming Call

To answer a call, use the `answer` method on the call to accept it. If a ringtone was previously played, it should be stopped now.

User presses the answer button:

```objectivec
// User answers the call
[call answer];

// Stop playing ringing tone
```

Now, the clients on both ends establish the connection. When the call is established and the voice streams are running in both directions, the `callDidEstablish:` delegate method is called.

### Declining an Incoming Call

If the call should not be answered, use the `hangup` method on the call to decline. The caller is notified that the incoming call was denied. If a ringtone was previously played, it should be stopped now.

User presses the hangup button:

```objectivec
// User does not want to answer
[call hangup];

// Stop playing ringing tone
```

## Disconnecting a Call

When the user wants to disconnect an ongoing call, use the `hangup` method. Either user taking part in a call can disconnect it.

Hanging up a call:

```objectivec
[call hangup];
```

When either party disconnects a call, the application is notified using the call delegate method `callDidEnd:`. This allows the user interface to be updated, an alert tone to be played, or similar actions to occur.

A call can be disconnected before it has been completely established.

Hanging up a connecting call:

```objectivec
// Starting a call
id<SINCall> call = [client callUserWithId:@"<remote user id>"];

// User changed his/her mind, let’s hangup
[call hangup];
```

Handling a call that ends:

```objectivec
// SINCallDelegate implementation

- (void)callDidEnd:(id<SINCall>) call {
  // update user interface, e.g. hide the call screen.
```
