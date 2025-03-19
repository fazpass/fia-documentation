# FIA Documentation

Documentation on how to use FIA SDK.

# Installation

Add the dependency in your app-level build.gradle (*project*/app/build.gradle)

```gradle
dependencies {
	// Another dependencies...
	implementation 'com.fazpass:fia:1.0.0-alpha.4'
}
```

Then sync project with gradle files.

# Getting Started

Before using this SDK, make sure to get the Merchant Key and Merchant App ID from Keypaz Dashboard.

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
- login(phone, callback)
- register(phone, callback)
- transaction(phone, callback)
- forgetPassword(phone, callback)

Whereas phone is user's inputted phone number, and callback is fired when OTP has been validated.

For example, you want to request OTP for login purpose, then the code will be:

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

### 2. To request for an OTP, call the `otp()` method then pick the method which fits the OTP purpose

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
	OtpAuthType.He -> {
		val intent = Intent(this@MainActivity, ValidateHEActivity::class.java)
		startActivity(intent)
	}
	OtpAuthType.Miscall -> {
		val intent = Intent(this@MainActivity, ValidateMiscallActivity::class.java)
		startActivity(intent)
	}
	OtpAuthType.Message -> {
		val intent = Intent(this@MainActivity, ValidateMessageActivity::class.java)
		startActivity(intent)
	}
	OtpAuthType.FIA -> {
		val intent = Intent(this@MainActivity, ValidateFIAActivity::class.java)
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
	case OtpAuthType.He:
		Intent intent = new Intent(MainActivity.this, ValidateHEActivity.class);
		startActivity(intent);
		break;
	case OtpAuthType.Miscall:
		Intent intent = new Intent(MainActivity.this, ValidateMiscallActivity.class);
		startActivity(intent);
		break;
	case OtpAuthType.Message:
		Intent intent = new Intent(MainActivity.this, ValidateMessageActivity.class);
		startActivity(intent);
		break;
	case OtpAuthType.FIA:
		Intent intent = new Intent(MainActivity.this, ValidateFIAActivity.class);
		startActivity(intent);
		break;
}
```

</details>

Recently, there are 4 auth type:

#### HE (Header Enrichment)

HE uses network to verify the user. User will not receive an OTP and does not need to input any OTP. Only available if user uses data carrier for internet.

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

#### Miscall

This OTP will call user's phone number. Only available if user has granted these 2 permissions for miscall autofill:
- Manifest.permission.READ_PHONE_STATE
- Manifest.permission.READ_CALL_LOG

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

#### Message

This OTP will send a Message to user's phone number.

User has to fill the OTP sent to their Sms inbox or any messaging service. Digit count can be obtained with `digitCount` property.

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

#### FIA

It's the OTP Intelligence System. User will not receive an OTP and does not need to input any OTP.

This auth type does not need to be validated. Immediately check for user verified status.

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

A successfully validated OTP DOES NOT mean that the user has also been successfully verified. To check for user's verified status, check this [Server Documentation](README.Server.md).
