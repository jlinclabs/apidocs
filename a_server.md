---
layout: default
---
## The A server

### Using signup URLs
When the front-end regauth page is accessed by a signup url, for example:  
_https://alpha.userprefs.com/regauth/65c29f1ff4975dff_  
the regauth page should put up a "please wait - this may take a few moments" type notice and then extract the token from the url to call the A server

_POST https://sandbox.a.jlinclabs.net/api/register/new_

```json
{
 "token": "65c29f1ff4975dff"
}
```

The A server will verify the token with the B server, and then return a `signup` token.

```json
{
 "success": true,
 "signuptoken": "ec281d5348c24ed0d8637297ed9faad4"
}
```

Store the `signuptoken` and redirect the user to the registration form.

If the user submits a token that has already been used, the `signuptoken` field will have `existing` instead of a hex value. In that case redirect the user to the login page.

```json
{
 "success": true,
 "signuptoken": "existing"
}
```

#### Signing up an existing user to a new vendor

Include the user's logintoken along with the registration token:  
_POST https://sandbox.a.jlinclabs.net/api/register/new_

```json
{
 "token": "65c29f1ff4975dff",
 "logintoken": "ecdf4593d3e134574794a061be"
}
```

On success the B server will return a `vendorPk` public key, which can be used to redirect the user to the new vendor's page..

```json
{
 "success": true,
 "vendorPk": "GbxQVQrjoYbiQ-ALGJNmpblsLSKt3ROMmpa7Zn0Rv4g"
}
```

{% include starsep.html %}

### Registering a new user
When a prospective new user arrives at the app's registration page with a signuptoken, as above, they should be presented with a form asking for an email address to use as a username, and passphrase and passphrase confirmation fields.

The app then submits the form data and the signuptoken to the A server.

_POST https://sandbox.a.jlinclabs.net/api/authn/register_
```json
{
 "username": "alice@example.com",
 "passphrase": "foobar",
 "passphrase2": "foobar",
 "signuptoken": "ec281d5348c24ed0d8637297ed9faad4"
}
```

On success it returns data (explained below);
```json
{
 "success": true,
 "user": {
  "account_id": "DA9yqCMEff5SV7fJvP51Vj44eZGts5nwkHrPp4D_8bQ",
  "username": "alice@example.com"
 },
 "nicepwd": "loader woodlot synthesizing sharp dative sativa",
 "logintoken": "6f81124583a583034df88a9664891a8b",
 "vendorname": "bobco",
 "vendor_pk": "Xp0A5Jj9wb1QpVSU0u8hhbks3FWcrgU1I3RZmghfNkU"
}
```
The `user:account_id` is a public key that has been assigned to this user.  
The `nicepwd` is a recovery key - you want to display that to them and encourage them to print it out and save it in a secure place.

Store the `logintoken`, `vendorname` and `vendor_pk` (the vendor's public key).  
The user is now logged in and can move to the logged-in user menu items.

{% include starsep.html %}

### Re-logging in

_POST https://sandbox.a.jlinclabs.net/api/authn/login_
```json
{
 "username": "alice@example.com",
 "passphrase": "foobar"
}
```
On success it returns
```json
{
 "success": true,
 "token": "de7a126fef783bd6d9ac1da48cada93ee"
}
```

{% include starsep.html %}

### Logging out
_POST https://sandbox.a.jlinclabs.net/api/authn/logout_
```json
{
 "token": "de7a126fef783bd6d9ac1da48cada93ee"
}
```
On success it returns just:
```json
{
 "success": true
}
```

{% include starsep.html %}

### Obtaining the vendor list
In order to retrieve vendor specific information for the user, you must provide both a login token and a vendor public key.

To get that vendor key, use this API. It returns a list of vendor choices for this user.  
The format is /api/vendors/{login_token}

_GET https://sandbox.a.jlinclabs.net/api/vendors/de7a126fef783bd6..._

Which returns:

```json
{
 "success": true,
 "vendors": [
   {
    "vendor_pk": "Xp0A5Jj9wb1QpVSU0u8hhbks3FWcrgU1I3RZmghfNkU",
    "vendor_name": "bobco"
   },
   {
    "vendor_pk": "V30PBeHuRk90M6K5r7rdLZ2ifMte6hnw2eA0X9uD_78",
    "vendor_name": "charlieco"
   }
  ]
}
```

{% include starsep.html %}

### Retrieving user account data
Send the login token and the vendor public key to /api/remotedata/accountdata/{login_token}/{vendor_pk}

_GET https://sandbox.a.jlinclabs.net/api/remotedata/accountdata/2dfb4668.../XxnnSLWq..._

Returns:

```json
{
 "success": true,
 "accountData": {
   "rhldr_id": "DrMVuLh_hUbvrUZt2SZqMFgw1DN9ZQl6OvzgNz9Fo98",
   "email": "alice@example.com",
   "firstname": "Alice",
   "lastname": "McPerson",
   "mailingstreet": "123 Main Street",
   "mailingcity": "Oakland",
   "mailingstate": "CA",
   "mailingpostalcode": "01234",
   "mailingcountry": "US",
   "homephone": "555-111-4444",
   "mobilephone": "555-111-2222",
   "email_share": true,
   "firstname_share": true,
   "lastname_share": true,
   "mailingstreet_share": true,
   "mailingcity_share": true,
   "mailingstate_share": true,
   "mailingpostalcode_share": true,
   "mailingcountry_share": true,
   "homephone_share": true,
   "mobilephone_share": true
 }
}
```

The fields ending in "_share" indicate whether the corresponding data field should be shared with the B server, or just retained on the A server.

On first access after a new registration, the data held by the B Server on the new account is returned, along with the "_share" fields defaulted to true. After that the data stored on the local A Server is returned.

{% include starsep.html %}

### Editing user account data

_POST https://sandbox.a.jlinclabs.net/api/remotedata/updatecontactdata_
```json
{
 "token": "c51f3eaf6fb2664916401b7be0c8c94c59aa427a30690530a000adfd944a33c2",
 "vendorpk": "mj4PuJVDjvYBmDjKKBGSXf_LOSv3BBG4Jr4Ij3iOtWo",
 "updateAll": true,
 "data": {
  "mailingpostalcode": "00001",
  "firstname": "Test"
 }
}
```

When `updateAll` is set to `true`, all the vendors that the user identified by the login token is registered with are updated. If the `updateAll` key is omitted or set to anything other than `true`, then the update is only applied to the vendor whose public key is supplied in `vendorpk`. The `vendorpk` value whose page the request is originating from must be supplied in either case.

The currently available fields are:

```
firstname
lastname
mailingstreet
mailingcity
mailingpostalcode
mailingcountry
phone
homephone
mobilephone
email
birthdate
gender
```

The birthdate value should be in the format `yyyy-mm-dd`. All other fields are plain text format.

On success the number of vendors updated is returned.

```json
{
 "success": true,
 "updated": 3
}
```

A JLINC receipt from each B server for the update will become available asynchronously.


{% include starsep.html %}

### Editing user account items to share
Each of the fields in a user's account with a vendor can be marked as to whether to share changes in that field with the vendor or not.

These show up when accessing the account data as the fields ending in `_share` which correspond to the field in the basename. For example, `mailingstreet_share` indicates whether to share `mailingstreet` changes with the vendor.

The API to edit these fields is:  
_POST https://sandbox.a.jlinclabs.net/api/remotedata/updatecontactshare_
```json
{
 "token": "c51f3eaf6fb2664916401b7be0c8c94c59aa427a30690530a000adfd944a33c2",
 "vendorpk": "mj4PuJVDjvYBmDjKKBGSXf_LOSv3BBG4Jr4Ij3iOtWo",
 "updateAll": true,
 "sharedata": {
  "mailingstreet": false,
  "mailingcity": false,
  "mailingpostalcode": false,
  "mailingcountry": true
 }
}
```

The `updateAll` key if set to `true` updates the changes with all the user's vendors.

On success the number of vendors updated is returned.

```json
{
 "success": true,
 "updated": 3
}
```  

{% include starsep.html %}

### Retrieving user preferences

Send the login token and the vendor public key to /api/remotedata/prefsdata/{login_token}/{vendor_pk}

_GET https://sandbox.a.jlinclabs.net/api/remotedata/prefsdata/2dfb4668.../XxnnSLWq..._

Returns:

```json
{
 "success": true,
 "prefsData": {
  "emailmarketing": false,
  "emailcomms": true
  }
}
```

Currently the `emailmarketing` and `emailcomms` fields are the only preference settings available.


{% include starsep.html %}

### Editing user preferences

The API to edit preference fields is:  
_POST https://sandbox.a.jlinclabs.net/api/remotedata/updateprefsdata_
```json
{
 "token": "c51f3eaf6fb2664916401b7be0c8c94c59aa427a30690530a000adfd944a33c2",
 "vendorpk": "mj4PuJVDjvYBmDjKKBGSXf_LOSv3BBG4Jr4Ij3iOtWo",
 "updateAll": true,
 "data": {
  "emailmarketing": true,
  "emailcomms": true
 }
}
```

The `updateAll` key if set to `true` updates the changes with all the user's vendors.

On success the number of vendors updated is returned.

```json
{
 "success": true,
 "updated": 3
}
```  

A JLINC receipt from each B server for the update will become available asynchronously.

{% include starsep.html %}

### Retrieving receipts

Send the login token and the vendor public key to /api/remotedata/rcptsdata/{login_token}/{vendor_pk}

_GET https://sandbox.a.jlinclabs.net/api/remotedata/rcptsdata/2dfb4668.../XxnnSLWq..._

Returns an array of the user's receipts:

```json
{
 "success": true,
  "receipts": [
  {
   "receipt": {
    "@context": "https://jlinclabs.org/v05/context/isa-rcpt.jsonld",
    "Version": "0.5",
    "DataTS": 1516483209480,
    "ISAHash": "KoCe5IQPtjtnXTyTNWTy8AeQleaw2Hxr8O6W02HjZ3k",
    "DataHash": "g8HVIhhavQsgVuDjoA-hnrd3vKr0bJ2AokMus_tWvfc",
    "RhldrPkID": "hRgwFkb9-7ed9xtX8009MltbpjaX7uIlJUYrMDRQUco",
    "RhldrSig": "I2lslUas26yV23r4WVj5hzcfELlHOya-atVHYE0BQ2dLPFoUOv_17dseoFLi4tQRQflhXYo_ojNfdkyp-NJYD8TYy5zsr1CdN_8dqultqtnhCvdeaC2IAXKaNH-d2uoT",
    "DcustPkAlg": "sha256:ed25519",
    "DcustPkID": "GbxQVQrjoYbiQ-ALGJNmpblsLSKt3ROMmpa7Zn0Rv4g",
    "DcustSig": "lNb6lrorwrMhSfh_0FRtNuRQrvSxbWYlm1ZZ287ZMfJ4RMWcSOBaxmHtnujO_rSn0itXJ2STwLaCHK1TQGcpDcTYy5zsr1CdN_8dqultqtnhCvdeaC2IAXKaNH-d2uoT"
   },
   "subject_data": {
    "homephone": "555-111-3334",
    "email": "user5@example.com"
   },
   "created": "2018-01-20T21:20:09.521Z"
  },
  {
   "receipt": {
    "@context": "https://jlinclabs.org/v05/context/isa-rcpt.jsonld",
    "Version": "0.5",
    "ISAHash": "KoCe5IQPtjtnXTyTNWTy8AeQleaw2Hxr8O6W02HjZ3k",
    "DataHash": "ONisgH5_rATj-JtbEFm4l8m5KNYk6bJSy4WR1ktB608",
    "DataTS": 1515997768035,
    "DcustPkAlg": "sha256:ed25519",
    "DcustPkID": "GbxQVQrjoYbiQ-ALGJNmpblsLSKt3ROMmpa7Zn0Rv4g",
    "RhldrPkID": "hRgwFkb9-7ed9xtX8009MltbpjaX7uIlJUYrMDRQUco",
    "DcustSig": "hRsQWOYhPlMSDn3UTBbeVN8HD8g30zSiBgekirq4Im8UG4vTqmC7AcwPik4XJxGeSWf3ap9br9a_onPkx3ZTDJoJQ0aDRO33LMfZkQtlVa0DdMaaoy99GXSMAvFJ5xdm"
   },
   "subject_data": {
    "firstname": "Test",
    "lastname": "User5",
    "name": "Test User5",
    "mailingstreet": "123 Main Street",
    "mailingcity": "Oakland",
    "mailingstate": "CA",
    "mailingpostalcode": "12345",
    "mailingcountry": "US",
    "homephone": "555-111-4444",
    "mobilephone": "555-111-2222",
    "email": "user5@example.com"
   },
   "created": "2018-01-15T06:29:28.050Z"
  }
 ]
}
```

Appending a query string with variables `page` and/or `num` controls the pagination.  Order doesn't matter, and either or both may be omitted.

If omitted, `page` defaults to 1 and `num` (the number of receipts to show per page) defaults to 10.

Receipts are always sorted by creation date/time, with the newest at the top.

Example:  
_GET https://.../api/remotedata/rcptsdata/.../...?page=2&num=20_
