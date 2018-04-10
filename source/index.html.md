---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  # - shell
  # - ruby
  # - python
  - javascript

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the Mixbook User Service API! You can use our API to all user related endpoints accross all franchises from the Mixbook group.

We have language bindings in Shell, Ruby, and Python! You can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right.

# Authentication

Mixbook user service API implements unified authentication along all the franchise specific platforms. 

Every service must have a unique API token that identifies the requesting service (e.g. admin, frouser, etc.), the requesting environment (e.g. production, staging, etc.), expires within 1 year, although more frequent expiration is acceptable (we want to force regular token rotation) and is unique from every other token.

Services must provide login information for a user. If authenticating the user for the first time (in a new browser session, for example) username and password should be provided. In all other cases, the session token should be provided. The session token could be whatever unique token appropriate for the service (mosaic application token, session token, visitor token, etc.).

If failed, a generic authentication failure is returned in all cases consisting on just a 401 not authorized error code. For security reasons, no descriptive information is returned to the calling service, but descriptive info should be logged in the user service.

If successful, the user service should return a signed JWT payload that expires in 30 minutes. The data should be relatively static and the ability to pass the data around could significantly reduce user service load and increase performance It can be passed around to other services which can use it to authenticate a user as long as the signing is still valid (i.e. correctly signed, not expired, etc.). If valid, other services do not need to re-authenticate or request the user info again. It includes all base access roles for the user (e.g. admin, printerops, etc.) and commonly used user information in the token.

## Authenticate a session

> To authorize, use something like this:

```javascript
data = { "email": "<user_email>", "password": "<user_password>" }

$.post("/authentication", function(data, status){
  let jwt_token = data["token"]
});
```

> The above command returns a JWT formatted JSON, structured like this:

> Header:

```json
{
  "alg": "H256",
  "type": "JWT",
  "exp": 1300819380
}
```

> Payload:

```json
{
  "user_id": 12,
  "signup_date": "2012-04-23T18:25:43.511Z",
  "notification_status": "subscribed",
  "first_name": "John",
  "roles": ["admin"]
}
```

> Make sure to replace `<user_email>` with your user registered email and `<user_password>` with your user registered password.


This endpoint authenticates the current session returning the token that should be included in all user actions

### HTTP Request

`POST /authentication`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
email | null | Email of the user we want to authenticate
password | null | Password of the user we want to authenticate

Once authenticated, Mixbook user service expects for the API token to be incldued in all API requests except login, registration and password recovering processes. The token shold be included as a header looking like the following:

`Authorization: JWT <jwt_token>`

<aside class="notice">
You must replace <code>jwt_token</code> with your personal obtained API token.
</aside>

# Users

## Create new user

```javascript
data = {
  email: "<user_email>", 
  password: "<user_password>", 
  first_name: "<user_first_name>", 
  last_name: "<user_last_name>",
  shipping_address: "<shipping_address>",
  billing_address: "<billing_address>",
  general_address: "<general_address>",
  notes: "<admin_notes>",
  feature_flags: ["<feature_flag1>", "<feature_flag2>"...],
  sessions: ["<session1>", "<session2>"...]
}

$.post("/users", function(data, status){
});
```

> The above command returns 201 status code (created) in case of successful creation, error code in any other case.

For sharing users accross all the franchise especific services, we need to create a shared user and 
link all the related services to him. This endpoint creates a new shared user.

### HTTP Request

`POST /users`

### Query Parameters

Parameter | Default | Description | Required
--------- | ------- | ----------- | --------
email | "" | Email of the user we want to create | True
password | "" | Password of the user we want to create | True 
first_name | "" | First name of the user we want to create | True
last_name | "" | Last name of the user we want to create | True
shipping_address | "" | Shipping address for the user we want to create | False
billing_address | "" | Billing address for the user we want to create | False
general_address | "" | General address for the user we want to create | False
notes | [] | Notes included by the admins about the user we want to create. Only available for admins | False
feature_flags | [] | Identify which buckets the user is in for each currently enabled feature. | False
sessions | [] | List of sessions of the user we want to create | False

## Find existing user

```javascript
$.get(
  "/users?pattern=<pattern>&sort=<sort>&asc=<asc>&limit=<limit>", 
  function(data, status){
  }
);
```

> The above command returns 200 status code in case of successful lookup, returning all the users related to the 
specified pattern. In case a non administrator of the platform is calling this endpoint, it will respond with a 401 (Unauthorized)

This endpoint finds all the users matching the specified regular expression pattern

### HTTP Request

`GET /users?pattern=<pattern>&sort=<sort>&asc=<asc>&limit=<limit>`

### Query Parameters

Parameter | Default | Description | Required
--------- | ------- | ----------- | --------
pattern | "" | Pattern interpreted as a regular expression used for finding related users | False
sort | id | User field used to sort the results | False
asc | False | Indicates if the sorted results should be returned in ascending order | False
limit | 20 | Indicates how many users should be returned by this service | False


This action is considered an administrative one so it should be only called by an
administrator of the platform, failing otherwise

## Update existing user

```javascript
data = {
  email: "<user_email>", 
  first_name: "<user_first_name>", 
  last_name: "<user_last_name>",
  shipping_address: "<shipping_address>",
  billing_address: "<billing_address>",
  general_address: "<general_address>",
  notes: "<admin_notes>",
  feature_flags: ["<feature_flag1>", "<feature_flag2>"...],
  sessions: ["<session1>", "<session2>"...]
}

$.put("/users/:user_id", function(data, status){
});
```

> The above command returns 200 status code in case of successful update, error code in any other case.

This endpoint updates the information about the specificed user accross all the linked accounts

### HTTP Request

`PUT /users/:user_id`

### Query Parameters

Parameter | Default | Description | Required
--------- | ------- | ----------- | --------
user_id | null | Id of the user whose information we want to link | True
email | "" | Email of the user we want to create | False
first_name | "" | First name of the user we want to create | False
last_name | "" | Last name of the user we want to create | False
shipping_address | "" | Shipping address for the user we want to create | False
billing_address | "" | Billing address for the user we want to create | False
general_address | "" | General address for the user we want to create | False
notes | [] | Notes included by the admins about the user we want to create. Only available for admins. | False
feature_flags | [] | Identify which buckets the user is in for each currently enabled feature. | False
sessions | [] | List of sessions of the user we want to create | False

## Update password for existing user

```javascript
data = {
  old_password: "<old_password>",
  new_password: "<new_password>", 
}

$.put("/users/:user_id/password", function(data, status){
});
```

> The above command returns 200 status code in case of successful password update, error code in any other case.

This endpoint updates the password of the specificed user accross all the linked accounts

### HTTP Request

`PUT /users/:user_id/password`

### Query Parameters

Parameter | Default | Description | Required
--------- | ------- | ----------- | --------
user_id | null | Id of the user whose information we want to link | True
old_password | "" | Old password of the user we want to update | True
new_password | "" | New password for the user we want to update | True



## Remove existing user

```javascript
$.delete("/users/:user_id", function(data, status){
});
```

> The above command returns 200 status code in case of successful delete, error code in any other case.

This endpoint deletes the specified shared user.

### HTTP Request

`DELETE /users/:user_id`

### Query Parameters

Parameter | Default | Description | Required
--------- | ------- | ----------- | --------
user_id | null | Id of the user whose information we want to link | True

<aside class="warning">
Calling this method, the shared user is deleted. However, it doesnt't remove the service specific users.
</aside>

## Password reset for existing user

```javascript
$.get("/users/:user_email/password_reset", function(data, status){
});
```

> The above command returns 200 status code in case of successful password reset, error code in any other case.

This endpoint starts the password reset process for the specificed user. If the call is accepted and considered 
successful, an email is sent to the user with a random generated code to allow reseting password

### HTTP Request

`GET /users/:user_email/password_reset`

### Query Parameters

Parameter | Default | Description | Required
--------- | ------- | ----------- | --------
user_email | "" | Email of the user whose information we want to reset the password | True

<aside class="info">No authentication token is required for this action</aside>

## Force password reset for existing user

```javascript
$.get("/users/:user_id/force_password_reset", function(data, status){
});
```

> The above command returns 200 status code in case of successful forced password reset, error code in any other case. In case a non administrator of the platform is calling this endpoint, it will respond with a 401 (Unauthorized)

This endpoint starts the password reset process for the specificed user. An email is sent to the user with a random generated code to allow reseting password

### HTTP Request

`GET /users/:user_id/force_password_reset`

### Query Parameters

Parameter | Default | Description | Required
--------- | ------- | ----------- | --------
user_id | "" | Id of the user whose information we want to reset the password | True

<aside class="warning">This action is considered an administrative one so it should be only called by an
administrator of the platform, failing otherwise</aside>

## Confirm password reset for existing user

```javascript
data={
  code: "<reset_code>",
  password: "<new_password>"
}

$.post("/users/:user_email/password_reset", function(data, status){
});
```

> The above command returns 200 status code in case of successful password reset confirmation, error code in any other case.

This endpoint finishes the reset password process for the specificed user. If the attached reset code is valid and accepted,
the included password will be the new password for the specified user.

<aside class="info">No authentication token is required for this action</aside>


### HTTP Request

`POST /users/:user_email/password_reset`

### Query Parameters

Parameter | Default | Description | Required
--------- | ------- | ----------- | --------
user_email | "" | Email of the user whose information we want to link | True
reset_code | "" | Reset code obtained in the received email sent in the previous step | True
password | "" | New password for the specified user | True


# Accounts

Accounts represent an account entity related with a specific user in any of the franchise specific services. 
Consequently, an user will have one or more accounts related with himself. 


## Get all accounts

```javascript
$.get("/users/:user_id/accounts", function(data, status){
});
```

> The above command returns JSON structured like this:

```json
[
  {
    "account_id": "<account_id>",
    "service_id": "<service_id>"
  },
  {
    "account_id": "<account_id>",
    "service_id": "<service_id>"
  }
]
```

This endpoint retrieves all accounts for a specified user.

### HTTP Request

`GET /users/:user_id/accounts`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
user_id | null | Id of the user whose accounts we want to list



## Link accounts

```javascript
data = {accounts: [{id: 1234, service: 'wedpics'}, {id: 3241, service: 'mixbook'}] }

$.post("/users/:user_id/accounts/link", function(data, status){
});
```

> The above command returns JSON structured like this:


This endpoint links the referenced accounts for the specified user.

### HTTP Request

`POST /users/:user_id/accounts/link`

### Query Parameters

Parameter | Default | Description | Required
--------- | ------- | ----------- | --------
user_id | null | Id of the user whose accounts we want to link | True
accounts | [] | Accounts we are interested in link to the provided user | False


## Unlink accounts

```javascript
data = {accounts: [{id: 1234, service: 'wedpics'}, {id: 3241, service: 'mixbook'}]}

$.post("/users/:user_id/accounts/unlink", function(data, status){
});
```

This endpoint unlinks the referenced accounts for the specified user.

### HTTP Request

`POST /users/:user_id/accounts/unlink`

### Query Parameters

Parameter | Default | Description | Required
--------- | ------- | ----------- | --------
user_id | null | Id of the user whose accounts we want to lock | True
accounts | [] | Accounts we are interested in unlink to the provided user | False

## Lock account

```javascript
$.post("/users/:user_id/accounts/:account_id/lock", function(data, status){
});
```

> The above command returns 200 status code in case of successful lock, error code in any other case. In case a
non administrator of the platform is calling this endpoint, it will respond with a 401 (Unauthorized)

This endpoint locks the referenced account for the specified user.

### HTTP Request

`POST /users/:user_id/accounts/:account_id/lock`

### Query Parameters

Parameter | Default | Description | Required
--------- | ------- | ----------- | --------
user_id | null | Id of the user whose accounts we want to link | True
account_id | null | Account and user we want to lock | True

<aside class="warning">This action is considered an administrative one so it should be only called by an
administrator of the platform, failing otherwise</aside>

## Unlock account

```javascript
$.post("/users/:user_id/accounts/:account_id/unlock", function(data, status){
});
```

> The above command returns 200 status code in case of successful unlock, error code in any other case. In case a
non administrator of the platform is calling this endpoint, it will respond with a 401 (Unauthorized)

This endpoint unlocks the referenced account for the specified user previously locked.

### HTTP Request

`POST /users/:user_id/accounts/:account_id/unlock`

### Query Parameters

Parameter | Default | Description | Required
--------- | ------- | ----------- | --------
user_id | null | Id of the user whose accounts we want to unlock | True
account_id | null | Account and user we want to unlock | True

<aside class="warning">This action is considered an administrative one so it should be only called by an
administrator of the platform, failing otherwise</aside>



# Sessions

Sessions represent a sequential series of interactions for a given user on a system's website or application. 
They are associated with a single account (e.g. WedPics) not a user (i.e. linked set of accounts). 

## Create a new session

```javascript

data = {
  user_id: "<user_id>",
  service_id: "<service_id>"
}

$.post("/sessions", function(data, status){
});
```

```json
{
  "id": "<session_id>",
  "service_id": "<service_id>",
  "user": {
    "id": "<user_id>",
    "permissions": [...],
  },
  "expirates": "<expiration_date>"
}
```

An unique identifier is generated at session creation. The expiration is determined by the calling service. It may be non-expiring or programmatically expired via API call. It is up to the system or application creating and using the session. It depends on the use case for the specific system. 
For authenticated users are associated with an exact account with an ID. However, for non-authenticated users are associated with a "virtual" account without an ID. That is, the account returned by the user service has a set of permissions, assigned feature buckets, etc. and is fully specified except for the ID. However, it's considered ephemeral until the user logs in and the session becomes associated with an exact account.

This endpoint creates a new session for the specified user for the specified account


### HTTP Request

`POST /sessions`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
user_id | null | Id of the user who is creating the new session
service_id | null | Id of the service which is being used by the user


## Expire current session

When the user logouts from the system or an unauthenticated user leaves the website, the session should be expirated because the sequential series of interactions arrived to the end.


```javascript
$.delete("/sessions/:session_id", function(data, status){
});
```
> The above command returns 200 status code in case of successful expiration, error code in any other case.

This endpoint expirates the current session for the specified user for the specified service

### HTTP Request

`DELETE /sessions/:session_id`

### Query Parameters

Parameter | Default | Description | Required
--------- | ------- | ----------- | --------
session_id | null | Id of the session we want to finish | True

## Expire all sessions


```javascript
$.delete("/users/:user_id/sessions", function(data, status){
});
```
> The above command returns 200 status code in case of successful expiration of all sessions, error code in any other case.

This endpoint expirates all the open sessions related with the specified user

### HTTP Request

`DELETE /users/:user_id/sessions`

### Query Parameters

Parameter | Default | Description | Required
--------- | ------- | ----------- | --------
user_id | null | Id of the user we want to expire all the sessions | True


# Features

The user service should allow a configurable set of feature flags. Feature flags should be included in the config files, not database entries.

## Create a new feature flag

```javascript
data = {
  id_label: "<feature_id_label>",
  description_label: "<feature_description_label>",
  team: "<team_label>",
  owner: "<owner_email>",
  enabled: "<enabled_boolean>",
  buckets: ["<user_bucket_1>", "<user_bucket_2>"]
}

$.post("/features", function(data, status){
});
```

> The above command returns 201 status code (created) in case of successful creation, error code in any other case.

This endpoint creates a new feature flag

### HTTP Request

`POST /features`

### Query Parameters

Parameter | Default | Description | Required
--------- | ------- | ----------- | --------
id_label | "" | Used in the URL to show a specific setting | True
description_label | "" | Descriptive human readable feature label | False
team | "" | Short text label for which team is responsible for the feature | True
owner | "" | Internal mixbook email address for the developer that owns the feature | False
enabled | True | Global on/off switch to enable/disable the feature | False
buckets | [] | Possible buckets the user could be in | True

<aside class="warning">Buckets parameter should include at least 2 buckets and one of the buckets must be specified as the 'control' bucket</aside>

## Update an existing feature flag

```javascript
data = {
  id_label: "<feature_id_label>",
  description_label: "<feature_description_label>",
  team: "<team_label>",
  owner: "<owner_email>",
  enabled: "<enabled_boolean>",
  buckets: ["<user_bucket_1>", "<user_bucket_2>"]
}

$.put("/features/:feature_label", function(data, status){
});
```

> The above command returns 200 status code in case of successful update, error code in any other case.

This endpoint updates an existing feature flag

### HTTP Request

`PUT /features/:feature_label`

### Query Parameters

Parameter | Default | Description | Required
--------- | ------- | ----------- | --------
feature_label | "" | Label that identifies the feature we want to update | True
id_label | "" | Used in the URL to show a specific setting | True
description_label | "" | Descriptive human readable feature label | False
team | "" | Short text label for which team is responsible for the feature | False
owner | "" | Internal mixbook email address for the developer that owns the feature | False
enabled | True | Global on/off switch to enable/disable the feature | False
buckets | [] | Possible buckets the user could be in | False

<aside class="warning">Buckets parameter should include at least 2 buckets and one of the buckets must be specified as the 'control' bucket</aside>


## Delete exsiting feature flag

```javascript
$.delete("/features/:feature_label", function(data, status){
});
```

> The above command returns 200 status code in case of successful delete, error code in any other case.

This endpoint deletes an existing feature flag

### HTTP Request

`DELETE /features/:feature_label`

### Query Parameters

Parameter | Default | Description | Required
--------- | ------- | ----------- | --------
feature_label | "" | Label that identifies the feature we want to delete | True

# Services

## Status of service

```javascript
$.get("/services/:service_token", function(data, status){
});
```

```json
{
  "id": "<service_id>",
  "description": "<service_description>",
  "access": "<access_granted>",
  "expirates": "<expiration_date>"
}
```


> The above command returns 200 status code in case of successful delete, error code in any other case. It also returns
the status information of the specified service. In case a non administrator of the platform is calling this endpoint, it will respond with a 401 (Unauthorized)

This endpoint retrieves the status information about the specified service

### HTTP Request

`GET /services/:service_token`

### Query Parameters

Parameter | Default | Description | Required
--------- | ------- | ----------- | --------
service_token | null | Service token realted with the service we want to know the status information about | True

<aside class="warning">This action is considered an administrative one so it should be only called by an
administrator of the platform, failing otherwise</aside>



