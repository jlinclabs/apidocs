---
layout: default
---
## The B server

### Onboarding a new user
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

### Registering a new org admin
An org is an organization or company or company division -- any entity that has control of one or more apikeys/apisecrets.

To create an login credential for a person or role that will administer these apikeys requires a signup token from the manager:

_POST https://sandbox.b.jlinclabs.net/api/orgs/register_

```json
{
 "username": "OrgAdmin",
 "password": "foobar",
 "pwdconfirm": "foobar",
 "signuptoken": "48ea8317920bb268ebc7739c8ecee4c5"
}
```

Usernames are converted to lowercase for both registration and login.  
On success the lowercased username will be returned:

```json
{
 "success": true,
 "username": "orgadmin"
}
```

### Checking whether an org admin username is already in use

_GET https://sandbox.b.jlinclabs.net/api/orgs/exists/{name-to-check}_

If the name (after lowercasing) is available (i.e. does not exist), the reply result will be `false`. If the name does exist the reply will be `true`.

```json
{
 "success": true,
 "result": false
}
```

### Logging in an org admin

_POST https://sandbox.b.jlinclabs.net/api/orgs/login

```json
{
 "username": "OrgAdmin",
 "password": "foobar"
}
```

Usernames are lowercased before checking. A successful login returns a session ID:

```json
{
 "success": true,
 "sessionID": "5e1668d7fffaadabf0474088e..."
}
```

### Verifying a session ID

_POST https://sandbox.b.jlinclabs.net/api/orgs/authn

```json
{
 "session_id": "5e1668d7fffaadabf0474088e..."
}
```

If the session is valid, updates the session time-to-live and returns true. Otherwise returns false with a 401 status.

```json
{
 "success": true
}
```

### Logging out an org admin

_POST https://sandbox.b.jlinclabs.net/api/orgs/logout

```json
{
 "session_id": "5e1668d7fffaadabf0474088e..."
}
```

If the session exists and has been destroyed returns true.

```json
{
 "success": true
}
```
