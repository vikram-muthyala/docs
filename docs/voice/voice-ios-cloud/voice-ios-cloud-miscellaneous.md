---
title: Miscellaneous
excerpt: ''
hidden: false
---

## Minimum requirements

We officially support the two latest major iOS versions. You can try older versions but there are no guarantees it will work as expected.

_Note:_ The Sinch SDK library uses Automatic Reference Counting (ARC). However, it can still be used in non-ARC projects.

## Note on Sinch.framework file size vs. linked size

The _Sinch.framework_ file includes a FAT-binary containing the architectures _armv7_, _arm64_, _i386_ and _x86_64_. When linking an application target against the _Sinch.framework_ targeting an iOS device, it will add a approximately 6Mb per _arm_ slice.

**Example**: Assuming linking _armv7_ and _arm64_ into the final application, it would add approximately 12Mb to the application.

## Restrictions on User IDs

User IDs can only contain characters in the _printable ASCII character set_. That is:

```text
!"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefghijklmnopqrstuvwxyz{|}~
```

User IDs **must not** be longer than **40** characters.

## Encryption export regulations

Please check the Summary of U.S. Export Controls Applicable to Commercial Encryption Products and ensure that the application is registered for the Encryption Regulations, if applicable. It can be found under this [link](http://www.sinch.com/).

## Statistics

The Sinch SDK client uploads statistics to the Sinch servers at the end of a call, a call failure, or similar event. The statistics are used for monitoring of network status, call quality, and other aspects regarding the general quality of the service.

Some of the information is not anonymous and may be associated with the User ID call participants.

The statistics upload is done by the client in the background.

## Linking against the C++ standard library

Since Sinch SDK version 3.4.0, it is required to link against _libc++_. Though if your application is also dependent on _libstdc++_ (which is now considered deprecated by Apple for use on iOS), you can actually link against both _libc++_ and _libstdc++_ by passing the following linker flags:

- Other Linker Flags -\> `-ObjC -Xlinker -lc++ -Xlinker -lstdc++`

## Request user permission for using the microphone

Since iOS 7, additional user privacy constraints are enforced which requires the application to be granted permission to use the device microphone. Unless the application has explicitly requested permission to use the microphone, the user is shown a dialog the first time the microphone is activated.

In the context of the Sinch SDK, this occurs once the first call is established _unless_ the application has been granted permission earlier. We **strongly recommend** you explicitly request permission to use the microphone in your application at an appropriate time such as when the user first sets up Sinch. You should not rely on the permission dialog shown when the first Sinch call is established as this will create an awkward user experience.

By explicitly requesting permission using the methods available in the iOS SDK, the application has more control over when the dialog is shown to the user. This results in a better user experience.

Starting with iOS 10.0, apps that access any of the device’s microphones must declare their intent to do so. This is done by including the NSMicrophoneUsageDescription key and a corresponding purpose string in your app’s Info.plist. When the system prompts the user to allow access, the purpose string is displayed as part of the alert. If an application attempts to access any of the device’s microphones without a corresponding purpose string, the app will exit.

Please see the [Apple iOS SDK documentation on the class AVAudioSession](http://developer.apple.com/library/ios/#documentation/AVFoundation/Reference/AVAudioSession_ClassReference/Reference Reference.html) for details on how to request permission to use the microphone.

## Request user permission for using the camera

The same rule applies to request user permission for using the camera. In iOS, the user must explicitly grant your app permission to access device cameras or microphones for photo, video, or audio capture. Your app must provide an explanation for its use of capture devices using the NSCameraUsageDescription and NSMicrophoneUsageDescription Info.plist keys; iOS displays this explanation when initially asking the user for permission, and thereafter in the Settings app.

Please see the [Apple iOS SDK documentation on the class AVCaptureDevice](https://developer.apple.com/documentation/avfoundation/avcapturedevice?language=objc) for details on how to request permission to use the camera.

## App Extensions

_App Extensions_ is a feature introduced in iOS 8. App extensions are compiled into executables that are separate from the main application executable. The Sinch SDK are using parts of the iOS SDK APIs that are unavailable to app extensions, thus it’s not supported to use the Sinch SDK in an app extension.

## Xcode and Bitcode intermediate representation

The Sinch SDK supports Bitcode intermediate representation.

## Deprecated features and APIs

### Active Connection in Background

Apple has since iOS 10 discontinued support for maintaining a _VoIP_ control connection alive via `-[UIApplication setKeepAliveTimeout:handler:]`. Attempting to use this method on an iOS device running iOS 10 results in the following warning log: `Legacy VoIP background mode is deprecated and no longer supported`. The Sinch feature _Active connection in background_ was using the keep alive handler API and is as a consequence no longer supported on iOS. It is recommended to use [VoIP Push Notifications](doc:voice-ios-cloud-local-and-remote-push-notifications) to achieve the equivalent functionality.
