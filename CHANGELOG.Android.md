# FIA Android SDK release notes

Version history, newest first, including alpha and development versions.

## 1.3.9 — 2026-11-02

- Security update patch.
- Fixed crashes on Android 10 devices.

## 1.3.8 — 2026-10-22

- Security update patch.

## 1.3.7 — 2026-10-12

- Security update patch.

## 1.3.6 — 2026-09-22

- Improved OTP service compatibility.

## 1.3.5 — 2026-09-17

- Security update patch.
- **API change:** `FIA.initialize` now requires an `Activity`. Update integrations that pass a general or application `Context`.

## 1.3.4 — 2026-09-02

- Security update patch.

## 1.3.3 — 2026-09-01

- Improved OTP service compatibility and error handling.

## 1.3.2 — 2026-08-31

- Added WhatsApp Zero Tap OTP reception through `OtpPromise.listenToWhatsappZeroTap` and the built-in WhatsApp OTP screen.
- Added `FIA.getWhatsappZeroTapAppHashes` and `FIA.isWhatsappZeroTapSupported` to help configure and detect Zero Tap support.
- General maintenance improvements.

## 1.3.1 — 2026-08-26

- General maintenance improvements.

## 1.3.0 — 2026-07-30

- Added user-selected OTP gateways through `OtpGatewayPromise` and gateway selection flows.
- Improved missed-call OTP reliability.
- **API change:** Pass `magicRedirect` to `launchWhatsappForMagicOtp` or `launchWhatsappForMagicLink`. The parameter was removed from OTP request methods.

## 1.2.9 — 2026-07-23

- Security update patch.

## 1.2.8 — 2026-07-16

- Fixed memory leaks and runtime crashes to improve SDK stability.

## 1.2.7 — 2026-07-10

- Improved compatibility for OTP requests without a phone number.

## 1.2.6 — 2026-06-29

- Added `OtpMagicRedirect.MANUAL` for choosing the WhatsApp application during Magic OTP and Magic Link redirects.

## 1.2.5 — 2026-06-19

- Security update patch.
- Improved compatibility with optimized release builds.

## 1.2.4 — 2026-06-15

- Fixed WhatsApp redirect compatibility in optimized release builds.

## 1.2.3 — 2026-06-11

- Added `OtpMagicRedirect` preferences for WhatsApp Magic redirects.
- Improved OTP service compatibility.

## 1.2.2 — 2026-04-28

- Fixed an error when requesting multiple Magic Links.

## 1.2.1 — 2026-04-27

- Improved verification compatibility and missed-call OTP handling.

## 1.2.0 — 2026-04-27

- General SDK maintenance and compatibility improvements.
- Fixed a library compatibility issue.
- Improved compatibility with optimized release builds.

## 1.1.9 — 2026-04-16

- Added Magic verification support without a supplied phone number.
- Security update patch.

## 1.1.8 — 2026-03-03

- Security update patch.

## 1.1.7 — 2025-12-23

- Improved Magic verification compatibility.
- Security update patch.

## 1.1.6 — 2025-11-18

- Security update patch.

## 1.1.5 — 2025-10-28

- Security update patch.
- Improved SDK reliability.

## 1.1.4 — 2025-10-08

- Expanded security feature configuration through `FIA.setFeatures`.
- Improved error diagnostics.
- General maintenance improvements.

## 1.1.3 — 2025-08-22

- **API change:** Removed `OtpAuthType.FIA`. Remove references to this value and update authentication-type handling.
- Improved SDK distribution compatibility.

## 1.1.2 — 2025-08-13

- Fixed release integration and improved network compatibility.

## 1.1.1 — 2025-07-31

- No SDK functionality changes.

## 1.1.0 — 2025-07-23

- Added distinct `SMS`, `Whatsapp`, and `Voice` authentication types.
- **API change:** Replace `OtpAuthType.He` with `OtpAuthType.HE`, and replace `OtpAuthType.Message` handling with the appropriate `SMS` and `Whatsapp` cases.

## 1.0.6 — 2025-07-18

- Improved Android build compatibility.

## 1.0.5 — 2025-07-18

- Improved Kotlin build compatibility.

## 1.0.4 — 2025-07-14

- Added Magic Link authentication and WhatsApp launch support.
- Fixed the WhatsApp Magic Link launcher and the built-in Magic OTP screen.
- Improved SDK distribution compatibility.

## 1.0.3 — 2025-06-19

- Added custom FIA exceptions for connection, initialization, validation, verification, and unknown failures.
- Improved error handling and SDK distribution compatibility.
- Includes the WhatsApp Magic OTP support introduced in `1.0.3-dev.1`.

## 1.0.3-dev.1 — 2025-06-18 — Development

- Added WhatsApp-initiated OTP authentication and its built-in screen.
- **API change:** Renamed the initiation authentication type to `MagicOtp`.
- Fixed header enrichment verification handling.
- Improved network compatibility.

## 1.0.2 — 2025-05-08

- Simplified the OTP workflow.
- **API behavior change:** Validation failures are reported through failure callbacks.
- **API behavior change:** `OtpPromise.digitCount` returns zero when no digit count is available.
- Improved missed-call OTP reliability.
- General maintenance and SDK distribution improvements.

## 1.0.1 — 2025-04-23

- Security update patch.

## 1.0.0 — 2025-04-22

- Initial stable FIA Android SDK release with OTP request and validation APIs and a built-in OTP activity.
- Added staging environment support.
- Improved OTP response handling.

## 1.0.0-alpha-5 — 2025-03-19 — Alpha

- **API change:** Renamed `OtpAuthType.SmsOrWhatsapp` to `Message` and `OtpAuthType.None` to `FIA`. Update references to the renamed authentication types.
- General maintenance improvements.

## 1.0.0-alpha-4 — 2025-03-17 — Alpha

- Initial alpha version in this release history.
- Added OTP request and validation flows, header enrichment verification, missed-call OTP listening, and built-in OTP screens.
