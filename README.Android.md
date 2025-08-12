# FIA Documentation (Android)

Documentation on how to use FIA Android SDK.

# Installation

Add the dependency in your app-level build.gradle (*project*/app/build.gradle)

```gradle
dependencies {
	// Another dependencies...
	implementation 'com.fazpass:fia:1.1.1'
}
```

Then sync project with gradle files.

# Getting Started

Before using this SDK, make sure to get the Merchant Key and Merchant App ID from Keypaz Dashboard. Check this [Dashboard Documentation](README.Dashboard.md#retrieve-your-merchant-key).

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
<summary>Kotlin</summary>
 
```kotlin
import com.fazpass.fia.FIAFactory

// get fia instance
val fia = FIAFactory.getInstance()

fia.initialize(this, "YOUR_MERCHANT_KEY", "YOUR_MERCHANT_APP_ID")
```

</details>

<details>
<summary>Java</summary>

```java
import com.fazpass.fia.FIAFactory;
import com.fazpass.fia.interfaces.FIA;

// get fia instance
FIA fia = FIAFactory.getInstance();

fia.initialize(this, "YOUR_MERCHANT_KEY", "YOUR_MERCHANT_APP_ID");
```
 
</details>

There are two ways to request an OTP:
1. <b>With premade activity</b><br>If you want to request an OTP without making any view/UI for the OTP activity.
2. <b>With custom-made activity</b><br>If you want to make your own view/UI for the OTP activity.

## Request OTP with a premade activity

Requesting an OTP with premade activity is easier than requesting an OTP with custom-made activity. Use this one if you don't want to make your own view/UI.

### 1. Create a class-level variable with type `OtpActivitySettings`

Then assign this variable in the activity `onCreate()` method with `fia.otpActivity(this)`.  

<details>
<summary>Kotlin</summary>
 
```kotlin
import com.fazpass.fia.FIAFactory
import com.fazpass.fia.objects.OtpActivitySettings

class MainActivity: AppCompatActivity() {

	private val fia = FIAFactory.getInstance()

	// class-level variable
	private lateinit var otp: OtpActivitySettings

	override fun onCreate(savedInstanceState: Bundle?) {
		super.onCreate(savedInstanceState)

		// If FIA has not been initialized once, uncomment the line below.
		// fia.initialize(this, "YOUR_MERCHANT_KEY", "YOUR_MERCHANT_APP_ID")
		otp = fia.otpActivity(this)
	}
}
```
 
</details>

<details>
<summary>Java</summary>

```java
import com.fazpass.fia.FIAFactory;
import com.fazpass.fia.interfaces.FIA;
import com.fazpass.fia.objects.OtpActivitySettings;

public class MainActivity extends AppCompatActivity {

	private final FIA fia = FIAFactory.getInstance();

	// class-level variable
	private OtpActivitySettings otp;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);

		// If FIA has not been initialized once, uncomment the line below.
		// fia.initialize(this, "YOUR_MERCHANT_KEY", "YOUR_MERCHANT_APP_ID");
		otp = fia.otpActivity(this);
	}
}
```
 
</details>

### 2. Launch the OTP activity

To launch the OTP activity, call one of the four methods which fits the purpose of the otp:
- login()
- register()
- transaction()
- forgetPassword()

For example, we will use the login method.

<details>
<summary>Kotlin</summary>
 
```kotlin
otp.login("PHONE_NUMBER") { transactionId: String? ->
	// If transactionId is null, OTP validation has an error.
	if (transactionId == null) {
		// handle failed OTP validation here...
		return@login 
	}

	// with the transactionId, check for the user verified status here...
}
```
 
</details>

<details>
<summary>Java</summary>

```java
otp.login("PHONE_NUMBER", transactionId -> {
	// If transactionId is null, OTP validation has an error.
	if (transactionId == null) {
		// handle failed OTP validation here...
		return null;
	}

	// with the transactionId, check for the user verified status here...
	return null;
});
```
 
</details>

### 3. Check for user verified status

Get the `transactionId`, then check the [segment down below](#check-for-user-verified-status) on how to check if user has been successfully verified.

## Request OTP with a custom-made activity

Unlike requesting OTP with premade activity, you don't have to assign the variable in the activity `onCreate()` method. Also, there are more steps involved.

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

To request for an OTP, call one of the four methods which fits the purpose of the otp:
- login()
- register()
- transaction()
- forgetPassword()

For example, we will use the register method.

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

### 3. Check which OTP type was being used with `otpPromise.authType`

Here, you can launch between activities according to their authentication type as described in the example below.

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
	OtpAuthType.FIA -> {
		val intent = Intent(this@MainActivity, ValidateFIAActivity::class.java)
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
	case OtpAuthType.FIA:
		Intent intent = new Intent(MainActivity.this, ValidateFIAActivity.class);
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

Recently, there are 6 auth type:

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

</details>

<details>
<summary><h4>FIA auth type</h4></summary>

It's the OTP Intelligence System. User will not receive an OTP and does not need to input any OTP.

This auth type does not need to be validated. Immediately check for user verified status.

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
<summary>Kotlin</summary>

```kotlin
Constants.otpPromise.launchWhatsappForMagicOtp(
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
Constants.otpPromise.launchWhatsappForMagicOtp(
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

<details>
<summary>Kotlin</summary>

```kotlin
Constants.otpPromise.launchWhatsappForMagicLink(
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
Constants.otpPromise.launchWhatsappForMagicLink(
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

Get the `transactionId` like this:

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

Then check the [segment down below](#check-for-user-verified-status) on how to check if user has been successfully verified.

## Important note

> [!CAUTION]
> You have to call method `otpActivity()` directly in the activity `onCreate()` method.
> Otherwise your app might crash.

> [!CAUTION]
> You have to call method `otpActivity()` or `otp()` using `FragmentActivity` OR `AppCompatActivity` as context.
> Otherwise your app might crash.

> [!TIP]
> If you use android jetpack compose for UI builder, it's okay to change `ComponentActivity` to one of these activities.
> Because `AppCompatActivity` extends `FragmentActivity`, which extends `ComponentActivity`.
> [See the reference here.](https://stackoverflow.com/a/67364675)

# Check for user verified status

A successfully validated OTP DOES NOT mean that the user has also been successfully verified. To check for user's verified status, check this [Server Documentation](README.Server.md#check-for-user-verified-status).
