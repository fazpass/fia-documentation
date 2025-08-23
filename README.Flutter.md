# FIA Documentation (Flutter)

Documentation on how to use FIA Flutter SDK.

# Installation

Run this command in your root project:

`$ flutter pub add fia`

This will add a line like this to your package's pubspec.yaml (and run an implicit flutter pub get):

```yaml
dependencies:
  fia: ^<version>
```

Alternatively, your editor might support Dart pub get. Check the docs for your editor to learn more.

Now in your Dart code, you can use:

```dart
import 'package:fia/fia.dart';
```

# Getting Started

Before using this SDK, make sure to get the Merchant Key and Merchant App ID from Keypaz Dashboard. 
Check this [Dashboard Documentation](README.Dashboard.md#retrieve-your-merchant-key).

<details>
<summary><h2>Setup iOS</h2></summary>

In your XCode, add these capabilities in 'Signing & Capabilities':
1. App Groups (container `group.com.keypaz`)
2. iCloud (service `Key-value storage`)

![XCode Signing & Capabilities](images/xcode-signing-capabilities.png)

</details>

<details>
<summary><h2>Setup Miscall (Android)</h2></summary>

Miscall needs these two permissions:
- Manifest.permission.READ_PHONE_STATE
- Manifest.permission.READ_CALL_LOG

Add these lines in your android manifest file:
```xml
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.READ_CALL_LOG" />
```

Then request for runtime permissions like this:
<details>
<summary>Kotlin</summary>

 ```kotlin
val requiredPermissions = arrayOf(Manifest.permission.READ_PHONE_STATE, Manifest.permission.READ_CALL_LOG)
ActivityCompat.requestPermissions(this, requiredPermissions, 0)
```

</details>

</details>

<details>
<summary><h2>Setup HE (Android)</h2></summary>

Add this line in your android manifest file, in the `application` tag:
```xml
android:networkSecurityConfig="@xml/fia_network_security_rules"
```

### Example

```xml
<application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:networkSecurityConfig="@xml/fia_network_security_rules">

	<!-- Your declared activity tags, service tags etc. -->
</application>
```

<details>
<summary>Already had a network security config rules in your app?</summary>

Then this is the configuration needed for FIA:

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>

	<!-- other domain configurations... -->

	<domain-config cleartextTrafficPermitted="true">
		<domain includeSubdomains="true">verify.klikaman.online</domain>
		<domain includeSubdomains="true">api.fazpass.com</domain>
		<trust-anchors>
			<certificates src="system" />
			<certificates src="user" />
		</trust-anchors>
	</domain-config>
</network-security-config>
```
</details>

</details>

<details>
<summary><h2>Setup Magic Link (Android)</h2></summary>

Add this code in your android manifest file, inside the `application` tag:

```xml
<activity
    android:name="com.fazpass.fia.activities.magiclink.MagicLinkActivity"
    android:exported="true">
    <intent-filter android:autoVerify="true">
	<action android:name="android.intent.action.VIEW" />

	<category android:name="android.intent.category.DEFAULT" />
	<category android:name="android.intent.category.BROWSABLE" />

	<data
	    android:host="YOUR_DOMAIN"
	    android:scheme="https" />
    </intent-filter>
</activity>
```

Fill `YOUR_DOMAIN` with your website domain.

Then create a new file named `assetlinks.json` with this content:

```json
[
  {
    "relation": ["delegate_permission/common.handle_all_urls"],
    "target": {
      "namespace": "android_app",
      "package_name": "YOUR_PACKAGE_NAME",
      "sha256_cert_fingerprints": ["YOUR_SHA256_CERT_FINGERPRINT"]
    }
  }
]
```

Fill `YOUR_PACKAGE_NAME` with your app package name (example: `com.example.app`), 
`YOUR_SHA256_CERT_FINGERPRINT` with your app SHA256 certificate fingerprint.

<details>
<summary><h3>How to get your app SHA256 Certificate Fingerprint</h3></summary>

In `assetlinks.json`, sha256_cert_fingerprints is an array. You can add more than one certificate fingerprints in here.

1. Follow this [Android App Signing Documentation](https://developer.android.com/studio/publish/app-signing) up until you created a keystore
2. Run this command in your console to check your keystore (.jks or .keystore) information: `keytool -list -v -keystore MY_KEYSTORE.jks`
3. Enter your keystore password
4. Console will print out your keystore information. Copy the SHA256 certificate fingerprints value
5. Add the certificate fingerprint to the sha256_cert_fingerprints array
6. After you uploaded your app to Playstore, open [Google Play Console](https://play.google.com/console)
7. Navigate to your app > Test & Release > App Integrity > App Signing
8. Copy the SHA256 certificate fingerprints value
9. If the value is different from the first one, add the certificate fingerprint to the sha256_cert_fingerprints array

</details>

Then save the `assetlinks.json` file and serve it in your domain with this link: https://YOUR_DOMAIN.com/.well-known/assetlinks.json. Make sure:
1. It's available for public access
2. No Redirect
3. Content-Type is application/json

</details>

# Usage

First, you have to initialize the sdk once.

<details>
<summary>Dart</summary>
 
```dart
import 'package:fia/fia.dart';

// get instance
final fia = Fia();
// initialize
fia.initialize("MERCHANT_KEY", "MERCHANT_APP_ID");
```

</details>

## Request and Validate OTP

Follow these steps to request and validate your otp.

### 1. Create a public class to hold a static variable of type `OtpPromise`

<details>
<summary>Dart</summary>

```dart
class Constants {
	static OtpPromise? otpPromise;
}
```

</details>

### 2. Request for an OTP

To request for an OTP, call one of the four methods which fits the purpose of the otp:
- login()
- register()
- transaction()
- forgetPassword()

For example, we will use the register method.

<details>
<summary>Dart</summary>

```dart
final promise = await fia.otp().register("PHONE_NUMBER");
if (promise.hasException) {
  final error = promise.exception;
  // handle failed OTP request here...
  return;
}

Constants.otpPromise = promise;
```
 
</details>

> [!NOTE]
> When you're finished with the promise, call `Constants.otpPromise!.clean()` to free object from the memory.

### 3. Check which OTP type was being used with `otpPromise.authType`

Here, you can launch between views according to their authentication type as described in the example below.

<details>
<summary>Dart</summary>

```dart
    switch (Constants.otpPromise!.authType) {
      case OtpAuthType.he:
      	// Navigate view to HE view...
        break;
      case OtpAuthType.miscall:
      	// Navigate view to Miscall view...
        break;
      case OtpAuthType.sms:
      	// Navigate view to Message view...
        break;
      case OtpAuthType.whatsapp:
      	// Navigate view to Whatsapp view...
        break;
      case OtpAuthType.magicOtp:
      	// Navigate view to Magic Otp view...
        break;
      case OtpAuthType.magicLink:
      	// Navigate view to Magic Link view...
        break;
      case OtpAuthType.voice:
      	// Navigate view to Voice view...
        break;
    }
```
 
</details>

Recently, there are 6 auth type:

<details>
<summary><h4>HE auth type</h4></summary>

HE (Header Enrichment) uses network to verify the user. User will not receive an OTP and does not need to input any OTP. Only available if user uses data carrier for internet.

To validate this auth type, call `validateHE()` method. 
First callback will be fired if there is an error. 
Second callback will be fired if validation has been successful.

<details>
<summary>Dart</summary>

```dart
try {
  await Constants.otpPromise!.validateHE();
  
  final transactionId = Constants.otpPromise!.transactionId;
  // with the transactionId, check for the user verified status here...
} catch (e) {
  // handle error here...
}
```
 
</details>

</details>

<details>
<summary><h4>Miscall auth type</h4></summary>

This OTP will call user's phone number.

User has to fill the last several digits of the caller's phone number. Digit count can be obtained with `digitCount` property. There is also a miscall listener method `listenToMiscall()` (ANDROID ONLY). See code snippet down below for example usage.

To validate this auth type, call `validate()` method and fill the inputted user OTP in the parameter.
First callback will be fired if there is an error.
Second callback will be fired if validation has been successful.

<details>
<summary>Dart</summary>

```dart
import 'dart:io' show Platform;

final digitCount = Constants.otpPromise!.digitCount;

try {
	String otp = "";
	if (Platform.isAndroid) {
		otp = await Constants.otpPromise!.listenToMiscall();
	} else {
		otp = "USER_INPUTTED_OTP";
	}
  await Constants.otpPromise!.validate(otp);
  
  final transactionId = Constants.otpPromise!.transactionId;
  // with the transactionId, check for the user verified status here...
} catch (e) {
  // handle error here...
}
```

</details>

</details>

<details>
<summary><h4>SMS auth type</h4></summary>

This OTP will send an SMS to user's phone number.

User has to fill the OTP sent to their SMS inbox. Digit count can be obtained with `digitCount` property.

To validate this auth type, call `validate()` method and fill the inputted user OTP in the parameter.
First callback will be fired if there is an error.
Second callback will be fired if validation has been successful.

<details>
<summary>Dart</summary>

```dart
final digitCount = Constants.otpPromise!.digitCount;

try {
  await Constants.otpPromise!.validate("USER_INPUTTED_OTP");
  
  final transactionId = Constants.otpPromise!.transactionId;
  // with the transactionId, check for the user verified status here...
} catch (e) {
  // handle error here...
}
```
 
</details>

</details>

<details>
<summary><h4>Whatsapp auth type</h4></summary>

This OTP will send a Whatsapp message to user's phone number.

User has to fill the OTP sent to their Whatsapp. Digit count can be obtained with `digitCount` property.

To validate this auth type, call `validate()` method and fill the inputted user OTP in the parameter.
First callback will be fired if there is an error.
Second callback will be fired if validation has been successful.

<details>
<summary>Dart</summary>

```dart
final digitCount = Constants.otpPromise!.digitCount;

try {
  await Constants.otpPromise!.validate("USER_INPUTTED_OTP");
  
  final transactionId = Constants.otpPromise!.transactionId;
  // with the transactionId, check for the user verified status here...
} catch (e) {
  // handle error here...
}
```
 
</details>

</details>

<details>
<summary><h4>Magic Otp auth type</h4></summary>

User will be redirected to Whatsapp and required to send a prepared message to a specified phone number. 
Then user has to input the incoming OTP from their Whatsapp to your application.

With this auth type, call `launchWhatsappForMagicOtp()` method to launch Whatsapp.
First callback will be fired if there is an error when launching Whatsapp.
Second callback will be fired if Whatsapp launched successfully.

After Whatsapp has been launched successfully, you can validate the OTP using `validate()` method. 
Check [documentation](#whatsapp-auth-type) about Whatsapp auth type above.

<details>
<summary>Dart</summary>

```dart
try {
  await Constants.otpPromise!.launchWhatsappForMagicOtp();
  
	// show user a textfield to input the incoming OTP,
	// then call the validate Whatsapp method (Constants.otpPromise.validate())
} catch (e) {
  // handle error here...
}
```
 
</details>

</details>

<details>
<summary><h4>Magic Link auth type</h4></summary>

User will be redirected to Whatsapp and required to send a prepared message to a specified phone number. 
Then user has to click on the link from their Whatsapp.

With this auth type, call `launchWhatsappForMagicLink()` method to launch Whatsapp.
First callback will be fired if there is an error.
Second callback will be fired if validation has been successful.

<details>
<summary>Dart</summary>

```dart
try {
  await Constants.otpPromise!.launchWhatsappForMagicLink();
  
  final transactionId = Constants.otpPromise!.transactionId;
  // with the transactionId, check for the user verified status here...
} catch (e) {
  // handle error here...
}
```
 
</details>

</details>

### 4. Check for user verified status

Get the `transactionId` like this:

<details>
<summary>Dart</summary>

```dart
final transactionId = Constants.otpPromise!.transactionId;
```
 
</details>

Then check the [segment down below](#check-for-user-verified-status) on how to check if user has been successfully verified.

# Check for user verified status

A successfully validated OTP DOES NOT mean that the user has also been successfully verified. 
To check for user's verified status, check this [Server Documentation](README.Server.md#check-for-user-verified-status).
