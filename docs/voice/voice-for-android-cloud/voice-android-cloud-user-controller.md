---
title: Push notifications
excerpt: ''
hidden: false
next:
  pages:
    - voice-android-cloud-application-authentication
---

### Push token registration via _UserController_ API

> **Note**
>
> UserController provides a way to register an user for incoming calls via push notifications, while providing the callback about success/failure of such registration. You can also use it to un-register push token if receiving of incoming calls is no longer desireable.

There are certain situations where it is either desirable to explicitly register push token and/or get assurance that the push token is indeed registered, e.g.:

- The application is designed to receive calls only, and thus must register push token with the Sinch backend on the very first start, while it's desirable to terminate SinchClient as soon as the registration concludes (e.g. to free resources). In this situation, the application should be notified by a specific callback on the registration result.
- The application detects that FCM push token is invalidated and should be refreshed and re-registered with Sinch backend. Here, if SinchClient is running, it would take care of re-registering of the push token itself, otherwise the application is responsible for re-registering.

Both situations should be handled with using new _UserController API_ (see [Reference](reference\com\sinch\android\rtc\UserController.html)), which can be used independently from the _SinchClient_ and does not require ???.

```java
public UserController getUserController(String userId) {
     Sinch.getUserControllerBuilder()
                .context(getApplicationContext())
                .applicationKey(applicationKey)
                .userId(userId)
                .environmentHost("ocra.api.sinch.com")
                .build();
}
```

The former situation is showcased in _LoginActivity.java_ in _sinch-rtc-sample-push_ and _sinch-rtc-sample-video-push_ sample applications. The activity implements [UserRegistrationCallback](reference\com\sinch\android\rtc\UserRegistrationCallback.html) and [PushTokenRegistrationCallback](reference\com\sinch\android\rtc\PushTokenRegistrationCallback.html) interfaces:

```java
public class LoginActivity extends BaseActivity implements SinchService.StartFailedListener, PushTokenRegistrationCallback, UserRegistrationCallback {

    private void loginClicked() {
        ...
        UserController uc = Sinch.getUserControllerBuilder()
                .context(getApplicationContext())
                .applicationKey(APP_KEY)
                .userId(mUserId)
                .environmentHost(ENVIRONMENT)
                .build();
        uc.registerUser(this, this);
    }

    @Override
    public void onUserRegistrationFailed(SinchError sinchError) {
        dismissSpinner();
        Toast.makeText(this, "Registration failed!", Toast.LENGTH_LONG).show();
    }

    @Override
    public void onUserRegistered() {
        // Instance is registered, but we'll wait for another callback, assuring that the push token is
        // registered as well, meaning we can receive incoming calls.
    }

    @Override
    public void tokenRegistrationFailed(SinchError sinchError) {
        dismissSpinner();
        Toast.makeText(this, "Push token registration failed - incoming calls can't be received!", Toast.LENGTH_LONG).show();
    }

    @Override
    public void tokenRegistered() {
        dismissSpinner();
        startClientAndOpenPlaceCallActivity();
    }

    // The most secure way is to obtain the credentials from the backend,
    // since storing the Application Secret in the app is not safe.
    // Following code demonstrates how the JWT that serves as credential should be created,
    // provided the Application Key (APP_KEY), Application Secret (APP_SECRET) and User ID.

    @Override
    public void onCredentialsRequired(ClientRegistration clientRegistration) {
        // NB: This implementation just emulates what should be an async procedure, with JWT.create() being
        // run on your backend.
        clientRegistration.register(JWT.create(APP_KEY, APP_SECRET,mUserId));
    }
}
```

User registration is a two step process, where first the step is registering _user_ (after which you can make outgoing calls using the _SinchClient_), and second is registering the push token for receiving incoming calls via FCM push notifications. Each step has correspondent _success_ and _failure_ callbacks, where you are mostly interested in the _tokenRegistered_, After receivng it, you can terminate / close the application and be sure that incoming calls will be received.

![Token-based User Registration](images\20200221-user_and_push_registration.png)

> **IMPORTANT**
>
> User Controller requires signed registration token the same way as it is required by the SinchClient for the first time. Provide it via [ClientRegistration.register()](reference\com\sinch\android\rtc\ClientRegistration.html) callback in response to _onCredentialsRequired()_.
> See more information about authentication [here](doc:voice-android-cloud-application-authentication).

When both conditions:
- SinchClient is started
- push token is registered
are met, the authentication process has finished and e.g. UI can advance.

> **Note**
>
> It is safe to close application right after receiving _tokenRegistered()_ callback - you'll keep receving incoming calls unless you _force stop_ the application or unregister the push token using _UserController_.

### Push token un-registration via _UserController_ API

When you want to _logout_ and stop receiving incoming calls via push, unregister the push token using _UserController_:

```java
    UserController uc = Sinch.getUserControllerBuilder()
        .context(getApplicationContext())
        .applicationKey("<application key>")
        .userId("<user id>")
        .environmentHost("ocra.api.sinch.com")
        .build();
    uc.unregisterPushToken();
```
