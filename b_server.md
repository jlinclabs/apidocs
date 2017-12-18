---
layout: default
---
## The B server

### [ ](#onboarding)Onboarding a new user
This is an API to create a signup link that BobCo can send to Alice, inviting her to take charge of the contact information he has about her, and set her preferences about what he can do with it.

_POST https://sandbox.b.jlinclabs.net/api/userdata/new_user_

```json
{
 "apikey": "bobco",
 "apisecret": "670b5bba405d62961751a6b97dbfaf5c",
 "data": {
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
The `apikey` and `apisecret` are required for authentication.

Under the `data` key, the `email` is required, and must be unique in BobCo's account. All other data fields are optional. Any other fields that might be included will be ignored. From time to time new optional data fields may be added.

On success a signup link will be returned:

```json
{
 "success": true,
 "signup": "https://alpha.userprefs.com/regauth/65c29f1ff4975dff"
}
```

This link should be sent to Alice to register her account with BobCo.
