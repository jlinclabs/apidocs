---
layout: default
---

# The A API Server

## Unauthenticated Requests

### Server status

Endpoint:
```
GET /status
```

Response body:
```json
{
  "status": "OK"
}
```


### Signup

Endpoint:
```
POST /api/signup
```

Post body:
```json
{
  "email": "example@example.com",
  "password": "Pa$$w0rd",
  "passwordConfirmation": "Pa$$w0rd",
  "keepMeLoggedIn": true,
  "destinationPath": "/"
}
```

_`destinationPath` is used to store where the user was trying to go
before signing up so they can be redirected there after completing
their signup via the link in their email_

Response body:
```json
{
  "success": true
}
```

At this point an email will be sent to the given email address.
The email will contain a link that will verify the the given email.

This is an exmple of that link:
_https://sandbox.jlinclabs.net/verify/13j0f49fefe2e1b7_

You can verify your email via this API using the [Complete Signup](#complete-signup)
endpoint and the token embeded in the url.

<a name="complete-signup"></a>
### Complete Signup

Endpoint:
```
POST /api/complete_signup
```

Post body:
```json
{
  "emailVerificationCode": "13j0f49fefe2e1b7"
}
```

Response body:
```json
{
  "success": true,
  "sessionId": "305e7cca1f2db7faf6353e00ed043186",
  "email": "newuser002@example.com",
  "destinationPath": "/"
}
```

The `sessionId` provided in the response should be used as the value of the
`Session-Id` header in [authenticated requests](#authenticated-requests).


<a name="login"></a>

### Login

Endpoint:
```
POST /api/login
```

Post body:
```json
{
  "email": "example@example.com",
  "password": "Pa$$w0rd",
  "keepMeLoggedIn":true
}
```

Response body:

```json
{
  "sessionId": "c97cb8cd5cc19177d4db61882ee258df",
  "success": true
}
```

The `sessionId` provided in the response should be used as the value of the
`Session-Id` header in [authenticated requests](#authenticated-requests).

<a name="request-password-reset-email"></a>

### Request Password Reset Email

Endpoint:
```
POST /api/reset_password/request
```

Post body:
```json
{
  "email": "example@example.com"
}
```

Response body:

```json
{
  "success": true
}
```

At this point an email is sent to the user with a link that ends with the
`resetPasswordToken` required by the [Reset Password](#reset-password) endpoint.

<a name="reset-password"></a>

### Reset Password

Endpoint:
```
POST /api/reset_password
```

Post body:
```json
{
  "resetPasswordToken": "GET_THIS_FROM_YOUR_EMAIL",
  "password": "newPassword",
  "passwordConfirmation": "newPassword"
}
```

Response body:

```json
{
  "success": true
}
```

<a name="authenticated-requests"></a>

## Authenticated Requests


Authenticated Requests require that you define the header `Session-Id`.
The value should be the `sessionId` provided by a [Login](#login) request.

<a name="logout"></a>

### Logout

Endpoint:
```
POST /api/logout
```

Post body:
```json
{}
```

Response body:

```json
{
  "success": true
}
```

### Get Default Account Data

Endpoint:
```
GET /api/default-account-data
```

Response body:

```json
{
  "success": true,
  "defaultAccountData": {
    "shared_personal_data": {},
    "personal_data": {},
    "consents": {},
    "communication_channels": {}
  }
}
```


### Update Default Account Data

Endpoint:
```
POST /api/default-account-data
```

Post body:
```json
{
  "defaultAccountData": {
    "shared_personal_data": {},
    "personal_data": {},
    "consents": {},
    "communication_channels": {}
  }
}
```

Response body:
```json
{
  "success": true,
  "defaultAccountData": {
    "shared_personal_data": {},
    "personal_data": {},
    "consents": {},
    "communication_channels": {}
  }
}
```


### Get Overview

Endpoint:
```
GET /api/overview
```

Response body:

```json
{
  "success": true,
  "sisas": [],
  "myOrganizations": {},
  "recentSisaEvents": [],
  "defaultAccountData": {
    "shared_personal_data": {},
    "personal_data": {},
    "consents": {},
    "communication_channels": {}
  },
  "preferences": {}
}
```

## Update Preferences

Endpoint:
```
POST /api/preferences
```

Post body:
```json
{
  "preferences": {
    "skip_future_view_sisas": true
  }
}
```

Response body:
```json
{
  "success": true,
  "preferences": {
    "skip_future_view_sisas": true
  }
}
```

### Get All Organizations

Endpoint:
```
GET /api/organizations
```

Response body:

```json
{
  "success": true,
  "organizations": [
    {
      "apikey": "jlinclabs",
      "public_key": "",
      "logo": "",
      "icon": "",
      "banner": "",
      "name": "JLINC Labs",
      "dpo_email": "dpo@jlinclabs.com",
      "domain": "jlinclabs.com",
      "contact_phone": "",
      "contact_phone_2": "",
      "address": "",
      "city": "",
      "post_code": "",
      "country": "",
      "consents": {},
      "communication_channels": {},
      "requested_data": {},
      "description": "",
      "upsell_description": "",
      "state": "",
      "public": true,
      "marketplace_tags": []
    }
  ]
}
```

### Get A Single Organization

Endpoint:
```
GET /api/organizations/:organizationApikey
```

Response body:

```json
{
  "success": true,
  "organization": {
    "apikey": "jlinclabs",
    "public_key": "",
    "logo": "",
    "icon": "",
    "banner": "",
    "name": "JLINC Labs",
    "dpo_email": "dpo@jlinclabs.com",
    "domain": "jlinclabs.com",
    "contact_phone": "",
    "contact_phone_2": "",
    "address": "",
    "city": "",
    "post_code": "",
    "country": "",
    "consents": {},
    "communication_channels": {},
    "requested_data": {},
    "description": "",
    "upsell_description": "",
    "state": "",
    "public": true,
    "marketplace_tags": []
  }
}
```

### Sign A SISA with an Organization

Endpoint:
```
POST /api/sign_sisa
```

Post body:
```json
{
  "organizationApikey": "jlinclabs"
}
```

Response body:
```json
{
  "success": true,
  "sisa": {
    "@context": "https://protocol.jlinc.org/context/jlinc-v6.jsonld",
    "sisaId": "",
    "acceptedSisa": {
      "@context": "https://protocol.jlinc.org/context/jlinc-v6.jsonld",
      "rightsHolderSigType": "sha256:ed25519",
      "rightsHolderId": "",
      "rightsHolderSig": "",
      "createdAt": "",
      "offeredSisa": {
        "@context": "https://protocol.jlinc.org/context/jlinc-v6.jsonld",
        "dataCustodianSigType": "sha256:ed25519",
        "dataCustodianId": "",
        "dataCustodianSig": "",
        "createdAt": "",
        "agreement": {
          "@context": "https://protocol.jlinc.org/context/jlinc-v6.jsonld",
          "jlincId": "",
          "agreementURI": "https://sisa.jlinc.org/v1/3b39160c2b9ae7b2ef81c3311c7924f1c4d4fa9ca47cfe7c840c9852b50d68d5"
        }
      }
    },
    "organizationApikey": "jlinclabs"
  },
  "organizationAccountData": {
    "consents": {},
    "communication_channels": {},
    "shared_personal_data": {},
    "personal_data": {}
  },
  "organization": {}
}
```

_For more information in the `sisa` object please see the
<a href="‭https://protocol.jlinc.org">JLINC Protocol</a> documentation
‭https://protocol.jlinc.org_

### Get Your Account Data for an Organization

Endpoint:
```
GET /api/organizations/accountData/:organizationApikey
```

Response body:
```json
{
  "success": true,
  "organizationAccountData": {
    "shared_personal_data": {},
    "personal_data": {},
    "consents": {},
    "communication_channels": {}
  }
}
```

### Update Your Account Data for an Organization

Endpoint:
```
POST /api/organizations/accountData
```

Post body:
```json
{
  "organizationApikeys": [
    "organizationApikeyOne",
    "organizationApikeyTwo"
  ],
  "changes": {
    "shared_personal_data": {},
    "personal_data": {},
    "consents": {},
    "communication_channels": {}
  }
}
```

Response body:
```json
{
  "success": true,
  "organizationApikeyOne": {
    "organizationAccountData": {
      "shared_personal_data": {},
      "personal_data": {},
      "consents": {},
      "communication_channels": {}
    },
    "sisaEvents": []
  },
  "organizationApikeyTwo": {
    "organizationAccountData": {
      "shared_personal_data": {},
      "personal_data": {},
      "consents": {},
      "communication_channels": {}
    },
    "sisaEvents": []
  },
}
```

### Get Organization SISA Events

Endpoint:
```
GET /api/organizations/sisa_events/:organizationApikey?page=1
```

Params:
```
page (number) (optional)
```

Response body:
```json
{
  "success": true,
  "organizationSisaEvents": []
}
```


### Get Organization Feed Posts

Endpoint:
```
GET /api/organization/:organizationApikey/feed/posts?before=2018-11-07T22%3A19%3A26.235Z
```

Params:
```
before (url encoded ISO Date String) (optional)
```

Response body:
```json
{
  "success": true,
  "posts": []
}
```

### Create Organization Feed Post

Endpoint:
```
POST /api/organization/:organizationApikey/feed/posts/create
```

Post body:
```json
{
  "post": {
    "title": "This is the post title",
    "body": "This is the post body"
  }
}
```

Response body:
```json
{
  "success": true,
  "post": {
    "uid": "69ac21843f5836b975ca1c8c31e800f2",
    "organizationApikey": "jlinclabs",
    "title": "This is the post title",
    "body": "This is the post body",
    "createdAt": "2018-11-08T00:54:27.389Z",
    "organizationPost": false
  }
}
```


### Delete Organization Feed Post

Endpoint:
```
POST /api/organization/:organizationApikey/feed/posts/delete
```

Post body:
```json
{
  "feedPostUid": "69ac21843f5836b975ca1c8c31e800f2"
}
```

Response body:
```json
{
  "success": true
}
```

### Get Buying Interests


Endpoint:
```
GET /api/buying_interest
```

Response body:
```json
{
  "success": true,
  "buyingInterests": []
}
```

### Create A Buying Interest


Endpoint:
```
POST /api/buying_interest/create
```

Post body:
```json
{
  "buyingInterest": {
    "industry": "Energy",
    "currency": "$",
    "description": "i fine pair of shoes",
    "location": "San Francisco",
    "price_low": 10000,
    "price_high": 20000,
    "beginning_date": "2018-11-08",
    "end_date": "2020-12-31",
    "brands": [
      "Liberty Utilities",
      "Ecotricity",
      "UK Power Networks"
    ],
    "tags": [
      "Fixed rate",
      "Green energy",
      "Variable rate"
    ]
  }
}
```

Response body:
```json
{
  "success": true,
  "buyingInterest": {}
}
```



### Delete A Buying Interest

Endpoint:
```
POST /api/buying_interest/delete
```

Post body:
```json
{
  "buyingInterestUid": "fd324158c148efe7e88498cd089055e4"
}
```

Response body:
```json
{
  "success": true
}
```









































