# FIA Documentation (iOS) — Advanced

Full SDK configuration and customization guide for FIA iOS SDK.

For a minimal quickstart without customization, see the [Simple Documentation](README.IOS.SIMPLE.md).

# Installation

You can add this package using Swift Package Manager (SPM) or CocoaPods.

## Using Swift Package Manager (SPM)

1. In Xcode, click **File > Add Packages...**
2. Enter this package URL: `https://github.com/fazpass/fazpass-intelligence-authentication-ios.git`
3. Click **Add Package**

## Using CocoaPods

1. Open your Podfile
2. Add this line inside your target: `pod 'FiaIOS'`
3. Run `pod install`

# Getting Started

Before using this SDK, make sure to get the Merchant Key and Merchant App ID from the Keypaz Dashboard. See the [Dashboard Documentation](README.Dashboard.md#retrieve-your-merchant-key).

In Xcode, add these capabilities under **Signing & Capabilities**:
1. **App Groups** — add container `group.YOUR_INVERTED_DOMAIN`
2. **iCloud** — enable **Key-value storage** service

![Xcode Signing & Capabilities](images/xcode-signing-capabilities.png)

<details>
<summary><h2>Setup Magic OTP</h2></summary>

Add this to your `Info.plist` file:

```xml
<key>LSApplicationQueriesSchemes</key>
<array>
	<string>whatsapp</string>
	<string>whatsappbusiness</string>
</array>
```

</details>

<details>
<summary><h2>Setup Magic Link</h2></summary>

Add this to your `Info.plist` file:

```xml
<key>LSApplicationQueriesSchemes</key>
<array>
	<string>whatsapp</string>
	<string>whatsappbusiness</string>
</array>
```

In **Signing & Capabilities**, add an **Associated Domains** capability and add this entry: `applinks:YOUR_DOMAIN`, where `YOUR_DOMAIN` is your website domain.

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

Finally, in your `AppDelegate`, add this line inside `application(_:continue:restorationHandler:)`:

```swift
func application(_ application: UIApplication, continue userActivity: NSUserActivity, restorationHandler: @escaping ([UIUserActivityRestoring]?) -> Void) -> Bool {
	// add this line
	fia.onMagicLink(userActivity: userActivity)
	return true
}
```

If you are using SwiftUI, add `onOpenURL(perform:)` to your root view instead:

```swift
var body: some Scene {
	WindowGroup {
		ContentView()
			.onOpenURL { url in fia.onMagicLink(url: url.absoluteString) }
	}
}
```

</details>

# Initialize the SDK

Initialize the SDK once before using it.

```swift
import FiaIOS

let fia = FIAFactory.getInstance()

fia.initialize("YOUR_MERCHANT_KEY", "YOUR_MERCHANT_APP_ID", "YOUR_APP_GROUP_ID")
```

# Request OTP with a Custom-Made View

Unlike the premade view approach, you build your own OTP screen and handle each authentication type manually. There are more steps involved.

### 1. Create a public class to hold a static variable of type `OtpPromise`

```swift
public class Constants {
	static var otpPromise: OtpPromise? = nil
}
```

### 2. Request an OTP

Call one of the four methods that fits your use case: `login()`, `register()`, `transaction()`, or `forgetPassword()`.

```swift
fia.otp().register("PHONE_NUMBER") { promise in
	if promise.hasError {
		let error = promise.error
		// handle failed OTP request here...
		return
	}

	Constants.otpPromise = promise
}
```

#### WhatsApp Magic Redirection (Optional)

When using one of the four aforementioned methods, you can pass the optional `magicRedirect` parameter to control which WhatsApp app is used for redirection.

| Value | Description |
|---|---|
| `OtpMagicRedirect.AUTO` | Automatically selects WhatsApp or WhatsApp Business (default) |
| `OtpMagicRedirect.WHATSAPP_NORMAL` | Always redirects to WhatsApp |
| `OtpMagicRedirect.WHATSAPP_BUSINESS` | Always redirects to WhatsApp Business |
| `OtpMagicRedirect.MANUAL` | Shows a dialog letting the user choose which WhatsApp app to use when both WhatsApp and WhatsApp Business are installed |

<details>
<summary>Swift</summary>

```swift
fia.otp(this).login("PHONE_NUMBER", magicRedirect: OtpMagicRedirect.WHATSAPP_NORMAL) { promise in
	if promise.hasError {
		let error = promise.error
		// handle failed OTP request here...
		return@login 
	}

	Constants.otpPromise = promise
}
```

</details>

### 3. Check which OTP type was used with `otpPromise.authType`

Navigate to the appropriate view for each authentication type.

```swift
switch Constants.otpPromise!.authType {
case .HE:
	// Navigate to HE view...
case .Miscall:
	// Navigate to Miscall view...
case .SMS:
	// Navigate to SMS view...
case .Whatsapp:
	// Navigate to WhatsApp view...
case .MagicOtp:
	// Navigate to Magic OTP view...
case .MagicLink:
	// Navigate to Magic Link view...
}
```

There are 6 auth types:

<details>
<summary><h4>HE auth type</h4></summary>

HE (Header Enrichment) uses the network to verify the user. The user does not receive an OTP and does not need to input anything. Only available when the user is connected via a mobile data carrier.

To validate this auth type, call the `validateHE()` method.
The first callback fires if there is an error.
The second callback fires if validation succeeds.

```swift
Constants.otpPromise!.validateHE(
	{ err in
		// handle error here...
	},
	{
		let transactionId = Constants.otpPromise!.transactionId
		// with the transactionId, check for the user verified status here...
	}
)
```

</details>

<details>
<summary><h4>Miscall auth type</h4></summary>

This OTP calls the user's phone number.

The user must enter the last several digits of the caller's phone number. The required digit count can be obtained from the `digitCount` property.

To validate this auth type, call the `validate()` method with the OTP the user entered.
The first callback fires if there is an error.
The second callback fires if validation succeeds.

```swift
let digitCount = Constants.otpPromise!.digitCount

Constants.otpPromise!.validate(
	"USER_INPUTTED_OTP",
	{ err in
		// handle error here...
	},
	{
		let transactionId = Constants.otpPromise!.transactionId
		// with the transactionId, check for the user verified status here...
	}
)
```

</details>

<details>
<summary><h4>SMS auth type</h4></summary>

This OTP sends an SMS to the user's phone number.

The user must enter the OTP from their SMS inbox. The required digit count can be obtained from the `digitCount` property.

To validate this auth type, call the `validate()` method with the OTP the user entered.
The first callback fires if there is an error.
The second callback fires if validation succeeds.

```swift
let digitCount = Constants.otpPromise!.digitCount

Constants.otpPromise!.validate(
	"USER_INPUTTED_OTP",
	{ err in
		// handle error here...
	},
	{
		let transactionId = Constants.otpPromise!.transactionId
		// with the transactionId, check for the user verified status here...
	}
)
```

</details>

<details>
<summary><h4>WhatsApp auth type</h4></summary>

This OTP sends a WhatsApp message to the user's WhatsApp number.

The user must enter the OTP from their WhatsApp inbox. The required digit count can be obtained from the `digitCount` property.

To validate this auth type, call the `validate()` method with the OTP the user entered.
The first callback fires if there is an error.
The second callback fires if validation succeeds.

```swift
let digitCount = Constants.otpPromise!.digitCount

Constants.otpPromise!.validate(
	"USER_INPUTTED_OTP",
	{ err in
		// handle error here...
	},
	{
		let transactionId = Constants.otpPromise!.transactionId
		// with the transactionId, check for the user verified status here...
	}
)
```

</details>

<details>
<summary><h4>Magic OTP auth type</h4></summary>

The user is redirected to WhatsApp and asked to send a prepared message to a specified phone number. They then receive an OTP back in WhatsApp and must enter it in your app.

Call `launchWhatsappForMagicOtp()` to open WhatsApp.
The first callback fires if there is an error when launching WhatsApp.
The second callback fires if WhatsApp launched successfully.

Once WhatsApp has launched, validate the OTP using the `validate()` method — see the [WhatsApp auth type](#whatsapp-auth-type) section above.

```swift
Constants.otpPromise!.launchWhatsappForMagicOtp(
	{ err in
		// handle error here...
	},
	{
		// show the user a text field to enter the incoming OTP,
		// then call Constants.otpPromise!.validate()
	}
)
```

</details>

<details>
<summary><h4>Magic Link auth type</h4></summary>

The user is redirected to WhatsApp and asked to send a prepared message to a specified phone number. They then receive a link in WhatsApp and must tap it to complete verification.

Call `launchWhatsappForMagicLink()` to open WhatsApp.
The first callback fires if there is an error.
The second callback fires if validation succeeds.

```swift
Constants.otpPromise!.launchWhatsappForMagicLink(
	{ err in
		// handle error here...
	},
	{
		let transactionId = Constants.otpPromise!.transactionId
		// with the transactionId, check for the user verified status here...
	}
)
```

</details>

### 4. Check for user verified status

Get the `transactionId`:

```swift
let transactionId = Constants.otpPromise!.transactionId
```

Then see the [Server Documentation](README.Server.md#check-for-user-verified-status) to verify the user.
