# FIA Documentation (Android) — Quick Start

Get up and running with FIA Android SDK in minutes.

For full SDK configuration and customization, see the [Advanced Documentation](README.Android.ADVANCED.md).

# Installation

Add the dependency in your app-level build.gradle (*project*/app/build.gradle):

```gradle
dependencies {
	// Another dependencies...
	implementation 'com.fazpass:fia:1.2.7'
}
```

Then sync project with gradle files.

# Getting Started

Before using this SDK, make sure to get the Merchant Key and Merchant App ID from Keypaz Dashboard. Check this [Dashboard Documentation](README.Dashboard.md#retrieve-your-merchant-key).

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

# Request OTP with a Premade Activity

The premade activity handles the OTP UI for you. Use this if you don't want to build your own OTP screen.

### 1. Create a class-level variable with type `OtpActivitySettings`

Assign it in the activity `onCreate()` method with `fia.otpActivity(this)`.

<details>
<summary>Kotlin</summary>

```kotlin
import com.fazpass.fia.FIAFactory
import com.fazpass.fia.objects.OtpActivitySettings

class MainActivity: AppCompatActivity() {

	private val fia = FIAFactory.getInstance()

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

Call one of the four methods that fits your use case: `login()`, `register()`, `transaction()`, or `forgetPassword()`.

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

With the `transactionId`, check the [Server Documentation](README.Server.md#check-for-user-verified-status) to verify the user.

# Important Notes

> [!CAUTION]
> You must call `otpActivity()` directly inside the activity `onCreate()` method. Calling it elsewhere may crash your app.

> [!CAUTION]
> You must call `otpActivity()` using a `FragmentActivity` or `AppCompatActivity` as context. Otherwise your app might crash.

> [!TIP]
> If you use Jetpack Compose, it's safe to change `ComponentActivity` to `AppCompatActivity`.
> `AppCompatActivity` extends `FragmentActivity`, which extends `ComponentActivity`.
> [See the reference here.](https://stackoverflow.com/a/67364675)
