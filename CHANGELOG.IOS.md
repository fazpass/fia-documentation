# FIA iOS SDK changelog

Version history, newest first.

## 1.3.6 — 2026-10-12

- Security update patch.

## 1.3.5 — 2026-09-22

- Updated OTP request compatibility with the FIA service for automatic and manual gateway flows.

## 1.3.4 — 2026-09-17

- Security update patch.

## 1.3.3 — 2026-09-01

- Improved compatibility with FIA service responses across OTP requests, gateway selection, and validation.
- Validation callbacks now treat unsuccessful server results as failures even when the network request succeeds, and return the server's error message when available.

## 1.3.2 — 2026-08-26

- General maintenance.

## 1.3.1 — 2026-08-19

- Added `OtpGatewayPromise.activityId: String` and `OtpGatewayPromise.isBlocked: Bool` to expose the device-analysis activity and blocking result during manual gateway selection.
- Manual gateway requests now use the current permission state when `OtpGatewayPromise.pick(_:_:)` is called.

## 1.3.0 — 2026-08-05

- Added `FIA.otpManual() -> OtpUserSettings` for choosing an OTP gateway in your own interface, and `FIA.otpViewManual() -> OtpViewUserSettings` for the SDK's built-in gateway picker. Both offer `login`, `register`, `transaction`, and `forgetPassword`, with optional `additionalInfo`.
- Added `OtpGateway.number`, `OtpGateway.name`, and `OtpGateway.init(number:name:)`, plus `OtpGatewayPromise.hasError`, `error`, `isAuthenticated`, `transactionId`, and `gateways`.
- For manual flows, check `hasError` first. If `isAuthenticated` is true, use `transactionId`; otherwise pass a gateway's `number` to `pick(_ number: Int, _ callback: @escaping (OtpPromise) -> Void)` and validate the returned `OtpPromise` using the existing channel APIs.
- Moved WhatsApp redirect selection to `OtpPromise.launchWhatsappForMagicOtp(_ redirect: OtpMagicRedirect = .AUTO, _ onFailed: ((Error) -> Void)? = nil, _ onSuccess: (() -> Void)? = nil)` and `launchWhatsappForMagicLink(_ redirect: OtpMagicRedirect = .AUTO, _ onFailed: ((Error) -> Void)? = nil, _ onValidated: (() -> Void)? = nil)`.
- Migration: remove `redirect:` from `login`, `register`, `transaction`, and `forgetPassword` calls on `OtpSettings` and `OtpViewSettings`. With custom views, pass the redirect as the first argument to the WhatsApp launch method, before any positional callbacks. Both built-in OTP flows now use `.MANUAL` WhatsApp selection.

## 1.2.5 — 2026-06-29

- Added `OtpMagicRedirect.MANUAL` to let users choose between WhatsApp and WhatsApp Business when both are installed. If only one is installed, it opens directly. Cancelling the chooser does not invoke a launch callback.
- Fixed successful Magic OTP launching to invoke the `onSuccess` callback supplied to `OtpPromise.launchWhatsappForMagicOtp`.

## 1.2.4 — 2026-06-18

- Added WhatsApp Business support for Magic OTP and Magic Link, including the SDK's built-in OTP screens.
- Added `OtpMagicRedirect.WHATSAPP_NORMAL`, `.WHATSAPP_BUSINESS`, and `.AUTO`. Automatic selection prefers normal WhatsApp when available and otherwise attempts WhatsApp Business.
- Added `redirect: OtpMagicRedirect = .AUTO` and labeled `additionalInfo: [String: String]? = nil` arguments to `login`, `register`, `transaction`, and `forgetPassword` on both `OtpSettings` and `OtpViewSettings`.
- Migration: change positional extra-information calls such as `login(phone, info, callback)` to `login(phone, additionalInfo: info, callback)`. Calls that omit extra information can continue using the default arguments.
- Security update patch.

## 1.2.3 — 2026-05-05

- Added support for phone-less OTP requests in supported flows. Pass an empty string (`""`) as the phone argument; the public parameter remains `String`.
- Added extra information for server-configured message templates to custom and built-in OTP flows. The `login`, `register`, `transaction`, and `forgetPassword` methods accept an additional unlabeled `[String: String]?` argument before the callback; existing phone-and-callback calls remain supported.

## 1.2.2 — 2026-04-16

- Security update patch.

## 1.2.1 — 2026-03-09

- Improved error descriptions for device-security and biometric failures.
- Corrected `FIAError.errorDescription` to return `String?`, matching `LocalizedError`. Handle the optional value when accessing it directly, or use `localizedDescription`.
- Security update patch.

## 1.2.0 — 2026-01-07

- Initialization now requires an App Group identifier: `FIA.initialize(_ merchantKey: String, _ merchantAppId: String, _ groupId: String)`.
- Migration: replace two-argument initialization with `initialize(merchantKey, merchantAppId, groupId)`, using your application's configured App Group identifier. Call initialization once at application launch.
- Updated Magic Link completion handling for supported return links. Continue checking verification status on your server after `onValidated`.
- Security update patch.

## 1.1.4 — 2025-12-16

- Improved WhatsApp launch-link compatibility for Magic OTP and Magic Link.
- Added `FIA.onMagicLink(url: String)` for forwarding URLs from SwiftUI's `onOpenURL`, alongside `onMagicLink(userActivity:)`.
- Changed `FIAError.verificationFailed(message:)` to accept `String?`, preserving supplied messages and using a default when absent. Added an explicit description for uninitialized use.
- At this version, `FIAError.errorDescription` returns `String`; version `1.2.1` restores its optional return type for `LocalizedError` compatibility.

## 1.1.3 — 2025-10-30

- Added `OtpPromise.activityId: String` and `OtpPromise.isBlocked: Bool` to expose the device-analysis activity and blocking result alongside OTP state.
- Security update patch.

## 1.1.2 — 2025-10-09

- Added support for optional device-security checks and access to device-analysis capabilities.
- Improved request and validation error handling, preserving descriptive failures for unsuccessful requests and unavailable WhatsApp installations.
- Changed `FIAError.verificationFailed` to `verificationFailed(message: String)`. Supply a message when constructing this case and handle its associated value when inspecting it.

## 1.1.1 — 2025-08-23

- Removed `OtpAuthType.FIA`. Remove `.FIA` cases from authentication switches; unrecognized authentication values fall back to `.HE`.
- Fixed `OtpPromise.validateHE` to complete through `onValidated` when no network verification step is needed. Continue checking verification status on your server after the callback.

## 1.1.0 — 2025-07-23

- Replaced `OtpAuthType.Message` with `.SMS` and added `.Whatsapp` and `.Voice`. Update authentication switches to handle these channels; unrecognized authentication values fall back to `.HE`.
- Extended `OtpPromise.validate` to SMS, WhatsApp, and voice OTP alongside Miscall and Magic OTP.
- Added built-in WhatsApp and voice OTP screens, with WhatsApp instructions for the Magic OTP follow-up step.
- Changed `FIAError.badConnection` to `badConnection(message: String? = nil)` to preserve descriptive connection errors. Update explicit enum construction and associated-value handling as needed.

## 1.0.1 — 2025-07-23

- Added `OtpAuthType.MagicOtp` and `.MagicLink`, with built-in SDK screens for both flows.
- Added `OtpPromise.launchWhatsappForMagicOtp(_ onFailed: ((Error) -> Void)? = nil, _ onSuccess: (() -> Void)? = nil)` and `launchWhatsappForMagicLink(_ onFailed: ((Error) -> Void)? = nil, _ onValidated: (() -> Void)? = nil)`.
- Added `FIA.onMagicLink(userActivity: NSUserActivity)`. Forward Universal Link events from the app delegate's `application(continue:)` method to support Magic Link authentication.
- Extended `OtpPromise.validate` to Magic OTP. `OtpPromise.digitCount` now returns `0` when no count is available, replacing `-1`.
- Security update patch.

## 1.0.0 — 2025-06-26

- Initial tagged FIA iOS SDK version, with iOS 14+ support and Swift Package Manager/CocoaPods distribution.
- Added `FIAFactory.getInstance()` and required `FIA.initialize(_ merchantKey: String, _ merchantAppId: String)`, called once at application launch.
- Added `FIA.otp()` for custom interfaces and `FIA.otpView()` for built-in OTP screens, supporting `login`, `register`, `transaction`, and `forgetPassword`.
- Added `OtpAuthType.HE`, `.Miscall`, `.Message`, and `.FIA`, plus `OtpPromise.validate` and `validateHE`, transaction IDs, digit counts, and error results.
- Added `FIAError.badConnection`, `illegalValidate(message:)`, `verificationFailed`, `uninitialized`, and `unknownError(message:cause:)`.
- Custom-interface callbacks return `OtpPromise`; built-in-view callbacks return an optional transaction ID. Validation completion does not establish final authentication: check verification status on your server.
