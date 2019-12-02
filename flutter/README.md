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

## Page Theme

Okay allows developers to customize the UI of their apps to suit their desired look and feel to match the currrent theme of their app.

Okay Flutter plugin provides a class called **PageTheme**. The class provides all customizable color properties that are available for development.

The class is presented below

```dart
class PageTheme {

  static const String ACTION_BAR_BACKGROUND_COLOR = "actionBarBackgroundColor";
  static const String ACTION_BAR_TEXT_COLOR = "actionBarTextColor";
  static const String BUTTON_TEXT_COLOR = "buttonTextColor";
  static const String BUTTON_BACKGROUND_COLOR = "buttonBackgroundColor";
  static const String SCREEN_BACKGROUND_COLOR = "screenBackgroundColor";
  static const String NAME_TEXT_COLOR = "nameTextColor";
  static const String TITLE_TEXT_COLOR = "titleTextColor";
  static const String MESSAGE_TEXT_COLOR = "messageTextColor";
  static const String ACTION_BAR_LOGO_PATH = "actionBarLogoPath";
  static const String ACTION_BAR_TITLE = "actionBarTitle";
  static const String INPUT_TEXT_COLOR = "inputTextColor";
  static const String INPUT_SELECTION_COLOR = "inputSelectionColor";
  static const String INPUT_ERROR_COLOR = "inputErrorColor";
  static const String INPUT_DEFAULT_COLOR = "inputDefaultColor";
  static const String QUESTION_MARK_COLOR = "questionMarkColor";
  static const String TRANSACTION_TYPE_TEXT_COLOR = "transactionTypeTextColor";
  static const String INFO_SECTION_TITLE_COLOR = "infoSectionTitleColor";
  static const String INFO_SECTION_VALUE_COLOR = "infoSectionValueColor";
  static const String FROM_TEXT_COLOR = "fromTextColor";
  static const String AUTH_INFO_BACKGROUND_COLOR = "authInfoBackgroundColor";
  static const String SHOW_DETAILS_TEXT_COLOR = "showDetailsTextColor";
  static const String CONFIRM_BUTTON_BACKGROUND_COLOR = "confirmButtonBackgroundColor";
  static const String CONFIRM_BUTTON_TEXT_COLOR = "confirmButtonTextColor";
  static const String CANCEL_BUTTON_BACKGROUND_COLOR = "cancelButtonBackgroundColor";
  static const String CANCEL_BUTTON_TEXT_COLOR = "cancelButtonTextColor";
  static const String AUTH_CONFIRMATION_BACKGROUND_COLOR = "authConfirmationBackgroundColor";
  static const String AUTH_CONFIRMATION_TITLE_COLOR = "authConfirmationTitleColor";
  static const String AUTH_CONFIRMATION_MESSAGE_COLOR = "authConfirmationMessageColor";
  static const String AUTH_CONFIRMATION_THUMB_COLOR = "authConfirmationThumbColor";
  static const String AUTH_CONFIRMATION_APOSTROPHE_COLOR = "authConfirmationApostropheColor";
  static const String AUTH_CONFIRMATION_BUTTON_BACKGROUND_COLOR = "authConfirmationButtonBackgroundColor";
  static const String AUTH_CONFIRMATION_BUTTON_TEXT_COLOR = "authConfirmationButtonTextColor";
  static const String AUTH_CANCELLATION_BACKGROUND_COLOR = "authCancellationBackgroundColor";
  static const String AUTH_CANCELLATION_TITLE_COLOR = "authCancellationTitleColor";
  static const String AUTH_CANCELLATION_MESSAGE_COLOR = "authCancellationMessageColor";
  static const String AUTH_CANCELLATION_THUMB_COLOR = "authCancellationThumbColor";
  static const String AUTH_CANCELLATION_APOSTROPHE_COLOR = "authCancellationApostropheColor";
  static const String AUTH_CANCELLATION_BUTTON_BACKGROUND_COLOR = "authCancellationButtonBackgroundColor";
  static const String AUTH_CANCELLATION_BUTTON_TEXT_COLOR = "authCancellationButtonTextColor";
  static const String PIN_TITLE_TEXT_COLOR = "pinTitleTextColor";
  static const String PIN_VALUE_TEXT_COLOR = "pinValueTextColor";
  static const String PIN_NUMBER_BUTTON_TEXT_COLOR = "pinNumberButtonTextColor";
  static const String PIN_NUMBER_BUTTON_BACKGROUND_COLOR = "pinNumberButtonBackgroundColor";
  static const String PIN_REMOVE_BUTTON_TEXT_COLOR = "pinRemoveButtonTextColor";
  static const String PIN_REMOVE_BUTTON_BACKGROUND_COLOR = "pinRemoveButtonBackgroundColor";
}
```

### Usage

Using the Okay PageTheme class is pretty easy. You will find below a typical illustration on how to use this class.

```dart
import 'package:okaythis_flutter_plugin/okaythis_flutter_plugin.dart' ;

class BaseTheme {

// create a Map<String, String> that holds all
// key-value pairs for your theme
  static Map<String, String> getTheme () {
    return {
      PageTheme.ACTION_BAR_BACKGROUND_COLOR: ThemeColors.primaryDarkColor,
      PageTheme.ACTION_BAR_TEXT_COLOR: ThemeColors.primaryTextColor,
      PageTheme.SCREEN_BACKGROUND_COLOR: ThemeColors.primaryColor,
      PageTheme.BUTTON_BACKGROUND_COLOR: ThemeColors.secondaryColor,
      PageTheme.BUTTON_TEXT_COLOR: ThemeColors.secondaryTextColor,


      PageTheme.PIN_NUMBER_BUTTON_TEXT_COLOR: ThemeColors.secondaryColor,
      PageTheme.PIN_NUMBER_BUTTON_BACKGROUND_COLOR: ThemeColors.secondaryLightColor,
      PageTheme.PIN_REMOVE_BUTTON_BACKGROUND_COLOR: ThemeColors.secondaryLightColor,
      PageTheme.PIN_REMOVE_BUTTON_TEXT_COLOR: ThemeColors.secondaryTextColor,
      PageTheme.PIN_TITLE_TEXT_COLOR: ThemeColors.primaryTextColor,
      PageTheme.PIN_VALUE_TEXT_COLOR: ThemeColors.primaryTextColor,

      PageTheme.TITLE_TEXT_COLOR: ThemeColors.primaryTextColor,
      PageTheme.QUESTION_MARK_COLOR: ThemeColors.primaryLightColor,
      PageTheme.TRANSACTION_TYPE_TEXT_COLOR: ThemeColors.primaryTextColor,

      PageTheme.AUTH_INFO_BACKGROUND_COLOR: ThemeColors.transactionInfoBackground,
      PageTheme.INFO_SECTION_TITLE_COLOR: ThemeColors.secondaryLightColor,
      PageTheme.INFO_SECTION_VALUE_COLOR: ThemeColors.secondaryTextColor,
      PageTheme.FROM_TEXT_COLOR: ThemeColors.secondaryTextColor,
      PageTheme.MESSAGE_TEXT_COLOR: ThemeColors.secondaryTextColor,

      PageTheme.CONFIRM_BUTTON_BACKGROUND_COLOR: ThemeColors.secondaryLightColor,
      PageTheme.CONFIRM_BUTTON_TEXT_COLOR: ThemeColors.secondaryTextColor,
      PageTheme.CANCEL_BUTTON_BACKGROUND_COLOR: ThemeColors.primaryLightColor,
      PageTheme.CANCEL_BUTTON_TEXT_COLOR: ThemeColors.primaryTextColor,


      PageTheme.AUTH_CONFIRMATION_BUTTON_BACKGROUND_COLOR: ThemeColors.secondaryColor,
      PageTheme.AUTH_CONFIRMATION_BUTTON_TEXT_COLOR: ThemeColors.secondaryTextColor,
      PageTheme.AUTH_CANCELLATION_BUTTON_BACKGROUND_COLOR: ThemeColors.primaryColor,
      PageTheme.AUTH_CONFIRMATION_BUTTON_TEXT_COLOR: ThemeColors.primaryTextColor,

      PageTheme.NAME_TEXT_COLOR: ThemeColors.secondaryTextColor,

      PageTheme.BUTTON_BACKGROUND_COLOR: ThemeColors.primaryLightColor,
      PageTheme.BUTTON_TEXT_COLOR: ThemeColors.primaryTextColor,
      PageTheme.INPUT_TEXT_COLOR: ThemeColors.secondaryTextColor,
      PageTheme.INPUT_SELECTION_COLOR: ThemeColors.green,
      PageTheme.INPUT_ERROR_COLOR: ThemeColors.red,
      PageTheme.INPUT_DEFAULT_COLOR: ThemeColors.gray,
    };
  }
}

class ThemeColors {
  static final String primaryColor = '#1976d2';
  static final String primaryLightColor = '#63a4ff';
  static final String primaryDarkColor = '#004ba0';
  static final String secondaryColor = '#f9a825';
  static final String secondaryLightColor = '#ffd95a';
  static final String secondaryDarkColor = '#c17900';
  static final String pinControlsBackground = '#cfd8dc';
  static final String transactionInfoBackground = white;
  static final String primaryTextColor = white;
  static final String secondaryTextColor = black;

  static final String red = '#ff0000';
  static final String gray = '#c5c5c5';
  static final String green = '#00ff00';
  static final String white = '#ffffff';
  static final String black = '#000000';
}
```

You can now use the class to customize your authorization screen like so

```dart
// using the theme above to customize the 
// authorization screen
  OkaythisFlutterPlugin.startAuthorization(
                          int.parse(sessionId), _appPns, BaseTheme.getTheme());
```