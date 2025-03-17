# FIA Documentation

Documentation on how to use FIA SDK.

# Installation

## Add the dependency in your app-level build.gradle (*project*/app/build.gradle)

```gradle
dependencies {
	// Another dependencies...
	implementation 'com.fazpass:fia:1.0.0-alpha.4'
}
```

Then sync project with gradle files.

# Getting Started

Before using this SDK, make sure to get the Merchant Key and Merchant App ID from Keypaz Dashboard.

## Usage

First, you have to initialize the sdk once.

<details>
<summary>Kotlin</summary>
 
```kotlin
// get keypaz instance
val fia = FIAFactory.getInstance()

fia.initialize(this, "YOUR_MERCHANT_KEY", "YOUR_MERCHANT_APP_ID")
```
 
</details>

<details>
<summary>Java</summary>

```java
// get keypaz instance
FIA fia = FIAFactory.getInstance();

fia.initialize(this, "YOUR_MERCHANT_KEY", "YOUR_MERCHANT_APP_ID");
```
 
</details>

There are two ways to request an OTP:
1. <b>With premade activity</b><br>lets you to request an OTP without making any view/UI for the OTP activity.
2. <b>With custom-made activity</b><br>lets you to make your own view/UI for the OTP activity.

### Request OTP with a premade activity

First, you have to create a class-level variable with type `OtpActivitySettings`.
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

> [!CAUTION]
> You have to call method `otpActivity()` directly in the activity `onCreate()` method.
> Otherwise your app might crash.

Then, to launch the OTP activity, call one of the four methods which fits the purpose of the otp:
- login(phone, callback)
- register(phone, callback)
- transaction(phone, callback)
- forgetPassword(phone, callback)

Whereas phone is user's inputted phone number, and callback is called when OTP has been validated.

> [!NOTE]
> A successfully validated OTP DOES NOT mean that the user has been successfully verified.
> Check the [segment down below](#check-for-user-verified-status) on how to check if user has been successfully verified.

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

### Request OTP with a custom-made activity

Unlike requesting OTP with premade activity, you don't have to assign the variable in the activity `onCreate()` method.

### Important note

> [!CAUTION]
> You have to call method `otpActivity()` or `otp()` using `FragmentActivity` OR `AppCompatActivity` as context.
> Otherwise your app might crash.

> [!TIP]
> If you use android jetpack compose for UI builder, it's okay to change `ComponentActivity` to one of these activities.
> Because `AppCompatActivity` extends `FragmentActivity`, which extends `ComponentActivity`.
> [See the reference here.](https://stackoverflow.com/a/67364675)

## Check for user verified status

A successfully validated OTP DOES NOT mean that the user has been successfully verified. 
