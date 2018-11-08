---
layout: default
---
## The B server

### Server status

Endpoint:
```
GET http://sandbox.jlinclabs.net/status
```

If the server is responding, returns:

```json
{
  "status": "OK"
}
```
<a name="create-organization"></a>
### Create organization

Endpoint:
```
POST http://sandbox.jlinclabs.net/api/signup
```

Generates a signup token and sends an email to the given email address with a verification link.

Requires either a `stripeToken` id (see [Stripe API docs: tokens](https://stripe.com/docs/api/tokens)) or an `inviteToken`. An `inviteToken` is created by running the `createOrganizationAdminInviteToken` script on the B Server. It should then added as a param to an invite url that can be sent to a prospective organization admin, example: `http://sandbox.b-portal.jlinclabs.net?it=V7i1ldl8HoeVFBDtL9paUbS2kFl3rlB`. If an `inviteToken` is provided, it should first be [verified](#verify-invite-token) before posting to `api/signup`.

Post body:
```json
{
  "email": "neworg@example.com",
  "stripeToken": "tok_1CR5JRFNXsZIJI8lCl1l04S4",
  "inviteToken:": "d1830fcda87ae8993039694e6f7aea95"
}
```

A successful response returns success `true`.

Response body:
```json
{
 "success": true
}
```

Example signup link sent in the verification email: `http://sandbox.b-portal.jlinclabs.net/signup/eV7i1ldl8HoeVFBDtL9paUbS2kFl3rlB`.

The signup token is the last parameter of the signup url above. You can check the validity of the signup token using the endpoint below.

Endpoint:
```
POST http://sandbox.jlinclabs.net/api/verify_signup
```

Validates the signup token.

Post body:
```json
{
  "signupToken": "eV7i1ldl8HoeVFBDtL9paUbS2kFl3rlB",
}
```

Returns the email associated with the signup token.

Response body:
```json
{
  "success": true,
  "email": "neworg@example.com"
}
```

Once the signup token has been validated, the organization can be created.

Endpoint:
```
POST http://sandbox.jlinclabs.net/api/complete_signup
```

Creates an organization, organization admin, and stores a session.

Post body:
```json
{
  "name": "Bob Co",
  "email": "neworg@example.com",
  "password": "examplepassword",
  "passwordConfirmation": "examplepassword",
  "signupToken": "eV7i1ldl8HoeVFBDtL9paUbS2kFl3rlB",
  "salesforce": true,
  "keepMeLoggedIn": true
}
```

`salesforce` is a boolean representing whether or not the organization has a Salesforce connection. Defaults to `false`.

Returns a session ID and initial organization details.

Response body:
```json
{
  "success": true,
  "sessionId": "5b2dbdc40a22ecc8ca2e12c3da0f0319",
  "organization": {
    "apikey": "bobco",
    "logo": null,
    "icon": null,
    "name": "Bob Co",
    "dpo_email": "neworg@example.com",
    "domain": null,
    "contact_phone": null,
    "contact_phone_2": null,
    "address": null,
    "city": null,
    "post_code": null,
    "country": null,
    "communication_channels": {},
    "requested_data": {},
    "consents": {},
    "salesforce": false,
    "description": null,
    "state": null,
    "public": false,
    "banner": null,
    "upsell_description": null,
    "public_allowed": false,
    "published": false,
    "marketplace_tags": [],
    "industries": [],
    "brands": [],
    "public_key": "Vi6djLwwoiga_XWE93sdbYPbgP2jxjCjHDNfq5qFcRg",
    "dcust_id": "Vi6djLwwoiga_XWE93sdbYPbgP2jxjCjHDNfq5qFcRg",
    "api_secret": "457b855bbfb36a3bd5df045066f34de40b2db65314cef8b9d83ae272f40fc127",
    "vendor_name": "bobco"
  }
}
```

<a name="login"></a>
### Login

Endpoint:
```
POST http://sandbox.jlinclabs.net/api/login
```

Post body:
```json
{
  "email": "neworg@example.com",
  "password": "examplepassword",
  "keepMeLoggedIn": true
}
```

A successful login returns a session ID:

Response body:
```json
{
  "success": true,
  "sessionID": "5e1668d7fffaadabf0474088e..."
}
```

The sessionId provided in the response should be used as the value of the
`Session-Id` header of [authenticated requests](#authenticated-requests) section that appears later in this document.

<a name="verify-invite-token"></a>
### Verify invite token

Endpoint:
```
POST http://sandbox.jlinclabs.net/api/verify_invite_token
```

Post body:
``` json
{
  "email": "neworg@example.com",
  "inviteToken": "d1830fcda87ae8993039694e6f7aea95"
}
```

Response body:
```json
{
  "success": true
}
```

<a name="reset-password"></a>
### Reset password

Sends an email with a reset password link to the given email address.

Endpoint:
```
POST http://sandbox.jlinclabs.net/api/reset_password/request
```

Post body:
``` json
{
  "email": "neworg@example.com"
}
```

Response body:
```json
{
  "success": true
}
```

Example link sent in request password email: `http://sandbox.b-portal.jlinclabs.net/reset-password/db2506e27894aee3216273b49910062d`. The last parameter of this link is the `resetPasswordToken`.

Before a password is reset, the `resetPasswordToken` should be verified.

Endpoint:
```
GET http://sandbox.jlinclabs.net/api/reset_password/verify/:resetPasswordToken
```

Checks if the `resetPasswordToken` is valid and returns the email associated with the `resetPasswordToken`.

Response body:
```json
{
  "success": true,
  "email": "neworg@example.com"
}
```

Now the password can be reset.

Endpoint:
```
POST http://sandbox.jlinclabs.net/api/reset_password
```

Post body:
``` json
{
  "resetPasswordToken": "db2506e27894aee3216273b49910062d",
  "password": "passwordeagle",
  "passwordConfirmation": "passwordeagle"
}
```

When the password has been successfully reset.

Response body:
```json
{
  "success": true
}
```

<a name="authenticated-requests"></a>
## Authenticated Requests

These endpoints require requests to have set a `Session-Id` header with a valid session ID value.

<a name="get-organization-profile"></a>
### Get organization profile

Endpoint:
```
GET http://sandbox.jlinclabs.net/api/orgs/profile
```

Response body:
```json
{
  "success": true,
  "organizationProfile": {
    "apikey": "bobco",
    "logo": "",
    "icon": "",
    "name": "Bob Co",
    "dpo_email": "neworg@example.com",
    "domain": "bobco.com",
    "contact_phone": "111-222-3333",
    "contact_phone_2": "222-333-4444",
    "address": "1 Bob St",
    "city": "San Francisco",
    "state": "California",
    "post_code": "94606",
    "country": "United States",
    "communication_channels": {},
    "requested_data": {},
    "consents": {},
    "salesforce": false,
    "description": "Voluptates sit enim impedit eligendi in.",
    "public": true,
    "banner": "http://imgurl.jpg",
    "upsell_description": "Corporis sed temporibus sint.",
    "public_allowed": true,
    "published": true,
    "marketplace_tags": [],
    "industries": [],
    "brands": []
  }
}
```

<a name="update-organization-profile"></a>
### Update organization profile

Endpoint:
```
POST http://sandbox.jlinclabs.net/api/orgs/profile
```

Post body:
```json
{
  "organizationProfile": {
    "logo": "",
    "dpo_email": "bobco@newemail.org",
    "domain": "newedomain.com",
    "upsell_description": "New description.",
    "consents": {},
  }
}
```

Returns the updated organization profile.

Response body:
```json
{
  "success": true,
  "organization_profile": {
    "apikey": "bobco",
    "logo": "",
    "icon": "",
    "name": "Bob Co",
    "dpo_email": "bobco@newemail.org",
    "domain": "newedomain.com",
    "contact_phone": "111-222-3333",
    "contact_phone_2": "222-333-4444",
    "address": "1 Bob St",
    "city": "San Francisco",
    "state": "California",
    "post_code": "94606",
    "country": "United States",
    "communication_channels": {},
    "requested_data": {},
    "consents": {},
    "salesforce": false,
    "description": "Voluptates sit enim impedit eligendi in.",
    "public": true,
    "banner": "http://imgurl.jpg",
    "upsell_description": "New description.",
    "public_allowed": true,
    "published": true,
    "marketplace_tags": [],
    "industries": [],
    "brands": []
  }
}
```

<a name="logout"></a>
### Logout

Endpoint:
```
POST http://sandbox.jlinclabs.net/api/logout
```

If the session exists and has been destroyed.

Response body:
```json
{
  "success": true
}
```

<a name="get-consent-report"></a>
### Get consent report

Endpoint:
```
GET http://sandbox.jlinclabs.net/api/orgs/consent_report
```

Returns a consent report - an array of objects each representing an end-user, sorted by newest SISA permission event date, listing the permissions given by the end user and the date at which each of these permissions was granted.

```json
{
  "success": true,
  "consentReport": [
    {
      "id": "004Lxn3w_7-A5JVyJp9elhD8ejdh3TTP3YL0fnE62N",
      "lastUpdated": "2018-11-07T11:44:19-08:00",
      "consents": {},
      "personal_data": {
        "email": "example-end-user-1@example.com"
      }
    },
    {
      "id": "9GFWGz1tbJ8YMF1uJAiny0bqs5RiAV2rjv4wym8U5WM",
      "lastUpdated": "2018-11-07T11:44:19-08:00",
      "consents": {},
      "personal_data": {
        "email": "example-end-user-2@example.com"
      }
    }
  ]
}
```

<a name="onboard-end-user"></a>
### Onboard end-user

This is an API to create a signup link that BobCo can send to Alice, inviting her to take charge of the contact information he has about her, and set her preferences about what he can do with it.

Endpoint:
```
POST http://sandbox.jlinclabs.net/orgs/invite_end_user
```

`email` is required, and must be unique in BobCo's account. All other data fields are optional. Any other fields that might be included will be ignored. From time to time new optional data fields may be added.

Post body:
```json
{
  "endUserPersonalData": {
    "email": "alice@example.com",
	  "firstname": "Alice",
	  "lastname": "McPerson",
	  "fullname": "Alice McPerson",
	  "mailingstreet": "123 Main Street",
	  "mailingcity": "Oakland",
	  "mailingstate": "CA",
	  "mailingpostalcode": "01234",
	  "mailingcountry": "US",
	  "homephone": "555-111-4444",
	  "mobilephone": "555-111-2222"
  }
}
```

On success a signup link will be returned:

Response body:
```json
{
  "success": true,
  "signup": "http://a-portal.jlinc.test/regauth/gbobcogc3b1b27093b2d95eb23d3b381b3542bc"
}
```

This link should be sent to Alice to register her account with BobCo.

<a name="create-multiple-end-user-invites-from-a-csv"></a>
### Create multiple end-user invites from a CSV

Endpoint:
```
POST http://sandbox.jlinclabs.net/api/orgs/invite_end_user_batch
```

Requires submitting a multipart/form-data form with a text `session_id` field containing
a valid organization admin session ID, and field of type "file" named `emailListCSVStream` containing the CSV file.

The CSV file can contain columns with the following headers:
* Email (required)
* First Name
* Surname
* Salutation
* Title
* Birthdate
* Gender
* Street
* City
* State / Province / County
* Postcode / Zip code
* Country
* Home Phone Number (Land Line)
* Mobile Phone Number
* Fax Number
* Business Name
* Business Industry
* Business Street
* Business City
* Business Country
* Business Postcode / Zip code
* Business Phone Number

Returns an `invites.csv` file with a signup url for each row with a valid unique email.

<a name="get-all-end-user-invites"></a>
### Get all end-user invites

Gets a list of end-user invites in batches of 10. Takes `page` query param that determines which set of 10 invites to retrieve.

Endpoint:
```
GET http://sandbox.jlinclabs.net/api/end_user_invites?page=1
```

Params:
```
page (number)
```

Returns an array of objects with invite urls and associated data.

Response body:
```json
{
  "success": true,
  "endUserInvites": [
    {
      "email": "user1@example.com",
      "invite_token": "0667841d8babb10ae98f4ba2426f75f2",
      "organization_apikey": "bobco",
      "external_id__c": "jlinc-s33xHBpBrc59kh-rSxVQPBMgMKzyHQTe",
      "personal_data": {
        "email": "user1@example.com",
        "firstname": "alice",
        "lastname": "ecila",
        "homephone": "555-111-4444",
        "mailingcity": "Oakland",
        "mobilephone": "555-111-2222",
        "mailingstate": "CA",
        "mailingstreet": "123 Main Street",
        "mailingcountry": "US",
        "mailingpostalcode": "01234"
      },
      "created": "2018-11-07T20:53:14.322Z",
      "rights_holder_public_key": null,
      "used_at": null,
      "invite_url": "http://a-portal.jlinc.test/regauth/gbobcog08300400501981586g0667841d8babb10ae98f4ba2426f75f2"
    },
    {
      "email": "user2@example.com",
      "invite_token": "0667841d8babb10ae98f4ba2426n75f2",
      "organization_apikey": "bobco",
      "external_id__c": "jlinc-s33xHBpBrc59kh-rSpVQPBMgMKzyHQTe",
      "personal_data": {
        "email": "user1@example.com",
        "firstname": "carol",
        "lastname": "lorac",
        "homephone": "123-234-3456",
        "mailingcity": "Oakland",
        "mobilephone": "111-111-2222",
        "mailingstate": "CA",
        "mailingstreet": "121 Main Street",
        "mailingcountry": "US",
        "mailingpostalcode": "01237"
      },
      "created": "2018-11-07T20:53:14.322Z",
      "rights_holder_public_key": null,
      "used_at": null,
      "invite_url": "http://a-portal.jlinc.test/regauth/gbobcog08300400501981586g1b8df697e27dce7b08dbae96cd69e1b2"
    }
  ]
}
```

<a name="delete-end-user-invite"></a>
### Delete end-user invite

Deletes the end-user invite matching the given `inviteToken`.

Endpoint:
```
POST http://sandbox.jlinclabs.net/api/delete_end_user_invite
```

Post body:
```json
{
  "inviteToken": "08300400501981586g1b8df697e27dce7b08dbae96cd69e1b2"
}
```

If a matching invite exists and has been deleted.

Response body:
```json
{
  "success": true
}
```

<a name="get-all-end-user-account-data"></a>
### Get all end-user account data

Endpoint:
```
GET http://sandbox.jlinclabs.net/api/end_user_account_data
```

Returns an array of all end-user's account data that the end-user has shared with the organization.

Response body:
```json
{
  "success": true,
  "allEndUserAccountData": [
    {
      "rhldr_id": "W_puM0eb-iRRvhUcKNUE9wcDCokSD6IHk5-FO0p-GJM",
      "personal_data": {
        "email": "al.pacino@example.com"
      },
      "shared_personal_data": {},
      "consents": {},
      "communication_channels": {}
    },
    {
      "rhldr_id": "fDry8k1Y7J0uZ1Tnkmb6UtPqBDkvRiVOcSqYHfULr-M",
      "personal_data": {
        "email": "alice.walker@example.com",
        "firstname": "alice",
        "lastname": "walker",
        "gender": "unique",
      },
      "shared_personal_data": {},
      "consents": {},
      "communication_channels": {}
    }
  ]
}
```

<a name="get-end-user-account-data"></a>
### Get end-user account data

Requires a rights-holder (i.e. end-user) id.

Endpoint:
```
GET http://sandbox.jlinclabs.net/api/end_user_account_data/:rhldr_id
```

Returns the end-user account data.

Response body:
```json
{
  "success": true,
  "endUserAccountData": {
    "rhldr_id": "W_puM0eb-iRRvhUcKNUE9wcDCokSD6IHk5-FO0p-GJM",
    "consents": {},
    "communication_channels": {},
    "personal_data": {
      "email": "al.pacino@example.com"
    },
    "shared_personal_data": {}
  }
}
```

<a name="get-filtered-end-user-account-data"></a>
### Get filtered end-user account data

Expects the query param `page` that determines the batch to retrieve.

Accepts the `filters` query param. Each value is of the form `[type, value]`. The following is a list of accepted types:
* 'Permission Enabled'
* 'Permission Disabled'
* 'Requested Data Shared'
* 'Requested Data Not Shared'

A `type` with permission in the name requires a valid permission field as its `value`. A `type` with requested data requires a valid personal data field as its `value`.

For example, if we were filtering for users with the `Membership` permission enabled, who were also sharing their `firstname`:
```js
const filters = [['Permission Enabled', 'Membership'], ['Requested Data Shared', 'firstname']]
const filtersQueryParam = encodeURIComponent(JSON.stringify(filters))

// filtersQueryParam: %5B%5B%22Permission%20Enabled%22%2C%22Membership%22%5D%2C%5B%22Requested%20Data%20Shared%22%2C%22firstname%22%5D%5D
```

Endpoint:
```
GET http://sandbox.jlinclabs.net/api/end_users?page=1?filters=%5B%5B%22Permission%20Enabled%22%2C%22Membership%22%5D%2C%5B%22Requested%20Data%20Shared%22%2C%22firstname%22%5D%5D
```

Params:
```
page (number)
filters (url encoded JSON array)
```

Returns an arrays of arrays of which the first value is a list of headers. The subsequent values each represent an end-user's data.

Response body:
```json
{
  "success": true,
  "endUsers": [
    [
      "Rights Holder Id",
      "Sisa Signed At",
      "Email",
      "First Name",
      "Surname",
      "Birthdate",
      "Gender",
      "Street",
      "Country",
      "Home Phone Number (Land Line)",
      "Business Industry",
      "Business Street",
      "Business City",
      "Business Country",
      "Discount Offers Consent Enabled",
      "Newsletter Consent Enabled",
      "Membership Consent Enabled",
      "Volunteering Consent Enabled",
      "Sharing Data within Group Consent Enabled",
      "Automated Decision Making Consent Enabled",
      "Online Tracking Consent Enabled",
      "Fax Communication Channel Enabled",
      "Voice Communication Channel Enabled"
    ],
    [
      "fDry8k1Y7J0uZ1Tnkmb6UtPqBDkvRiVOcSqYHfULr-M",
      "2018-11-07T19:52:06.323Z",
      "alice.walker@example.com",
      "",
      "",
      "",
      "",
      "",
      "",
      "",
      "",
      "",
      "",
      "",
      "Enabled",
      "Enabled",
      "Enabled",
      "Disabled",
      "Disabled",
      "Disabled",
      "Disabled",
      "Disabled",
      "Disabled"
    ],
    [
      "W_puM0eb-iRRvhUcKNUE9wcDCokSD6IHk5-FO0p-GJM",
      "2018-11-07T19:52:06.408Z",
      "Al.Pacino@example.com",
      "",
      "",
      "",
      "",
      "",
      "",
      "",
      "",
      "",
      "",
      "",
      "Disabled",
      "Disabled",
      "Disabled",
      "Disabled",
      "Disabled",
      "Disabled",
      "Disabled",
      "Disabled",
      "Disabled"
    ]
  ]
}
```

<a name="get-csv-of-filtered-end-user-account-data"></a>
### Get CSV of filtered end-user account data

This routes does not authenticate via the `Session-Id` header, but instead through `sessionId` query param. Set `sessionId` to a valid session ID. Also accepts the same `filters` param as outlined in [preceding]() endpoint.

Endpoint:
```
GET http://sandbox.jlinclabs.net/api/end_users.csv
```

Returns a CSV with rows for all users that match the given `filters`.

<a name="get-feed-posts"></a>
### Get feed posts

Takes the optional query param `before`, a date string that will cause the server to only send feed posts that were created before that date.

Endpoint:
```
GET http://sandbox.jlinclabs.net/api/feed/posts?before=2018-11-07T23%3A31%3A16.046Z
```

Params:
```
before (url encoded ISO date string)
```

Returns an array of the organization's feed posts.

Response body:
```json
{
  "success": true,
  "posts": [
    {
      "uid": "ae5492a5c6da7524b948df9df02a2528",
      "organizationApikey": "bobco",
      "body": "Post body",
      "title": "Example post title",
      "mediaUrl": "http://imageurl.com/image.jpg",
      "mediaMimeType": "image/jpeg",
      "createdAt": "2018-11-07T23:32:19.518Z",
      "myPost": true,
      "organizationPost": false
    },
    {
      "uid": "ae5492a5c6da7524b948df9df02a2521",
      "organizationApikey": "bobco",
      "body": "Post body 2",
      "title": "Example post title 2",
      "mediaUrl": "http://imageurl.com/image.jpg",
      "mediaMimeType": "image/jpeg",
      "createdAt": "2018-11-07T23:34:19.518Z",
      "myPost": true,
      "organizationPost": false
    },
  ]
}
```

<a name="create-feed-post"></a>
### Create feed post

Endpoint:
```
POST http://sandbox.jlinclabs.net/api/feed/posts/create
```

Post body:
```json
{
  "post": {
    "title": "This is a post title",
    "body": "This is a post body",
    "mediaUrl": "http://videourl.com/video.mp4",
    "mediaMimeType": "video/mp4"
  }
}
```

Returns the created post.

Response body:
```json
{
  "success": true,
  "post": {
    "uid": "e80f11b39aa3294ed3b5a37b69012a73",
    "organizationApikey": "bobco",
    "body": "This is a post body",
    "createdAt": "2018-11-07T23:35:55.961Z",
    "deletedAt": null,
    "hiddenAt": null,
    "title": "This is a post title",
    "mediaUrl": "http://videourl.com/video.mp4",
    "mediaMimeType": "video/mp4",
    "organizationPost": true
  }
}
```

<a name="delete-feed-post"></a>
### Delete feed post

Soft deletes a feed post.

Endpoint:
```
POST http://sandbox.jlinclabs.net/api/feed/posts/delete
```

Post body:
```json
{
  "post": {
    "feedPostUid": "e80f11b39aa3294ed3b5a37b69012a73"
  }
}
```

If the post has been deleted.

Response body:
```json
{
  "success": true
}
```

<a name="hide-feed-post"></a>
### Hide feed post

Hides a post from the feed.

Endpoint:
```
POST http://sandbox.jlinclabs.net/api/feed/posts/hide
```

Post body:
```json
{
  "post": {
    "feedPostUid": "e80f11b39aa3294ed3b5a37b69012a73"
  }
}
```

If the post has been hidden.

Response body:
```json
{
  "success": true
}
```

<a name="show-feed-post"></a>
### Show feed post

Shows a previously hidden feed post.

Endpoint:
```
POST http://sandbox.jlinclabs.net/api/feed/posts/show
```

Post body:
```json
{
  "post": {
    "feedPostUid": "e80f11b39aa3294ed3b5a37b69012a73"
  }
}
```

If the post has been shown.

Response body:
```json
{
  "success": true
}
```

<a name="get-buying-interests"></a>
### Get buying interests

Gets a list of all the buying interests that match the `industries`, `brands`, and `marketplace tags` in the organization's profile.

Endpoint:
```
GET http://sandbox.jlinclabs.net/api/buying_interests
```

Response body:
```json
{
  "success": true,
  "buyingInterests": [
    {
      "uid": "10504b9f0c13c5b9c1e21dc30a3618c6",
      "email": "alice.walker@example.com",
      "first_name": "alice",
      "last_name": "walker",
      "tags": [
        "4wd"
      ],
      "brands": [
        "Ford"
      ],
      "currency": "$",
      "end_date": "2019-10-08",
      "industry": "Automotive",
      "location": "San Francisco",
      "price_low": 1000,
      "price_high": 100000,
      "description": "Buying interest description",
      "beginning_date": "2018-11-07"
    }
  ]
}
```

<a name="get-verified-emails"></a>
### Get verified emails

Endpoint:
```
GET http://sandbox.jlinclabs.net/api/orgs/emails
```

Returns a list of verified and pending verification emails.

Response body:
```json
{
  "success": true,
  "emails": [
    {
      "email": "alice@example.com",
      "verified": true
    },
    {
      "email": "alice2@example.com",
      "verified": false
    }
  ]
}
```

<a name="add-verified-email"></a>
### Add verified email

Sends an email to the given email with a verification link. Email must be unique.

Endpoint:
```
POST http://sandbox.jlinclabs.net/api/orgs/emails/add
```

Post body:
```json
{
  "email": "newemail@example.com"
}
```

If the email has been successfully added and is pending verification.

Response body:
```json
{
  "success": true,
  "email": "newemail@example.com",
  "verified": false
}
```

Example verification link: `http://sandbox.b-portal.jlinclabs.net/verify_email/f91f371c9e2ec35b94fe9abf8213bc0d`

The last paramater of the verification link is the `verificationCode` which can be used to verify the email.

Endpoint:
```
POST http://sandbox.jlinclabs.net/api/orgs/emails/verify
```

When the email has been successfully verified.

Post body:
```json
{
  "success": true,
  "email": "newemail@example.com",
  "verified": true
}
```

<a name="get-email-list-csv-for-persmission"></a>
### Get email mailing list CSV for permission

This routes does not authenticate via the `Session-Id` header, but instead through `sessionId` query param. Set `sessionId` to a valid session ID. Also requires a `consentType` query param with a valid permission as its value.

Endpoint:
```
GET http://sandbox.jlinclabs.net/api/email_list/:sessionId/:consentType
```

Returns a CSV with rows for all users who have SISAs with this organization and who have the given `consentType` enabled.
