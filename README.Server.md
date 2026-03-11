# FIA Documentation (Server)

Documentation on how to use FIA Server Endpoint.

# Check for user verified status

To check for user's verified status, you have to hit our "Check user verified status" API.

REQUEST
- Url: https://api.fazpass.com/v1/otp/fia/verification-status/THE_TRANSACTION_ID
- Method: GET
- Header:
	- Authorization: Bearer Token (Use your 'Merchant Key' as token)

JSON RESPONSE
- status (boolean)
- message (string)
- code (string)
- data :
	- is_verified (boolean)
 	- validated_by (string)

The json key is data -> is_verified. If this value is true, then user is verified.
json key data -> validated_by could be any of these according to the authentication type: he, message, miscall, fia



# Callback (Webhook) Integration Documentation

Our system provides a **Callback** mechanism to monitor user verification status in real-time. Our server will send an HTTP POST request to your registered URL whenever there is a change in the verification status.

---

## 1. Security & Recommendations

We **highly recommend** using this callback method for the following security reasons:
* **Server-to-Server (S2S) Connection**: The data transmission occurs directly between our server and your server.
* **Reduced Client-Side Exposure**: No sensitive information (such as `transaction_id`) is transmitted through the user's device or phone, significantly reducing the risk of interception or manipulation.

---

## 2. Configuration & Activation

To enable this feature, you must contact our **Point of Contact (PIC)** to:
1.  Provide your **Callback URL** (Endpoint) for registration.
2.  Activate the callback service for your account.
3.  Obtain your unique `x-callback-key` for authentication.

---

## 3. Security (Headers)

Every request sent to your endpoint will include the following authentication key in the header:

| Header Key | Description |
| :--- | :--- |
| `x-callback-key` | A unique token used for callback authentication and validation. |

---

## 4. Data Structure (JSON Body)

The following JSON format will be sent to your endpoint:

```
json
{
  "transaction_id": "",
  "phone": "",
  "is_verified": true,
  "message": "",
  "initiate_message": "",
  "verified_by": ""
}
```
### Parameter Definitions

| Parameter | Data Type | Description |
| :--- | :--- | :--- |
| **`transaction_id`** | `String (UUID)` | A unique identifier for the specific verification transaction. |
| **`phone`** | `String` | The user's phone number, formatted in international E.164 standard (e.g., `62812...`). |
| **`is_verified`** | `Boolean` | Indicates the final status of the verification. |
| **`message`** | `String` | The message sent by the system to the user (applicable for all verification types). |
| **`initiate_message`** | `String` | The incoming message sent by the user to our system; specifically used and available only for the **WA Magic** flow. |
| **`verified_by`** | `String` | The specific channel used for verification (e.g., `WA_MAGIC_OTP`, `SMS_OTP`). |
# Whitelist IP

Contact our admin via whatsapp for more information about whitelist IP. To contact our admin, check this [Dashboard Documentation](README.Dashboard.md#whitelist-ip).
