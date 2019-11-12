# ADDING PSD2 TO YOUR FLUTTER APP

With the recent success and trend in the Flutter SDK, it is no suprise that a good number of mobile developers will consider using Flutter as their preffered choice for cross-platform mobile app development. 

This article will provide developers with details about adding PSD2 security to their App using the Okay Flutter plugin.
For more information about [PSD2](https://okaythis.com/blog/psd2-explained-and-why-you-should-care) please see this [article](https://okaythis.com/blog/psd2-explained-and-why-you-should-care).

## Setting up Okay
In order to use this plugin you also need to setup Firebase messaging for Flutter. Check this [documentation](https://pub.dev/packages/firebase_messaging) on adding Firebase messaging to your Flutter app.

We start off by adding Okay dependency to our flutter app.

You can get the Okay plugin from this github [repository](https://github.com/Okaythis/FlutterOkaySDK). Add the module to your project folder. Then reference the folder in 

```yaml
dependencies:
  flutter:
    sdk: flutter

  # The following adds the Cupertino Icons font to your application.
  # Use with the CupertinoIcons class for iOS style icons.
  cupertino_icons: ^0.1.2
  okaythis_flutter_plugin:
    path: <PATH_TO_OKAY_PLUGIN>/okaythis_flutter_plugin
```

## Using the Flutter Plugin

Import the Okay plugin into your flutter app like so.

```Dart
import "package:okaythis_flutter_plugin/okaythis_flutter_plugin.dart";
```

## Configuring Okay Callbacks

We are going to add callback handlers to listen for Okay events (i.e Linking, UnLinking, Authorization and Enrollment events). We wrap this in a method called `_configOkay()`

```Dart
  void _configOkay() {
    OkaythisFlutterPlugin.config(
        host: "https://demostand.okaythis.com",
        onEnrollmentHandler: (bool enrollmentStatus) async {
          print("Enrollment Status: ${linkingStatus ? "Success" : "Failed"}");
        },
        onLinkingHandler: (bool linkingStatus) async {
          print("Linking Status: ${linkingStatus ? "Success" : "Failed"}");
        },
        onUnLinkingHandler: (bool unlinkingStatus) async {
          print("Unlinking Status: ${unlinkingStatus ? "Success" : "Failed"}");
        },
        onAuthorizationHandler: (bool authStatus) async {
          print("Auth Status: ${authStatus ? "Success" : "Failed"}");
        });
  }

```

You will call this method from your `initState` method in you widget to properly setup the plugin. To complete this setup on Android devices we will need to call one more method, the `OkaythisFlutterPlugin.init()` just after calling the `OkaythisFlutterPlugin.config()` method.

The Android platform requires us to request certain permissions from the user in order to complete the setup process. The Okay plugin comes with a convenience method `OkaythisFlutterPlugin.requestRequiredPermissions()` that allows us to fetch all require permissions. You can then use this array of permissions to prompt the about granting them.

```Dart

class _MyHomePageState extends State<MyHomePage> {

  @override
  void initState() {
    super.initState();

    _configOkay();
    // Request permissions
    if (Platform.isAndroid) {
      _requestPermission();
      _initPsa();
    }
  }

  void _configOkay() {
    OkaythisFlutterPlugin.config(
        host: "https://demostand.okaythis.com",
        onEnrollmentHandler: (bool enrollmentStatus) async {
          print("Enrollment Status: ${linkingStatus ? "Success" : "Failed"}");
        },
        onLinkingHandler: (bool linkingStatus) async {
          print("Linking Status: ${linkingStatus ? "Success" : "Failed"}");
        },
        onUnLinkingHandler: (bool unlinkingStatus) async {
          print("Unlinking Status: ${unlinkingStatus ? "Success" : "Failed"}");
        },
        onAuthorizationHandler: (bool authStatus) async {
          print("Auth Status: ${authStatus ? "Success" : "Failed"}");
        });
  }

  // fetch all permissions required by Okay
  void _requestPermission() async {
    print(await OkaythisFlutterPlugin.requestRequiredPermissions());
  }
}
```

We can now proceed to enrolling a new user using the plugin.

# User Enrollment with Okay
Once we are all set up we can now enroll a our user using Okay. The Flutter Okay plugin comes with a method called `OkaythisFlutterPlugin.startEnrollment(String appPns, String pubPss, String installationId)` that allows us to initiate the enrollment process like so. 

```Dart

  void _startEnrollment(String appPns, String pubPss, String installationId) async {
    await OkaythisFlutterPlugin.startEnrollment(appPns, pubPss, installationId);
  }

```

It takes three arguments `appPns`, `pubPss` and `installationId`. The `appPns` is the token we got from Firebase, our `pubPss` is a token we get from Okay server and installationId is an **ID** we get from Okay servers. The tokens for Android and iOS are different, so in order to retrieve the correct token for your platform please visit this [page](https://github.com/Okaythis/okay-example/wiki/Mobile-Client-Settings) for more info.

We add these tokens to the top of our widgets

```Dart
// Tokens
  String _pubPss = Platform.isAndroid ? 
      "MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAxgyacF1NNWTA6rzCrtK60se9fVpTPe3HiDjHB7MybJvNdJZIgZbE9k3gQ6cdEYgTOSG823hkJCVHZrcf0/AK7G8Xf/rjhWxccOEXFTg4TQwmhbwys+sY/DmGR8nytlNVbha1DV/qOGcqAkmn9SrqW76KK+EdQFpbiOzw7RRWZuizwY3BqRfQRokr0UBJrJrizbT9ZxiVqGBwUDBQrSpsj3RUuoj90py1E88ExyaHui+jbXNITaPBUFJjbas5OOnSLVz6GrBPOD+x0HozAoYuBdoztPRxpjoNIYvgJ72wZ3kOAVPAFb48UROL7sqK2P/jwhdd02p/MDBZpMl/+BG+qQIDAQAB"
      : "MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAxgyacF1NNWTA6rzCrtK60se9fVpTPe3HiDjHB7MybJvNdJZIgZbE9k3gQ6cdEYgTOSG823hkJCVHZrcf0/AK7G8Xf/rjhWxccOEXFTg4TQwmhbwys+sY/DmGR8nytlNVbha1DV/qOGcqAkmn9SrqW76KK+EdQFpbiOzw7RRWZuizwY3BqRfQRokr0UBJrJrizbT9ZxiVqGBwUDBQrSpsj3RUuoj90py1E88ExyaHui+jbXNITaPBUFJjbas5OOnSLVz6GrBPOD+x0HozAoYuBdoztPRxpjoNIYvgJ72wZ3kOAVPAFb48UROL7sqK2P/jwhdd02p/MDBZpMl/+BG+qQIDAQAB";
  // Installation IDs
  String INSTALLATION_ID = Platform.isAndroid ? "9990" : "9980";
```

If our enrollment was successful Okay will trigger the `onEnrollmentHandler` we registered on our config earlier and prints to console this text `"Enrollment Status: Success`

## Linking a Tenant with Okay
Okay provides a method for linking a tenant called `OkaythisFlutterPlugin.linkTenant(linkingCode)`. This method takes a `linkingCode` provided by Okay servers, that links a user to a particular tenant.

``` Dart
void _linkTenant(String linkingCode) {
  OkaythisFlutterPlugin.linkTenant(linkingCode);
}
```

If our linking was successful Okay will trigger our `onLinkingHandler` we registered earlier with the linking status.

## Authorization with Okay

Okay is a PSD2 Compliant Strong Customer Authentication service and provides a method to handle authorization from your Flutter app with the `OkaythisFlutterPlugin.startAuthorization(int sessionId, String appPns,  Map<String, dynamic> pageTheme)` method.

```Dart

void _startAuthorization(int sessionId, String appPns,  Map<String, dynamic> pageTheme) {
  OkaythisFlutterPlugin.startAuthorization(sessionId, _appPns, pageTheme);
}
```

The `sessionId` is a numeric ID that is generated by Okay Servers once an authorization request has been sent to Okay, It send this sessionId as a push notification to your mobile app. The `appPns` is your token generated by Firebase and the `pageTheme` is a map of color value-key pairs that allows tenants to customize the look and feel of the okay authorization/authentication page.

If authorization was successful, your `onAuthorizationHandler` will be called with the status of the authorization.

## Unlinking a Tenant

If a user no longer wishes to be linked to a tenant, the user can be unlinked from that tenant. Okay provides a method `OkaythisFlutterPlugin.unlinkTenant(int tenantId)` that takes as input the `tenantId` of the tenant that is to be unlinked.

```Dart
void _unlinkTenant(int tenantId) {
  OkaythisFlutterPlugin.linkTenant(tenantId);
}
```

If unlinking was successful our `onUnLinkingHandler` will be called with status of our tenant unlinking.