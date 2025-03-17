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
// get fia instance
val fia = FIAFactory.getInstance()

fia.initialize(this, "YOUR_MERCHANT_KEY", "YOUR_MERCHANT_APP_ID")
```
 
</details>

<details>
<summary>Java</summary>

```java
// get fia instance
FIA fia = FIAFactory.getInstance();

fia.initialize(this, "YOUR_MERCHANT_KEY", "YOUR_MERCHANT_APP_ID");
```
 
</details>

There are two ways to request an OTP:
1. <b>With premade activity</b><br>If you want to request an OTP without making any view/UI for the OTP activity.
2. <b>With custom-made activity</b><br>If you want to make your own view/UI for the OTP activity.

## Request OTP with a premade activity

To request an OTP with premade activity, you have to create a class level variable and assign it in the activity `onCreate()` method.

### 1. Create a class-level variable with type `OtpActivitySettings`

Then assign this variable in the activity `onCreate()` method with `fia.otpActivity(this)`.  

<details>
<summary>Kotlin</summary>
 
```kotlin
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

Whereas phone is user's inputted phone number, and callback is called when OTP has been validated.

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
fia.otp(this).register("PHONE_NUMBER") { promise: OtpPromise ->
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

Here, you can differentiate between views according to their authentication type.

<details>
<summary>Kotlin</summary>

```kotlin
when (Constants.otpPromise.authType) {
	OtpAuthType.He -> {
		// On HE
	}
	OtpAuthType.Miscall -> {
		// On Miscall
	}
	OtpAuthType.SmsOrWhatsapp -> {
		// On Sms or Whatsapp
	}
	OtpAuthType.None -> {
		// On None
	}
}
```
 
</details>

<details>
<summary>Java</summary>

```java
switch (Constants.otpPromise.authType) {
	case OtpAuthType.He:
		// On HE
		break;
	case OtpAuthType.Miscall:
		// On Miscall
		break;
	case OtpAuthType.SmsOrWhatsapp:
		// On Sms or Whatsapp
		break;
	case OtpAuthType.None:
		// On None
		break;
}
```

</details>

#### OTP authentication type

Recently, there are 4 auth type:

##### HE (Header Enrichment)

HE uses network to verify the user.

User will not receive an OTP and does not need to input any OTP.

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
		// with the transactionId, check for the user verified status here...
	}
)
```

</details>

##### Miscall

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
			// with the transactionId, check for the user verified status here...
			return null;
		}
	);
	return null;
});
```

</details>

##### Sms or Whatsapp

This OTP will send an Sms or Whatsapp message to user's phone number.

User has to fill the OTP sent to their Sms inbox or Whatsapp. Digit count can be obtained with `digitCount` property.

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
		// with the transactionId, check for the user verified status here...
		return null;
	}
);
```

</details>

##### None (FIA)

None, it means there is no need to do OTP since user has already been verified.

User will not receive an OTP and does not need to input any OTP.

This auth type does not need to be validated. When user get this auth type, proceed to check user verified status.

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
> You have to call method `otpActivity()` or `otp()` using `FragmentActivity` OR `AppCompatActivity` as context.
> Otherwise your app might crash.

> [!CAUTION]
> You have to call method `otpActivity()` directly in the activity `onCreate()` method.
> Otherwise your app might crash.

> [!NOTE]
> A successfully validated OTP DOES NOT mean that the user has been successfully verified.
> Check the [segment down below](#check-for-user-verified-status) on how to check if user has been successfully verified.

> [!TIP]
> If you use android jetpack compose for UI builder, it's okay to change `ComponentActivity` to one of these activities.
> Because `AppCompatActivity` extends `FragmentActivity`, which extends `ComponentActivity`.
> [See the reference here.](https://stackoverflow.com/a/67364675)

# Check for user verified status

A successfully validated OTP DOES NOT mean that the user has been successfully verified. To check for user verified status, you have to hit our "Check user verified status" API.

REQUEST
Url: https://api.fazpass.com/v1/otp/fia/verification-status/THE_TRANSACTION_ID
Method: GET
Header:
- Authorization: Bearer Token (Use your MERCHANT_KEY as token)

JSON RESPONSE
- status (boolean)
- message (string)
- code (string)
- data :
	- is_verified (boolean)

The json key is data -> is_verified. If this value is true, then user is verified.
