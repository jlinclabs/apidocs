---
layout: default
---
## The A server

### [ ](#signup_urls)Using signup URLs
When the front-end gets an invite url such as for example:  
_https://alpha.userprefs.com/regauth/65c29f1ff4975dff_  
the regauth page should put up a "please wait - this may take a few moments" type notice and then

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
