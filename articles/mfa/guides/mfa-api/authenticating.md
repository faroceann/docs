---
title: Associate One-Time Password Authenticators
description: Configure your application so users can self-associate one-time password (OTP) authenticators.
topics:
  - mfa
  - mfa-api
  - mfa-authenticators
  - otp
contentType:
  - how-to
  - reference
useCase:
  - customize-mfa
---
# Authenticating With Resource Owner Password Grant and MFA

When you use the [Resource Owner Password Grant](/api-auth/tutorials/password-grant)to authenticate users while MFA is enabled, you need to use the MFA API to complete the authentication flow. The required steps are described below.

<%= include('../../_includes/_authenticator-before-start') %>

## 1. Authenticate with `/oauth/token`

Call the `/oauth/token` endpoint with the user's username/password. The call will return an `mfa_required` error and a `mfa_token`.

```har
{
"method": "POST",
"url": "https://${account.namespace}/oauth/token",
"headers": [
    { "name": "Content-Type", "value": "application/x-www-form-urlencoded" }
],
"postData": {
    "mimeType": "application/x-www-form-urlencoded",
    "params": [
    {
        "name": "grant_type",
        "value": "password"
    },
    {
        "name": "username",
        "value": "user@example.com"
    },
    {
        "name": "password",
        "value": "pwd"
    },
    {
        "name": "client_id",
        "value": "${account.clientId}"
    },
    {
        "name": "client_secret",
        "value": "YOUR_CLIENT_SECRET"
    },
    {
        "name": "audience",
        "value": "https://someapi.com/api"
    },
    {
        "name": "scope",
        "value": "openid profile read:sample"
    }
    ]
}
}
```

The response will include the `mfa_required` error and a `mfa_token`:

```json
{
    "error": "mfa_required",
    "error_description": "Multifactor authentication required",
    "mfa_token": "Fe26...Ha"
}
```

## 2. Retrieve the enrolled authenticators

You'll need to find out if the user has an MFA factor enrolled or not. To do that, you should call the `/mfa/authenticators` endpoint, using the MFA token obtained in the previous step.

```
{
    "method": "GET",
	"url": "https://${account.namespace}/mfa/authenticators",
    "headers": [
    { "name": "Authorization", "value": "Bearer MFA_TOKEN" },
    { "name": "Content-Type", "value": "application/x-www-form-urlencoded" }
  ]
}
```

You will get an array with the available authenticators. The array will be empty if the user did not enroll any factor. 

```json
[
    {
        "id": "recovery-code|dev_O4KYL4FtcLAVRsCl",
        "authenticator_type": "recovery-code",
        "active": true
    },
    {
        "id": "email|dev_NU1Ofuw3Cw0XCt5x",
        "authenticator_type": "oob",
        "active": true,
        "oob_channel": "email",
        "name": "email@address.com"
    },
]
```

You will need to use the authenticator's `id` when challenging the user for MFA.

## 3. Enroll an MFA factor

If the user is not enrolled in MFA, enroll it using the `/mfa/associate` endpoint: 

- [Enrolling with SMS](/mfa/guides/mfa-api-sms#enrolling-with-sms).
- [Enrolling with OTP](/mfa/guides/mfa-api-sms#enrolling-with-otp).
- [Enrolling with Push](/mfa/guides/mfa-api-sms#enrolling-with-push).
- [Enrolling with Email](/mfa/guides/mfa-api-sms#enrolling-with-email).

## 4. Challenge the user with MFA

If the user is already enrolled in MFA, you need to challenge the user with one existing factor. Use the `authenticator_id` returned by the `/mfa/authenticators` endpoint when calling the `/mfa/challenge` endpoint. After the challenge is completed, call `/oauth/token` again to finalize the authentication flow and get the authentication tokens.

- [Challenging with SMS](/mfa/guides/mfa-api/sms#challenging-with-sms).
- [Challenging with OTP](/mfa/guides/mfa-api/otp#challenging-with-otp).
- [Challenging with Push](/mfa/guides/mfa-api/push#challenging-with-push).
- [Challenging with Email](/mfa/guides/mfa-api/email#challenging-with-email).
- [Challenging with Recovery Code](/mfa/guides/mfa-api/recovery-code).

## Keep reading

* [Managing MFA Enrollments](/mfa/guides/mfa-api/manage)
