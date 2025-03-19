# FIA Documentation (Server)

## Check for user verified status

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

The json key is data -> is_verified. If this value is true, then user is verified.
