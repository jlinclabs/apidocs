---
layout: default
---
## The B server

### Server status
_GET https://sandbox.b.jlinclabs.net/status_

If the server is responding returns:

```json
{
  "status": "OK"
}
```

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

### Creating a new org

_POST https://sandbox.b.jlinclabs.net/api/orgs/create_

Creates a new org, a new org admin, and registers the org with the A server.

```json
{
  "email": "neworg@example.com",
  "name": "New Org, LLC",
  "password": "foobar",
  "password_confirmation": "foobar",
  "apikey": "not required unless name is blank",
  "terms_urls": "https://terms.jlinc.org",
  "stripe_token": "tok_1CR5JRFNXsZIJI8lCl1l04S4",
  "salesforce": "boolean whether org has a Salesforce connection"
}

```

Returns a session_id for the new org admin, and the org parameters.

```json
{
 "success": true,
 "sessionID": "5e1668d7fffaadabf0474088e...",
 "organization": "organization details..."
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

### Adding an org admin to an existing org

_POST https://sandbox.b.jlinclabs.net/api/orgs/register_

Requires a signup token created out-of-band

```json
{
  "email": "newadmin@example.com",
  "password": "foobar",
  "password_confirmation": "foobar",
  "signuptoken": "5e1668d7fffaadabf0474088e..."
}
```

Success returns the email and a session ID:

```json
{
 "success": true,
 "email": "newadmin@example.com",
 "sessionID": "5e1668d7fffaadabf0474088e..."
}
```

### Logging in an org admin

_POST https://sandbox.b.jlinclabs.net/api/orgs/login_

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

_POST https://sandbox.b.jlinclabs.net/api/orgs/authn_

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

### Getting org profile data

_GET https://sandbox.b.jlinclabs.net/api/orgs/profile/:sessionID_

If the session ID is valid the organization profile data will be returned:
```json
  "success": true,
  "organization_profile": {
    "apikey": "bobco",
    "logo": "data:image/gif;base64,R0lGODlhPQBEAPeoAJosM//AwO/…",
    "icon": "data:image/gif;base64,R0lGODlhPQBEAPeoAJosM//AwO/…",
    "name": "Bob & Co",
    "dpo_email": "bobco@earthlingsunited.org",
    "domain": "endpotatoefamine.com",
    "contact_phone": "55555555555",
    "contact_phone2": "5555555555",
    "address": "83rd city ave",
    "city": "Megatropolis",
    "post_code": "84594",
    "country": "The One True Country"
  }
```

### Updating org profile data

_POST https://sandbox.b.jlinclabs.net/api/orgs/profile_

```json
  "session_id": "i4uthu749489844f…",
  "organization_profile": {
    "apikey": "bobco",
    "logo": "data:image/gif;base64,R0lGODlhPQBEAPeoAJosM//AwO/…",
    "icon": "data:image/gif;base64,R0lGODlhPQBEAPeoAJosM//AwO/…",
    "name": "Bob & Co",
    "dpo_email": "bobco@earthlingsunited.org",
    "domain": "endpotatoefamine.com",
    "contact_phone": "55555555555",
    "contact_phone2": "5555555555",
    "address": "81st city ave",
    "city": "Megatropolis",
    "post_code": "84592",
    "country": "The One True Earth"
  }
```

If the session ID is valid, updates the organization profile, then returns true and the updated profile data.

```json
{
  "success": true,
  "organization_profile": {
    "apikey": "bobco",
    "logo": "data:image/gif;base64,R0lGODlhPQBEAPeoAJosM//AwO/…",
    "icon": "data:image/gif;base64,R0lGODlhPQBEAPeoAJosM//AwO/…",
    "name": "Bob & Co",
    "dpo_email": "bobco@earthlingsunited.org",
    "domain": "endpotatoefamine.com",
    "contact_phone": "55555555555",
    "contact_phone2": "5555555555",
    "address": "81st city ave",
    "city": "Megatropolis",
    "post_code": "84592",
    "country": "The One True Earth"
  }
}
```

### Logging out an org admin

_POST https://sandbox.b.jlinclabs.net/api/orgs/logout_

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


### Getting a consent report

_GET https://sandbox.b.jlinclabs.net/api/orgs/consent_report/{session_id}_

Requires a valid org admin session id.

Returns a consent report (a sorted list of transaction events) for the current point in time.

```json
{
 "success": true,
 "consentReport": "consent report details..."
}
```

### Creating an end-user invitation URL

_POST https://sandbox.b.jlinclabs.net/api/orgs/invite_end_user_

Requires a valid org admin session id and an email address.

```json
{
 "session_id": "5e1668d7fffaadabf0474088e...",
 "email": "enduser@example.com"
}
```

Returns a signup URL for transmission to the invitee.

```json
{
 "success": true,
 "signup": "https://a.jlinclabs.net/regauth/gbobcog690bcd707..."
}
```

### Creating multiple end-user invitation URLs from a CSV file

_POST https://sandbox.b.jlinclabs.net/api/orgs/invite_end_user_batch_

Requires submitting a multipart/form-data form with a text (should be hidden) `session_id` field containing
a valid org admin session id, and field of type "file" named `emailListCSVStream` containing the CSV file.

Returns an `invites.csv` file with the signup URLs.

### Get all the end-user account data for an org

_GET https://sandbox.b.jlinclabs.net/api/end_user_account_data/{session_id}

Requires a valid org admin session id.

Returns an array of end-user account data.

```json
{
 "success": true,
 "allEndUserAccountData": ["..."]
}
```

### Get one end-user's account data

_GET https://sandbox.b.jlinclabs.net/api/end_user_account_data/{rhldr_id}/{session_id}

Requires a rights-holder (i.e. end-user) id and a valid org admin session id.

Returns the end-user account data.

```json
{
 "success": true,
 "endUserAccountData": "..."
}
```
