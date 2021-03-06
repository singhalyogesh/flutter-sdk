# truecaller_sdk

Flutter plugin that uses [Truecaller's Android SDK](https://docs.truecaller.com/truecaller-sdk/) to provide mobile number verification service to verify Truecaller users. This plugin currently supports only Android and can be used to verify only the Truecaller users at the moment. Since these users have already verified mobile number, verification via Truecaller SDK enables you to quickly verify/ signup/login your users, basis their mobile number - without the need of any SMS based OTP, and at the time same capture their mapped user profile.

## Steps to integrate

### 1. Update your pubspec.yaml
Include the latest truecaller_sdk in your `pubspec.yaml`
```yaml
dependencies:
  ...
  truecaller_sdk: ^0.0.1
  ...
```
### 2. Generate App key and add it to AndroidManifest.xml:
* Register [here](https://developer.truecaller.com/sign-up) for truecaller's developer account.
* Refer to the [official documentation](https://docs.truecaller.com/truecaller-sdk/android/generating-app-key) for generating app key.
* Open your `AndroidManifest.xml` and add a `meta-data` element to the `application` element with your app key like this - 
```xml
<application>  
...  
<activity>  
.. </activity>

<meta-data android:name="com.truecaller.android.sdk.PartnerKey" android:value="PASTE_YOUR_PARTNER_KEY_HERE"/>  
...  
</application>  
```

### 3. Make changes to `MainActivity.kt`:
* SDK requires the use of a FragmentActivity as opposed to Activity, so extend your `MainActivity.kt` with `FlutterFragmentActivity`.
* Override function `configureFlutterEngine(flutterEngine: FlutterEngine)` in your `MainActivity.kt` like this -
```kotlin
class MainActivity: FlutterFragmentActivity() {
    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        GeneratedPluginRegistrant.registerWith(flutterEngine)
    }
}
```

## Example

```dart
// Import package
import 'package:truecaller_sdk/truecaller_sdk.dart';

//Step 1: Initialize the SDK
TruecallerSdk.initializeSDK();

//Step 2: Check if SDK is usable
bool isUsable = await TruecallerSdk.isUsable;

//Step 3: If isUsable is true, you can call getProfile to show consent screen to verify user's number
isUsable ? TruecallerSdk.getProfile : print("***Not usable***");

//OR you can also replace Step 2 and Step 3 directly with this  
TruecallerSdk.isUsable.then((isUsable) {
 isUsable ? TruecallerSdk.getProfile : print("***Not usable***");
});
                   
//Step 4: Be informed about the getProfile callback result(success, failure, verification)
StreamSubscription streamSubscription = TruecallerSdk.getProfileStreamData.listen((truecallerUserCallback) {
  switch (truecallerUserCallback.result) {
    case TruecallerUserCallbackResult.success:
      print("First Name: ${truecallerUserCallback.profile.firstName}");
      print("Last Name: ${truecallerUserCallback.profile.lastName}");
      break;
    case TruecallerUserCallbackResult.failure:
      print("Error code : ${truecallerUserCallback.error.code}");
      break;
    case TruecallerUserCallbackResult.verification:
      print("Verification Required!!");
      break;
    default:
      print("Invalid result");
  }
});

//Step 5: Dispose streamSubscription
@override
  void dispose() {
    super.dispose();
    if (streamSubscription != null) {
      streamSubscription.cancel();
    }
  }
```

##### NOTE: For details on different kind of errorCodes refer [here](https://docs.truecaller.com/truecaller-sdk/android/integrating-with-your-app/handling-error-scenarios)

## Customization Options

### Language
To customise the profile consent screen in any of the supported Indian languages, add the following line before calling `TruecallerSdk.getProfile()` like this -
```dart
/// initialize the SDK and check isUsable first before calling this method
/// Default value is "en" i.e English
TruecallerSdk.setLocale("hi") // this sets the language to Hindi
```

### Dark Theme
You can also set the Dark Theme for consent screen by adding the following line before calling `TruecallerSdk.getProfile()` like this -
```dart
/// initialize the SDK and check isUsable first before calling this method
TruecallerSdk.setDarkTheme 
```
##### Note: Dark Theme is not applicable for `TruecallerSdkScope.CONSENT_MODE_BOTTOMSHEET`

### Consent screen UI
You can customize the consent screen UI using the options available in class `TruecallerSdkScope` under `scope_options.dart` and pass them while initializing the SDK.

```dart
  /// [sdkOptions] determines whether you want to use the SDK for verifying - 
  /// 1. [TruecallerSdkScope.SDK_OPTION_WITHOUT_OTP] i.e only Truecaller users
  /// 2. [TruecallerSdkScope.SDK_OPTION_WITH_OTP] i.e both Truecaller and Non-truecaller users
  ///
  /// NOTE: As of truecaller_sdk 0.0.1, only
  /// [TruecallerSdkScope.SDK_OPTION_WITHOUT_OTP] is supported
  ///
  /// [consentMode] determines which kind of consent screen you want to show to the user.
  /// [consentTitleOptions] is applicable only for [TruecallerSdkScope.CONSENT_MODE_POPUP]
  /// and [TruecallerSdkScope.CONSENT_MODE_FULLSCREEN] and it sets the title prefix
  /// [footerType] determines the footer button text. You can set it to
  /// [TruecallerSdkScope.FOOTER_TYPE_NONE] if you don't want to show any footer button
  /// There are some customization options applicable only for [TruecallerSdkScope.CONSENT_MODE_BOTTOMSHEET]
  /// which are following -
  /// [loginTextPrefix] determines prefix text in login sentence
  /// [loginTextSuffix] determines suffix text in login sentence
  /// [ctaTextPrefix] determines prefix text in login button
  /// [privacyPolicyUrl] to set your own privacy policy url
  /// [termsOfServiceUrl] to set your own terms of service url
  /// [buttonShapeOptions] to set login button shape
  /// [buttonColor] to set login button color
  /// [buttonTextColor] to set login button text color
  static initializeSDK(
          {int sdkOptions: TruecallerSdkScope.SDK_OPTION_WITHOUT_OTP,
          int consentMode: TruecallerSdkScope.CONSENT_MODE_BOTTOMSHEET,
          int consentTitleOptions: TruecallerSdkScope.SDK_CONSENT_TITLE_GET_STARTED,
          int footerType: TruecallerSdkScope.FOOTER_TYPE_SKIP,
          int loginTextPrefix: TruecallerSdkScope.LOGIN_TEXT_PREFIX_TO_GET_STARTED,
          int loginTextSuffix: TruecallerSdkScope.LOGIN_TEXT_SUFFIX_PLEASE_LOGIN,
          int ctaTextPrefix: TruecallerSdkScope.CTA_TEXT_PREFIX_USE,
          String privacyPolicyUrl: "",
          String termsOfServiceUrl: "",
          int buttonShapeOptions: TruecallerSdkScope.BUTTON_SHAPE_ROUNDED,
          int buttonColor,
          int buttonTextColor})
```

By default, `initializeSDK()` has default argument values set as above, so if you don't pass any explicit values to it, it will initialize the SDK with these scope options.

##### Note: For list of supported locales and details on different kind of customization refer [here](https://docs.truecaller.com/truecaller-sdk/android/integrating-with-your-app/customisation-1)