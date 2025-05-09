# FIA Documentation (iOS)

Documentation on how to use FIA IOS SDK.

# Installation

You can add this package into your project using swift package manager (SPM) or Cocoapods.

## Using Swift Package Manager (SPM)

1. In XCode, click File > Add Packages...
2. Enter this package URL: https://github.com/fazpass/fazpass-intelligence-authentication-ios.git
3. Click Add Package

## Using Cocoapods

1. Open your podfile
2. Add this line in your target app: `pod 'FiaIOS'`
3. run `pod install`

# Getting Started

Before using this SDK, make sure to get the Merchant Key and Merchant App ID from Keypaz Dashboard. 
Check this [Dashboard Documentation](README.Dashboard.md#retrieve-your-merchant-key).

Then, in your XCode, add these capabilities in 'Signing & Capabilities':
1. App Groups (container `group.com.keypaz`)
2. iCloud (service `Key-value storage`)

![XCode Signing & Capabilities](images/xcode-signing-capabilities.png)

# Usage

First, you have to initialize the sdk once.

<details>
<summary>Swift</summary>
 
```swift
import FiaIOS

// get fia instance
let fia = FIAFactory.getInstance()

fia.initialize("YOUR_MERCHANT_KEY", "YOUR_MERCHANT_APP_ID")
```

</details>

There are two ways to request an OTP:
1. <b>With premade view</b><br>If you want to request an OTP without making any view/UI for the OTP activity.
2. <b>With custom-made view</b><br>If you want to make your own view/UI for the OTP activity.

## Request OTP with a premade view

Requesting an OTP with premade view is easier than requesting an OTP with custom-made view. Use this one if you don't want to make your own view/UI.

### 1. Launch the OTP view

To launch the OTP activity, call one of the four methods which fits the purpose of the otp:
- login(phone, callback)
- register(phone, callback)
- transaction(phone, callback)
- forgetPassword(phone, callback)

Whereas phone is user's inputted phone number, and callback is fired when OTP has been validated.

For example, you want to request OTP for login purpose, then the code will be:

<details>
<summary>Swift</summary>
 
```swift
fia.otpView().login("PHONE_NUMBER") { tId in
	// If transactionId is nil, OTP validation has an error.
	guard let transactionId = tId else {
		// handle failed OTP validation here...
		return
	}

	// with the transactionId, check for the user verified status here...
}
```
 
</details>

### 2. Check for user verified status

Get the `transactionId`, then check the [segment down below](#check-for-user-verified-status) on how to check if user has been successfully verified.

## Request OTP with a custom-made view

Use this one if you don't want to make your own view/UI.

### 1. Create a public class to hold a static variable of type `OtpPromise`

<details>
<summary>Swift</summary>

```swift
public class Constants {
	static var otpPromise: OtpPromise? = nil
}
```

</details>

### 2. To request for an OTP, call the `otp()` method then pick the method which fits the OTP purpose

For example, we will use the register method.

<details>
<summary>Swift</summary>

```swift
fia.otp().register("PHONE_NUMBER") { promise in
	if (promise.hasError) {
		let error = promise.error
		// handle failed OTP request here...
		return
	}

	Constants.otpPromise = promise
}
```
 
</details>

### 3. Check which OTP type was being used with `otpPromise.authType`

Here, you can launch between views according to their authentication type as described in the example below.

<details>
<summary>Swift</summary>

```swift
switch (Constants.otpPromise!.authType) {
case .HE:
  // Navigate view to HE view...
  break
case .Miscall:
  // Navigate view to Miscall view...
  break
case .Message:
  // Navigate view to Message view...
  break
case .FIA:
  // Navigate view to FIA view...
  break
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
<summary>Swift</summary>

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

#### Miscall

This OTP will call user's phone number.

User has to fill the last several digits of the caller's phone number. Digit count can be obtained with `digitCount` property.

To validate this auth type, call `validate()` method and fill the inputted user OTP in the parameter.
First callback will be fired if there is an error.
Second callback will be fired if validation has been successful.

<details>
<summary>Swift</summary>

```swift
let digitCount = Constants.otpPromise!.digitCount

// validate OTP method
Constants.otpPromise.validate(
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

#### Message

This OTP will send a Message to user's phone number.

User has to fill the OTP sent to their Sms inbox or any messaging service. Digit count can be obtained with `digitCount` property.

To validate this auth type, call `validate()` method and fill the inputted user OTP in the parameter.
First callback will be fired if there is an error.
Second callback will be fired if validation has been successful.

<details>
<summary>Swift</summary>

```swift
let digitCount = Constants.otpPromise!.digitCount

Constants.otpPromise.validate(
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

#### FIA

It's the OTP Intelligence System. User will not receive an OTP and does not need to input any OTP.

This auth type does not need to be validated. Immediately check for user verified status.

### 4. Check for user verified status

Get the `transactionId` like this:

<details>
<summary>Swift</summary>

```swift
let transactionId = Constants.otpPromise!.transactionId
```
 
</details>

Then check the [segment down below](#check-for-user-verified-status) on how to check if user has been successfully verified.

# Check for user verified status

A successfully validated OTP DOES NOT mean that the user has also been successfully verified. 
To check for user's verified status, check this [Server Documentation](README.Server.md#check-for-user-verified-status).
