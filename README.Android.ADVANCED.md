# FIA Documentation (Android) — Advanced

Full SDK configuration and customization guide for FIA Android SDK.

For a minimal quickstart without customization, see the [Simple Documentation](README.Android.SIMPLE.md).

This SDK requires minimum android api level 24 (7.0 Nougat) to works.

# Installation

Add the dependency in your app-level build.gradle (*project*/app/build.gradle):

```gradle
dependencies {
	// Another dependencies...
	implementation 'com.fazpass:fia:1.3.4'
}
```

Then sync project with gradle files.

# Getting Started

Before using this SDK, make sure to get the Merchant Key and Merchant App ID from Fazpass Dashboard. Check this [Dashboard Documentation](README.Dashboard.md#retrieve-your-merchant-key).

<details>
<summary><h2>Setup Miscall</h2></summary>

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

<details>
<summary>Java</summary>

 ```java
String[] requiredPermissions = { Manifest.permission.READ_PHONE_STATE, Manifest.permission.READ_CALL_LOG };
ActivityCompat.requestPermissions(this, requiredPermissions, 0);
```

</details>

</details>

<details>
<summary><h2>Setup HE</h2></summary>

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
<summary><h2>Setup Magic Link</h2></summary>

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

Fill `YOUR_PACKAGE_NAME` with your app package name (example: `com.example.app`), `YOUR_SHA256_CERT_FINGERPRINT` with your app SHA256 certificate fingerprint.

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

<details>
<summary><h2>Setup Whatsapp Zero Tap</h2></summary>

Whatsapp Zero Tap delivers the OTP straight to your app. User does not have to open Whatsapp or input anything.

It only works after the Fazpass team has registered your app hashes in your Whatsapp Zero Tap template.

### 1. Retrieve your app hashes

Call `getWhatsappZeroTapHashes()` and print the result. FIA has to be initialized before calling this method.

<details>
<summary>Kotlin</summary>

```kotlin
val hashes = fia.getWhatsappZeroTapAppHashes(this)
Log.d("FIA", "Whatsapp Zero Tap hashes: $hashes")
```

</details>

<details>
<summary>Java</summary>

```java
List<String> hashes = fia.getWhatsappZeroTapAppHashes(this);
Log.d("FIA", "Whatsapp Zero Tap hashes: " + hashes);
```

</details>

### 2. Send the hashes to the Fazpass team

Send every value in the returned list to our admin, so they can register it in your Whatsapp Zero Tap template. To contact our admin, check this [Dashboard Documentation](README.Dashboard.md#whitelist-ip).

> [!IMPORTANT]
> The hash is derived from your app package name and signing certificate, so a debug build and a production build produce different hashes.
> Take the hashes from the build you actually ship to your user. If you use Play App Signing, install your app from Playstore (the internal testing track works too) and read the hashes from there, because Google re-signs your uploaded app with a different certificate.
> Send your debug hashes as well if you want Zero Tap to work while you are still developing.

### 3. Check for device support

Call `isWhatsappZeroTapSupported()` to know whether the current device is able to receive a Zero Tap OTP. Always keep the manual OTP input as a fallback for when it returns `false`.

<details>
<summary>Kotlin</summary>

```kotlin
val isSupported = fia.isWhatsappZeroTapSupported(this)
```

</details>

<details>
<summary>Java</summary>

```java
boolean isSupported = fia.isWhatsappZeroTapSupported(this);
```

</details>

> [!NOTE]
> To receive the Zero Tap OTP, use the `listenToWhatsappZeroTap()` method. Check this [documentation](#whatsapp-auth-type) about Whatsapp auth type.

</details>

# Initialize the SDK

Initialize the SDK once before using it.

<details>
<summary>Kotlin</summary>

```kotlin
import com.fazpass.fia.FIAFactory

val fia = FIAFactory.getInstance()

fia.initialize(this, "YOUR_MERCHANT_KEY", "YOUR_MERCHANT_APP_ID")
```

</details>

<details>
<summary>Java</summary>

```java
import com.fazpass.fia.FIAFactory;
import com.fazpass.fia.interfaces.FIA;

FIA fia = FIAFactory.getInstance();

fia.initialize(this, "YOUR_MERCHANT_KEY", "YOUR_MERCHANT_APP_ID");
```

</details>

# Request OTP with a Custom-Made Activity

Unlike the premade activity approach, you build your own OTP screen and handle each authentication type manually. There are more steps involved.

### 1. Create a public class to hold a static variable of type `OtpPromise`

<details>
<summary>Kotlin</summary>

```kotlin
import com.fazpass.fia.objects.OtpPromise

class Constants {
	companion object {
		lateinit var otpPromise: OtpPromise
	}
}
```

</details>

<details>
<summary>Java</summary>

```java
import com.fazpass.fia.objects.OtpPromise;

public class Constants {
	public static OtpPromise otpPromise;
}
```

</details>

### 2. Request for an OTP

Call one of the four methods that fits your use case: `login()`, `register()`, `transaction()`, or `forgetPassword()`.

<details>
<summary>Kotlin</summary>

```kotlin
fia.otp(this).register("PHONE_NUMBER") { promise ->
	if (promise.hasException) {
		val exception = promise.exception
		// handle failed OTP request here...
		return@register 
	}

	Constants.otpPromise = promise
}
```

</details>

<details>
<summary>Java</summary>

```java
fia.otp(this).register("PHONE_NUMBER", promise -> {
	if (promise.getHasException()) {
		Exception exception = promise.getException();
		// handle failed OTP request here...
		return null;
	}

	Constants.otpPromise = promise;
	return null;
})
```

</details>

### 3. Check which OTP type was used with `otpPromise.authType`

Launch the appropriate activity for each authentication type.

<details>
<summary>Kotlin</summary>

```kotlin
import com.fazpass.fia.objects.OtpAuthType

when (Constants.otpPromise.authType) {
	OtpAuthType.HE -> {
		val intent = Intent(this@MainActivity, ValidateHEActivity::class.java)
		startActivity(intent)
	}
	OtpAuthType.Miscall -> {
		val intent = Intent(this@MainActivity, ValidateMiscallActivity::class.java)
		startActivity(intent)
	}
	OtpAuthType.SMS -> {
		val intent = Intent(this@MainActivity, ValidateSMSActivity::class.java)
		startActivity(intent)
	}
	OtpAuthType.Whatsapp -> {
		val intent = Intent(this@MainActivity, ValidateWhatsappActivity::class.java)
		startActivity(intent)
	}
	OtpAuthType.MagicOtp -> {
		val intent = Intent(this@MainActivity, ValidateMagicOtpActivity::class.java)
		startActivity(intent)
	}
	OtpAuthType.MagicLink -> {
		val intent = Intent(this@MainActivity, ValidateMagicLinkActivity::class.java)
		startActivity(intent)
	}
}
```

</details>

<details>
<summary>Java</summary>

```java
import com.fazpass.fia.objects.OtpAuthType;

switch (Constants.otpPromise.getAuthType()) {
	case OtpAuthType.HE:
		Intent intent = new Intent(MainActivity.this, ValidateHEActivity.class);
		startActivity(intent);
		break;
	case OtpAuthType.Miscall:
		Intent intent = new Intent(MainActivity.this, ValidateMiscallActivity.class);
		startActivity(intent);
		break;
	case OtpAuthType.SMS:
		Intent intent = new Intent(MainActivity.this, ValidateSMSActivity.class);
		startActivity(intent);
		break;
	case OtpAuthType.Whatsapp:
		Intent intent = new Intent(MainActivity.this, ValidateWhatsappActivity.class);
		startActivity(intent);
		break;
	case OtpAuthType.MagicOtp:
		Intent intent = new Intent(MainActivity.this, ValidateMagicOtpActivity.class);
		startActivity(intent);
		break;
	case OtpAuthType.MagicLink:
		Intent intent = new Intent(MainActivity.this, ValidateMagicLinkActivity.class);
		startActivity(intent);
		break;
}
```

</details>

There are 6 auth types:

<details>
<summary><h4>HE auth type</h4></summary>

HE (Header Enrichment) uses network to verify the user. User will not receive an OTP and does not need to input any OTP. Only available if user uses data carrier for internet.

To validate this auth type, call `validateHE()` method. 
First callback will be fired if there is an error. 
Second callback will be fired if validation has been successful.

<details>
<summary>Kotlin</summary>

```kotlin
Constants.otpPromise.validateHE(
	{ err ->
		// handle error here...
	},
	{
		val transactionId = Constants.otpPromise.transactionId
		// with the transactionId, check for the user verified status here...
	}
)
```

</details>

<details>
<summary>Java</summary>

```java
Constants.otpPromise.validateHE(
	err -> {
		// handle error here...
	},
	() -> {
		String transactionId = Constants.otpPromise.getTransactionId();
		// with the transactionId, check for the user verified status here...
	}
)
```

</details>

</details>

<details>
<summary><h4>Miscall auth type</h4></summary>

This OTP will call user's phone number.

User has to fill the last several digits of the caller's phone number. Digit count can be obtained with `digitCount` property.
There is also a miscall listener method `listenToMiscall()`. See code snippet down below for example usage.

To validate this auth type, call `validate()` method and fill the inputted user OTP in the parameter.
First callback will be fired if there is an error.
Second callback will be fired if validation has been successful.

<details>
<summary>Kotlin</summary>

```kotlin
val digitCount = Constants.otpPromise.digitCount

// miscall OTP listener
Constants.otpPromise.listenToMiscall { otp ->
	// validate OTP method
	Constants.otpPromise.validate(
		otp,
		{ err ->
			// handle error here...
		},
		{
			val transactionId = Constants.otpPromise.transactionId
			// with the transactionId, check for the user verified status here...
		}
	)
}
```

</details>

<details>
<summary>Java</summary>

```java
Int digitCount = Constants.otpPromise.getDigitCount();

// miscall OTP listener
Constants.otpPromise.listenToMiscall(otp -> {
	// validate OTP method
	Constants.otpPromise.validate(
		otp,
		err -> {
			// handle error here...
			return null;
		},
		() -> {
			String transactionId = Constants.otpPromise.getTransactionId();
			// with the transactionId, check for the user verified status here...
			return null;
		}
	);
	return null;
});
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
<summary>Kotlin</summary>

```kotlin
val digitCount = Constants.otpPromise.digitCount

Constants.otpPromise.validate(
	"USER_INPUTTED_OTP",
	{ err ->
		// handle error here...
	},
	{
		val transactionId = Constants.otpPromise.transactionId
		// with the transactionId, check for the user verified status here...
	}
)
```

</details>

<details>
<summary>Java</summary>

```java
Int digitCount = Constants.otpPromise.getDigitCount();

Constants.otpPromise.validate(
	"USER_INPUTTED_OTP",
	err -> {
		// handle error here...
		return null;
	},
	() -> {
		String transactionId = Constants.otpPromise.getTransactionId();
		// with the transactionId, check for the user verified status here...
		return null;
	}
);
```

</details>

</details>

<details>
<summary><h4>Whatsapp auth type</h4></summary>

This OTP will send a Whatsapp message to user's Whatsapp number.

User has to fill the OTP sent to their Whatsapp. Digit count can be obtained with `digitCount` property.

To validate this auth type, call `validate()` method and fill the inputted user OTP in the parameter.
First callback will be fired if there is an error.
Second callback will be fired if validation has been successful.

<details>
<summary>Kotlin</summary>

```kotlin
val digitCount = Constants.otpPromise.digitCount

Constants.otpPromise.validate(
	"USER_INPUTTED_OTP",
	{ err ->
		// handle error here...
	},
	{
		val transactionId = Constants.otpPromise.transactionId
		// with the transactionId, check for the user verified status here...
	}
)
```

</details>

<details>
<summary>Java</summary>

```java
Int digitCount = Constants.otpPromise.getDigitCount();

Constants.otpPromise.validate(
	"USER_INPUTTED_OTP",
	err -> {
		// handle error here...
		return null;
	},
	() -> {
		String transactionId = Constants.otpPromise.getTransactionId();
		// with the transactionId, check for the user verified status here...
		return null;
	}
);
```

</details>

There is also a Zero Tap listener method `listenToWhatsappZeroTap()`.
If Whatsapp Zero Tap is supported and your app hashes have been registered (check [Setup Whatsapp Zero Tap](#setup-whatsapp-zero-tap)), the OTP will be delivered to your app automatically and user does not have to input anything.

First callback will be fired if there is an error. In that case, keep showing the OTP textfield so user can still input the OTP manually.
Second callback will be fired with the incoming OTP. Validate it with the `validate()` method.

<details>
<summary>Kotlin</summary>

```kotlin
if (fia.isWhatsappZeroTapSupported(this)) {
	// whatsapp zero tap OTP listener
	Constants.otpPromise.listenToWhatsappZeroTap(
		{ err ->
			// handle error here, then let user input the OTP manually...
		}
	) { otp ->
		// validate OTP method
		Constants.otpPromise.validate(
			otp,
			{ err ->
				// handle error here...
			},
			{
				val transactionId = Constants.otpPromise.transactionId
				// with the transactionId, check for the user verified status here...
			}
		)
	}
}
```

</details>

<details>
<summary>Java</summary>

```java
if (fia.isWhatsappZeroTapSupported(this)) {
	// whatsapp zero tap OTP listener
	Constants.otpPromise.listenToWhatsappZeroTap(
		err -> {
			// handle error here, then let user input the OTP manually...
			return null;
		},
		otp -> {
			// validate OTP method
			Constants.otpPromise.validate(
				otp,
				err -> {
					// handle error here...
					return null;
				},
				() -> {
					String transactionId = Constants.otpPromise.getTransactionId();
					// with the transactionId, check for the user verified status here...
					return null;
				}
			);
			return null;
		}
	);
}
```

</details>

> [!NOTE]
> The `onFailed` callback is optional in Kotlin, so `Constants.otpPromise.listenToWhatsappZeroTap { otp -> ... }` is valid too.

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

You can also pass the optional `magicRedirect` parameter to control which Whatsapp app is used for redirection.

| Value | Description |
|---|---|
| `OtpMagicRedirect.AUTO` | Automatically selects WhatsApp or WhatsApp Business (default) |
| `OtpMagicRedirect.WHATSAPP_NORMAL` | Always redirects to WhatsApp |
| `OtpMagicRedirect.WHATSAPP_BUSINESS` | Always redirects to WhatsApp Business |
| `OtpMagicRedirect.MANUAL` | Shows a dialog letting the user choose which WhatsApp app to use when both WhatsApp and WhatsApp Business are installed |

<details>
<summary>Kotlin</summary>

```kotlin
import com.fazpass.fia.objects.OtpMagicRedirect

Constants.otpPromise.launchWhatsappForMagicOtp(
	magicRedirect = OtpMagicRedirect.WHATSAPP_NORMAL,
	{ err ->
		// handle error here...
	},
	{
		// show user a textfield to input the incoming OTP,
		// then call the validate Whatsapp method (Constants.otpPromise.validate())
	}
)
```

</details>

<details>
<summary>Java</summary>

```java
import com.fazpass.fia.objects.OtpMagicRedirect;

Constants.otpPromise.launchWhatsappForMagicOtp(
	OtpMagicRedirect.WHATSAPP_NORMAL,
	err -> {
		// handle error here...
		return null;
	},
	() -> {
		// show user a textfield to input the incoming OTP,
		// then call the validate Whatsapp method (Constants.otpPromise.validate())
		return null;
	}
);
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

You can also pass the optional `magicRedirect` parameter to control which Whatsapp app is used for redirection. 
Check [documentation](#magic-otp-auth-type) about Magic Otp auth type above for the available values.

<details>
<summary>Kotlin</summary>

```kotlin
import com.fazpass.fia.objects.OtpMagicRedirect

Constants.otpPromise.launchWhatsappForMagicLink(
	magicRedirect = OtpMagicRedirect.WHATSAPP_NORMAL,
	{ err ->
		// handle error here...
	},
	{
		val transactionId = Constants.otpPromise.transactionId
		// with the transactionId, check for the user verified status here...
	}
)
```

</details>

<details>
<summary>Java</summary>

```java
import com.fazpass.fia.objects.OtpMagicRedirect;

Constants.otpPromise.launchWhatsappForMagicLink(
	OtpMagicRedirect.WHATSAPP_NORMAL,
	err -> {
		// handle error here...
		return null;
	},
	() -> {
		String transactionId = Constants.otpPromise.getTransactionId();
		// with the transactionId, check for the user verified status here...
		return null;
	}
);
```

</details>

</details>

### 4. Check for user verified status

A successfully validated OTP does **not** mean that the user has also been successfully verified. To check for user's verified status, get the `transactionId`:

<details>
<summary>Kotlin</summary>

```kotlin
val transactionId = Constants.otpPromise.transactionId
```

</details>

<details>
<summary>Java</summary>

```java
String transactionId = Constants.otpPromise.getTransactionId();
```

</details>

Then check the [Server Documentation](README.Server.md#check-for-user-verified-status) to verify the user.

# Request OTP with a User-Preferred Auth Type

This flow is mostly the same as the [Request OTP with a Custom-Made Activity](#request-otp-with-a-custom-made-activity) approach, with one extra step before requesting an OTP. Instead of letting the SDK decide which auth type to use, you first retrieve every available auth type (gateway) for the phone number, then let the user pick the one they prefer.

### 1. Request for the available auth types

Call `otpManual()` with one of the four methods that fits your use case: `login()`, `register()`, `transaction()`, or `forgetPassword()`. The callback returns an `OtpGatewayPromise`.

<details>
<summary>Kotlin</summary>

```kotlin
import com.fazpass.fia.objects.OtpGatewayPromise

fia.otpManual(this).register("PHONE_NUMBER") { gatewayPromise ->
	if (gatewayPromise.hasException) {
		val exception = gatewayPromise.exception
		// handle failed request here...
		return@register
	}

	if (gatewayPromise.isAuthenticated) {
		val transactionId = gatewayPromise.transactionId
		// user has already been authenticated, no OTP is needed.
		// with the transactionId, check for the user verified status here...
		return@register
	}

	Constants.otpGatewayPromise = gatewayPromise
	// show the available auth types (gatewayPromise.gateways) to the user...
}
```

</details>

<details>
<summary>Java</summary>

```java
import com.fazpass.fia.objects.OtpGatewayPromise;

fia.otpManual(this).register("PHONE_NUMBER", gatewayPromise -> {
	if (gatewayPromise.getHasException()) {
		Exception exception = gatewayPromise.getException();
		// handle failed request here...
		return null;
	}

	if (gatewayPromise.isAuthenticated()) {
		String transactionId = gatewayPromise.getTransactionId();
		// user has already been authenticated, no OTP is needed.
		// with the transactionId, check for the user verified status here...
		return null;
	}

	Constants.otpGatewayPromise = gatewayPromise;
	// show the available auth types (gatewayPromise.getGateways()) to the user...
	return null;
});
```

</details>

`OtpGatewayPromise` has these properties:

| Property | Description |
|---|---|
| `isAuthenticated` | `Boolean`. True if the user has already been authenticated and does not need to request an OTP |
| `transactionId` | `String`. Only filled when `isAuthenticated` is true, otherwise it is an empty string |
| `hasException` | `Boolean`. True if the request has failed |
| `exception` | `FIAException`. The cause of the failure when `hasException` is true |
| `gateways` | `List<OtpGateway>`. Every auth type available for this phone number |

And every `OtpGateway` has these properties:

| Property | Description |
|---|---|
| `number` | `Int`. The identifier of this auth type, to be passed to the `pick()` method |
| `name` | `String`. The name of this auth type, to be shown to the user |

> [!NOTE]
> If `isAuthenticated` is true, the flow ends here. Take the `transactionId` and go straight to [4. Check for user verified status](#4-check-for-user-verified-status).

This flow also needs the `Constants` class from [step 1 of the custom-made activity approach](#1-create-a-public-class-to-hold-a-static-variable-of-type-otppromise). Add an `otpGatewayPromise` variable next to the existing `otpPromise` one.

<details>
<summary>Kotlin</summary>

```kotlin
import com.fazpass.fia.objects.OtpGatewayPromise
import com.fazpass.fia.objects.OtpPromise

class Constants {
	companion object {
		lateinit var otpGatewayPromise: OtpGatewayPromise
		lateinit var otpPromise: OtpPromise
	}
}
```

</details>

<details>
<summary>Java</summary>

```java
import com.fazpass.fia.objects.OtpGatewayPromise;
import com.fazpass.fia.objects.OtpPromise;

public class Constants {
	public static OtpGatewayPromise otpGatewayPromise;
	public static OtpPromise otpPromise;
}
```

</details>

### 2. Let the user pick their preferred auth type

Call the `pick()` method with the `number` of the auth type the user has chosen. It requests an OTP through that auth type, and its callback returns an `OtpPromise` — the same object the standard flow produces.

<details>
<summary>Kotlin</summary>

```kotlin
val gateway = Constants.otpGatewayPromise.gateways[SELECTED_INDEX]

Constants.otpGatewayPromise.pick(gateway.number) { promise ->
	if (promise.hasException) {
		val exception = promise.exception
		// handle failed OTP request here...
		return@pick
	}

	Constants.otpPromise = promise
}
```

</details>

<details>
<summary>Java</summary>

```java
OtpGateway gateway = Constants.otpGatewayPromise.getGateways().get(SELECTED_INDEX);

Constants.otpGatewayPromise.pick(gateway.getNumber(), promise -> {
	if (promise.getHasException()) {
		Exception exception = promise.getException();
		// handle failed OTP request here...
		return null;
	}

	Constants.otpPromise = promise;
	return null;
});
```

</details>

### 3. Validate the user

From this point on, the flow is identical to the custom-made activity approach. Continue with:

- [3. Check which OTP type was used with `otpPromise.authType`](#3-check-which-otp-type-was-used-with-otppromiseauthtype)
- [4. Check for user verified status](#4-check-for-user-verified-status)

# Important Notes

> [!CAUTION]
> You must call `otp()` or `otpManual()` using a `FragmentActivity` or `AppCompatActivity` as context. Otherwise your app might crash.

> [!TIP]
> If you use Jetpack Compose, it's safe to change `ComponentActivity` to `AppCompatActivity`.
> `AppCompatActivity` extends `FragmentActivity`, which extends `ComponentActivity`.
> [See the reference here.](https://stackoverflow.com/a/67364675)
