---
layout: default
---
## The A server

### [ ](#signup_urls)Using signup URLs
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

{% include starsep.html %}

### [ ](#registration)Registering a new user
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

### [ ](#loggingin)Re-logging in

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

### [ ](#loggingout)Logging out
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

### [ ](#vendorlist)Obtaining the vendor list
In order to retrieve vendor specific information for the user, you must provide both a login token and a vendor public key.

To get that vendor key, use this API. It returns a list of vendor choices for this user.  
The format is /api/vendors/{login_token}

_GET https://staging.a.jlinclabs.net/api/vendors/de7a126fef783bd6..._

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
