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
