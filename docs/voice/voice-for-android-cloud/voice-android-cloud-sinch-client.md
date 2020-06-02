---
title: Sinch Client
excerpt: >-
  The Sinch SDK is a product that makes adding voice and video calling to mobile
  apps easy. Continue reading this step-by-step guide now.
hidden: false
next:
  pages:
    - voice-android-cloud-calling
---

The _SinchClient_ is the Sinch SDK entry point. It is used to configure the user’s and device’s capabilities, as well as to provide access to feature classes such as the _CallClient_, _AudioController_ and _VideoController_.

## Create the _SinchClient_

Set up the client's listener (_SinchClientListener_, see [Reference](reference\com\sinch\android\rtc\SinchClientListener.html) documentation).

```java
// Instantiate a SinchClient using the SinchClientBuilder.
android.content.Context context = this.getApplicationContext();
SinchClient sinchClient = Sinch.getSinchClientBuilder().context(context)
                                                  .applicationKey("<application key>")
                                                  .environmentHost("ocra.api.sinch.com")
                                                  .userId("<user id>")
                                                  .build();

sinchClient.addSinchClientListener(sinchClientListener);
```

The _Application Key_ is obtained from the Sinch Developer Dashboard. See [Production Environments](doc:voice-android-cloud-miscellaneous#production-environments) for valid values for _environmentHost_. The User ID should uniquely identify the user on the particular device.

_Note:_ All listener callbacks emitted from the Sinch SDK are invoked on the same thread that the call to `SinchClientBuilder.build` is made on. If the invoking thread is _not_ the main-thread, it needs to have an associated `Looper`.

## Specify capabilitiesx

The SinchClient can be configured to enable or disable certain functionality. Please see the [Reference](reference\index.html?com\sinch\android\rtc\SinchClient.html) for a comprehensive description of each capability.

The following example shows how to set up the client with voice calling enabled.

```java
// Specify the client capabilities.
sinchClient.setSupportManagedPush(true);
// or
sinchClient.setSupportActiveConnectionInBackground(true);
sinchClient.startListeningOnActiveConnection()
```

Calling `startListeningOnActiveConnection` allows your application to receive incoming calls and messages without using push notifications.

> **Note**
>
> If the application is meant to only make outgoing calls but not receive incoming calls, don’t call `startListeningOnActiveConnection` or `setSupportManagedPush`. Outgoing calls can be made after calling the start method.

> **IMPORTANT**
>
> Enable [Managed Push](doc:voice-android-cloud-push-notifications) to be able to receive incoming calls via push notifications even when the application is closed or in background. Listening on an active connection in the background service is not possible due to new Android 9 requirements for such services, and the execution of such services is not guaranteed - the Android OS can 'kill' them at any time.

## Start the Sinch client

Before starting the client, add a client listener (see [Reference](reference\index.html?com\sinch\android\rtc\SinchClientListener.html) documentation):

```java
sinchClient.addSinchClientListener(new SinchClientListener() {

    public void onClientStarted(SinchClient client) { }

    public void onClientFailed(SinchClient client, SinchError error) { }

    public void onRegistrationCredentialsRequired(SinchClient client, ClientRegistration registrationCallback) { }

    public void onLogMessage(int level, String area, String message) { }
});

sinchClient.start();
```

### Authorizing the Client / User

When the _SinchClient_ is started with a given _User ID_ it is required to provide an authorization token to register towards the _Sinch backend_. To authorize a client, implement `SinchClientListener.onRegistrationCredentialsRequired()` and provide a token (a [JSON Web Token](https://jwt.io/)) that is cryptographically signed with the _Application Secret_. The sample applications included in the Sinch SDK includes a class `JWT` that describes how to create the _JWT_ and sign it with the _Application Secret_.

```java
class MySinchClientListener implements SinchClientListener {
  @Override
        ...

        // The most secure way is to obtain the credentials is from the backend,
        // since storing the Application Secret in the client app is not safe.
        // Following code demonstrates how the JWT that serves as credential should be created,
        // provided the Application Key (APP_KEY), Application Secret (APP_SECRET) and User ID.

        // NB: JWT.create() should run on your backend, and return either valid JWT or signal that
        // user can't be registered.
        // In the first case, register user with Sinch using aquired JWT via clientRegistration.register(...).
        // In the latter - report failure by calling clientRegistration.registerFailed()

        @Override
        public void onRegistrationCredentialsRequired(SinchClient client,
                ClientRegistration clientRegistration) {
            clientRegistration.register(JWT.create(APP_KEY, APP_SECRET, client.getLocalUserId()));
        }
}
```

Look for specifics in [Application Authentication](doc:voice-android-cloud-application-authentication)

> **IMPORTANT**
>
> Do not store _Application Secret_ in the application and neither use `JWT` helper class in production, they are present in sample applications to demonstrate registration flow and provide a reference of how the signing of the registration token should be done. Implement the required functionality on your backend and fetch signed registration token when required.

### Registering the Client / User via UserController API

You can also register a user towards the _Sinch backend_ via [UserController API](doc:voice-android-cloud-user-controller). This lightweight component provides a way to register the user without starting the _SinchClient_. You can also register push token for _Managed Push_ to receive incoming calls even when the application is closed/in background. The _UserController_ uses the very same authentication scheme as the _SinchClient_ based on the signed JWT registration token that you provide in response to _onRegistrationCredentialsRequired()_ method of [UserRegistrationCallback](reference\com\sinch\android\rtc\UserRegistrationCallback.html). The _UserController_ provides better control over the registration process than the _SinchClient_ by providing callbacks for each step of the registration.

### Terminate the Sinch client

When the app is done using the SinchClient, it should be stopped. If the client is currently listening for incoming events, it needs to stop listening as well. After `terminateGracefully()` is called, any object retrieved directly from the client object (that is, `CallClient`, `AudioController` and `VideoController`) is considered invalid.

Terminating the client:

```java
sinchClient.stopListeningOnActiveConnection();
sinchClient.terminateGracefully();
```
