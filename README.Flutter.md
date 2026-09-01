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
import 'package:fia/fia.dart';                  // Fia
import 'package:fia/otp_promise.dart';          // OtpPromise
import 'package:fia/otp_auth_type.dart';        // OtpAuthType
import 'package:fia/otp_magic_redirect.dart';   // OtpMagicRedirect
import 'package:fia/otp_gateway_promise.dart';  // OtpGatewayPromise
import 'package:fia/otp_gateway.dart';          // OtpGateway
```

# Getting Started

Before using this SDK, make sure to get the Merchant Key and Merchant App ID from Fazpass Dashboard. 
Check this [Dashboard Documentation](README.Dashboard.md#retrieve-your-merchant-key).

<details>
<summary><h2>Setup iOS</h2></summary>

In your XCode, add these capabilities in 'Signing & Capabilities':
1. App Groups (container `group.YOUR_INVERTED_DOMAIN`)
2. iCloud (service `Key-value storage`)

![XCode Signing & Capabilities](images/xcode-signing-capabilities.png)

> [!IMPORTANT]
> The App Group container you create here has to be passed to `initialize()` as the `iosGroupId` argument. See the [Usage](#usage) section below.

</details>

<details>
<summary><h2>Setup Magic OTP (iOS)</h2></summary>

Add this to your `ios/Runner/Info.plist` file:

```xml
<key>LSApplicationQueriesSchemes</key>
<array>
	<string>whatsapp</string>
	<string>whatsappbusiness</string>
</array>
```

</details>

<details>
<summary><h2>Setup Magic Link (iOS)</h2></summary>

Add this to your `ios/Runner/Info.plist` file:

```xml
<key>LSApplicationQueriesSchemes</key>
<array>
	<string>whatsapp</string>
	<string>whatsappbusiness</string>
</array>
```

In XCode, under 'Signing & Capabilities', add an **Associated Domains** capability and add this entry: `applinks:YOUR_DOMAIN`, where `YOUR_DOMAIN` is your website domain.

Then create a new file named `apple-app-site-association` with this content:

```json
{
  "applinks": {
    "apps": [],
    "details": [
      {
        "appID": "YOUR_TEAM_ID.YOUR_APP_BUNDLE_ID",
        "paths": [ "*" ]
      }
    ]
  }
}
```

Fill `YOUR_TEAM_ID` with your Apple Team ID (example: `ABCD1234`) and `YOUR_APP_BUNDLE_ID` with your app bundle ID (example: `com.example.app`).

<details>
<summary><h3>How to get your Team ID</h3></summary>

1. Open the [Apple Developer Account website](https://developer.apple.com/account)
2. Check your **Membership** details — your Team ID is listed there

</details>

Save the `apple-app-site-association` file and serve it at: `https://YOUR_DOMAIN.com/.well-known/apple-app-site-association`. Make sure:
1. It is publicly accessible
2. There are no redirects
3. Content-Type is `application/json`

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
fia.initialize(
  "MERCHANT_KEY",
  "MERCHANT_APP_ID",
  iosGroupId: "group.YOUR_INVERTED_DOMAIN",
);
```

</details>

`iosGroupId` is the App Group container you created in the **Setup iOS** section. It is optional and ignored on Android, but iOS will not work correctly without it.

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
      case OtpAuthType.voice:
      	// Navigate view to Voice view...
        break;
      case OtpAuthType.magicOtp:
      	// Navigate view to Magic Otp view...
        break;
      case OtpAuthType.magicLink:
      	// Navigate view to Magic Link view...
        break;
    }
```
 
</details>

There are 7 auth types:

<details>
<summary><h4>HE auth type</h4></summary>

HE (Header Enrichment) uses network to verify the user. User will not receive an OTP and does not need to input any OTP. Only available if user uses data carrier for internet.

To validate this auth type, call `validateHE()` method. 
It completes when validation has been successful, and throws if there is an error.

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
It completes when validation has been successful, and throws if there is an error.

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
It completes when validation has been successful, and throws if there is an error.

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
It completes when validation has been successful, and throws if there is an error.

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
<summary><h4>Voice auth type</h4></summary>

This OTP will call user's phone number and read the OTP out loud.

User has to fill the OTP they heard. Digit count can be obtained with `digitCount` property.

To validate this auth type, call `validate()` method and fill the inputted user OTP in the parameter.
It completes when validation has been successful, and throws if there is an error.

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
It completes when Whatsapp has launched successfully, and throws if there is an error when launching Whatsapp.

After Whatsapp has been launched successfully, you can validate the OTP using `validate()` method. 
Check [documentation](#whatsapp-auth-type) about Whatsapp auth type above.

You can also pass the optional `magicRedirect` parameter to control which Whatsapp app is used for redirection.

| Value | Description |
|---|---|
| `OtpMagicRedirect.auto` | Automatically selects WhatsApp or WhatsApp Business (default) |
| `OtpMagicRedirect.whatsappNormal` | Always redirects to WhatsApp |
| `OtpMagicRedirect.whatsappBusiness` | Always redirects to WhatsApp Business |
| `OtpMagicRedirect.manual` | Shows a dialog letting the user choose which WhatsApp app to use when both WhatsApp and WhatsApp Business are installed |

<details>
<summary>Dart</summary>

```dart
import 'package:fia/otp_magic_redirect.dart';

try {
  await Constants.otpPromise!.launchWhatsappForMagicOtp(
    magicRedirect: OtpMagicRedirect.whatsappNormal,
  );
  
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
It completes when validation has been successful, and throws if there is an error.

You can also pass the optional `magicRedirect` parameter to control which Whatsapp app is used for redirection. 
Check [documentation](#magic-otp-auth-type) about Magic Otp auth type above for the available values.

<details>
<summary>Dart</summary>

```dart
import 'package:fia/otp_magic_redirect.dart';

try {
  await Constants.otpPromise!.launchWhatsappForMagicLink(
    magicRedirect: OtpMagicRedirect.whatsappNormal,
  );
  
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

## Request OTP with a User-Preferred Auth Type

This flow is mostly the same as the [Request and Validate OTP](#request-and-validate-otp) approach, with one extra step before requesting an OTP. Instead of letting the SDK decide which auth type to use, you first retrieve every available auth type (gateway) for the phone number, then let the user pick the one they prefer.

### 1. Request for the available auth types

Call `otpManual()` with one of the four methods that fits your use case: `login()`, `register()`, `transaction()`, or `forgetPassword()`. It returns an `OtpGatewayPromise`.

<details>
<summary>Dart</summary>

```dart
final gatewayPromise = await fia.otpManual().register("PHONE_NUMBER");
if (gatewayPromise.hasException) {
  final error = gatewayPromise.exception;
  // handle failed request here...
  return;
}

if (gatewayPromise.isAuthenticated) {
  final transactionId = gatewayPromise.transactionId;
  // user has already been authenticated, no OTP is needed.
  // with the transactionId, check for the user verified status here...
  await gatewayPromise.clean();
  return;
}

Constants.otpGatewayPromise = gatewayPromise;
// show the available auth types (gatewayPromise.gateways) to the user...
```

</details>

`OtpGatewayPromise` has these properties:

| Property | Description |
|---|---|
| `isAuthenticated` | `bool`. True if the user has already been authenticated and does not need to request an OTP |
| `transactionId` | `String`. Only filled when `isAuthenticated` is true, otherwise it is an empty string |
| `hasException` | `bool`. True if the request has failed |
| `exception` | `String?`. The cause of the failure when `hasException` is true |
| `gateways` | `List<OtpGateway>`. Every auth type available for this phone number |

And every `OtpGateway` has these properties:

| Property | Description |
|---|---|
| `number` | `int`. The identifier of this auth type, to be passed to the `pick()` method |
| `name` | `String`. The name of this auth type, to be shown to the user |

> [!NOTE]
> If `isAuthenticated` is true, the flow ends here. Take the `transactionId` and go straight to [4. Check for user verified status](#4-check-for-user-verified-status).

This flow also needs the `Constants` class from [step 1 above](#1-create-a-public-class-to-hold-a-static-variable-of-type-otppromise). Add an `otpGatewayPromise` variable next to the existing `otpPromise` one.

<details>
<summary>Dart</summary>

```dart
class Constants {
	static OtpGatewayPromise? otpGatewayPromise;
	static OtpPromise? otpPromise;
}
```

</details>

> [!NOTE]
> `OtpGatewayPromise` holds native memory just like `OtpPromise` does. When you're finished with it, call `Constants.otpGatewayPromise!.clean()` — either after `pick()` has returned, or after the `isAuthenticated` shortcut above.

### 2. Let the user pick their preferred auth type

Call the `pick()` method with the `number` of the auth type the user has chosen. It requests an OTP through that auth type and returns an `OtpPromise` — the same object the standard flow produces.

<details>
<summary>Dart</summary>

```dart
final gateway = Constants.otpGatewayPromise!.gateways[SELECTED_INDEX];

final promise = await Constants.otpGatewayPromise!.pick(gateway.number);
if (promise.hasException) {
  final error = promise.exception;
  // handle failed OTP request here...
  return;
}

Constants.otpPromise = promise;
await Constants.otpGatewayPromise!.clean();
```

</details>

### 3. Validate the user

From this point on, the flow is identical to the standard approach. Continue with:

- [3. Check which OTP type was being used with `otpPromise.authType`](#3-check-which-otp-type-was-being-used-with-otppromiseauthtype)
- [4. Check for user verified status](#4-check-for-user-verified-status)

# Check for user verified status

A successfully validated OTP DOES NOT mean that the user has also been successfully verified. 
To check for user's verified status, check this [Server Documentation](README.Server.md#check-for-user-verified-status).
