---
layout: default
---
## Introduction
The JLINC protocol takes place primarily between two servers.

One server represents the business, enterprise or organization's interests, and is named the "B" server.

The other server represents the end-user's interests, and is named the "A" server.

For the sake of intelligibility, in these docs we refer to the organization user as "BobCo" and the individual end-user as "Alice".

These docs describe the APIs for front-end applications to access the A server for Alice and the B server for BobCo.

{% include starsep.html %}

### API conventions
All APIs that receive body content expect that content in JSON form, and all calls return a JSON formatted response.

The philosophy behind these APIs is that an unsuccessful call should still return meaningful results (and an HTTP `200` code) to the calling application, so that the end user can be shown a meaningful response. To that end, most calls return a JSON object the first key of which is `success` containing a boolean value. If `success` is false, the caller should look for an `error` key which will contain an explanation of what went wrong.

```json
{
 "success": false,
 "error": "Bad Password"
}
```
If `success` is true, then the other data is context specific.

```json
{
 "success": true,
 "token": "b214f47a663108ea7b7bd952e3b655d1"
}
```
Although this does not conform to REST, the REST conventions are still respected in two ways. First, calls that return information without changing server state use `GET` and all others use `POST`.

Secondly, in the event of an authentication failure we return an HTTP `401` error so that the caller can redirect the user to a login page (or other authentication process) without further ado. And if the caller requests a non-existent endpoint we return an HTTP `404` error, and internal server errors return `500` responses.

{% include starsep.html %}

### Onboarding new users

A new user can be onboarded in one of two ways:
* either BobCo supplies customer information on Alice to the B server, which generates an invitation URL which BobCo transmits to Alice, enabling her to sign up,
* or Alice creates or accesses her account on her app, and her app displays a list of vendors to whom she can offer information via the A server.

The B server method can be reviewed <a href="{{ '/b_server#onboarding' | relative_url }}">here</a>.

The A server method is still under development.
