---
title: EVNotify API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - javascript

toc_footers:
  - <a href='#'>EVNotify</a>
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  #- errors

search: true
---

# Introduction

Welcome to the EVNotify API! You can use the API to set and fetch useful information.

To use the API, you can either send the requests directly (over Shell) or you can use the easy-to-use JavaScript Library, which you will find within the EVNotifyAPI Repository on GitHub.

This API is in an early-access development. Feel free to enhance this API Documentation and send suggestions to
improve the API by itself.

To be able to use this API, you must register an account. With registering an account, you agree to the terms of use
of EVNotify.

# Endpoints

The API for EVNotify can be accessed over HTTP as well as HTTPS. HTTP is deprecated and for security-reasons you
should use the HTTPS endpoint. Some requests may require HTTPS only, so HTTP will be removed some time.
If your application is still using the HTTP endpoint, you should be thinking about upgrading to HTTPS endpoint.

<aside class="notice">
HTTP endpoint uses port 8714 (http://evnotify.de:8714)
</aside>

<aside class="notice">
HTTPS endpoint uses port 8743 (https://evnotify.de:8743)
</aside>

<aside class="warning">
Using the HTTP endpoint is deprecated and is no longer recommended. You should switch to the HTTPS endpoint soon.
</aside>

# Authentication

To be able to interact with the EVNotify API, most requests requires an authentication.
This is done by providing your so called AKey, which is your account identifier, to the request together with
your personal token.
<aside class="warning">
Don't share your personal token with others! Keep it safe!
</aside>
## Get a new key

If you don't have an AKey yet, you have to create a new account in order to be able to interact with the whole API.
To get a new AKey, you can do so with the following request. This will return a currently unused AKey, which you can
register.

<aside class="notice">
This will not reserve the requested AKey. It just returns a possible AKey combination, which isn't in use yet.
</aside>

### HTTPS Request

`POST https://evnotify.de:8743/getkey`

```shell
curl "https://evnotify.de:8743/getkey"
  -H "Content-Type: application/json"
```

> The request returns JSON like this:

```json
{
  "akey": 1234
}
```

```javascript
var evnotify = new EVNotify();

// get an unused key
evnotify.getKey(function(err, key) {
  console.log('Key: ', key);  // Key: 1234 for example
});
```

## Register a new account
After retrieving an unused AKey, you can register it. You need to specify the AKey, as well as a password.
For your own safety, you should choose a strong password. The minimum length for a valid password is 6 characters.

<aside class="warning">
Currently there is no possibility to recover a lost password. You can only change the password if you know the old one.
</aside>

### HTTPS Request

`POST https://evnotify.de:8743/register`

### URL Parameters

Parameter | Description
--------- | -----------
akey | The AKey to register
password | The password to use

```shell
curl "https://evnotify.de:8743/register"
  -H "Content-Type: application/json"
  -X POST -d '{"akey":"akey","password":"password"}'
```

> The request returns JSON like this:

```json
{
  "message": "Registration successful",
  "token": "secrettoken"
}
```

```javascript
var evnotify = new EVNotify();

// register new account
evnotify.register('akey', 'password', function(err, token) {
  console.log('Token: ', token);  // Contains the token for authentication
});
```

<aside class="success">
You can save the returned token safely to use it later for direct authentication.
</aside>

## Login with existing account

Once you've registered an account - or already have an account, you can login with your credentials.
This will give you the token, which authenticates your account.

<aside class="notice">
As long as you don't force a renewal of your token, the token will stay the same.
Because of that you can save the token safely so you don't need to login every time.
</aside>

### HTTPS Request

`POST https://evnotify.de:8743/register`

### URL Parameters

Parameter | Description
--------- | -----------
akey | The AKey of the account to login
password | The password of the account

```shell
curl "https://evnotify.de:8743/login"
  -H "Content-Type: application/json"
  -X POST -d '{"akey":"akey","password":"password"}'
```

> The request returns JSON like this:

```json
{
  "message": "Login successful",
  "token": "secrettoken"
}
```

```javascript
var evnotify = new EVNotify();

// login with account
evnotify.login('akey', 'password', function(err, token) {
  console.log('Token: ', token);  // Contains the token for authentication
});
```

## Change the password for an account

In order to be able to change the password for an account, you need to first enter your old password.
Currently it is not possible to recover a lost password.

<aside class="warning">
Changing the password will NOT change the token. It just sets a new password, so next time
you try login, you will have to use the new password instead of the old one.
If you want to change the token instead, have a further look at the token renewal section.
</aside>

### HTTPS Request

`POST https://evnotify.de:8743/password`

### URL Parameters

Parameter | Description
--------- | -----------
akey | The AKey of the account to change the password for
token | The token of the account
password | The old password of the account
newpassword | The new password to set

```shell
curl "https://evnotify.de:8743/password"
  -H "Content-Type: application/json"
  -X POST -d '{"akey":"akey","token": "token","password":"oldpassword", "newpassword": "newpassword"}'
```

> The request returns JSON like this:

```json
{
  "message": "Password change succeeded"
}
```

```javascript
var evnotify = new EVNotify();

// change password for account
evnotify.changePW('akey', 'token', 'oldpassword', 'newpassword', function(err, changed) {
  console.log('Password changed: ', changed);  // True, if change was successful
});
```

## Get settings and stats from account
Every account has a collection of settings and stats. Those collection store information about the connection and settings for the state of charge monitoring and notification.

<aside class="notice">
This request returns all settings and stats. If you only want to fetch the current state of charge, you can use the syncSoC request.
This will decrease the data usage and processing time for the request.
</aside>

<aside class="notice">
If you want to get the settings without providing the password, use the sync request instead.
This require to have 'autoSync' enabled for the account.
</aside>

### HTTPS Request

`POST https://evnotify.de:8743/settings`

### URL Parameters

Parameter | Description
--------- | -----------
akey | The AKey of the account to retrieve the settings for
token | The token of the account
password | The password of the account
option | must be 'GET' to retrieve the settings

```shell
curl "https://evnotify.de:8743/settings"
  -H "Content-Type: application/json"
  -X POST -d '{"akey":"akey","token": "token","password":"password", "option": "GET"}'
```

> The request returns JSON like this:

```json
{
  "message": "Get settings succeeded",
  "settings": {
    "email": "email",
    "telegram": "telegram",
    "soc": "soc",
    "curSoC": "curSoC",
    "device": "device",
    "polling": "polling",
    "autoSync": "autoSync",
    "lng": "lng",
    "push": "push"
  }
}
```

```javascript
var evnotify = new EVNotify();

// get the settings and stats for account
evnotify.getSettings('akey', 'token', 'password', function(err, settingsObj) {
  console.log('Settings: ', settingsObj);  // Object containing all the settings and stats
});
```

## Set settings and stats for account
You can modify the settings for the state of charge monitoring, notification and other settings and stats for the account.
Be careful, since wrong changes can break a working notification or other things.

<aside class="warning">
Even when you only want to modify one setting property, you will need to send all the settings.
Otherwise missing keys will be reseted and emptied.
So it is recommended to first retrieve all the current settings with the getSettings request and then just modify the properties you want to change.
</aside>

<aside class="notice">
If you want to set the settings without providing the password, use the sync request instead.
This require to have 'autoSync' enabled for the account.
</aside>

### HTTPS Request

`POST https://evnotify.de:8743/settings`

### URL Parameters

Parameter | Description
--------- | -----------
akey | The AKey of the account to retrieve the settings for
token | The token of the account
password | The password of the account
option | must be 'SET' to apply the settings
optionObj | object containing all the properties to save (see getSettings properties output to get full list)

```shell
curl "https://evnotify.de:8743/settings"
  -H "Content-Type: application/json"
  -X POST -d '{"akey":"akey","token": "token","password":"password", "option": "SET", "optionObj": "{}"}'
```

> The request returns JSON like this:

```json
{
  "message": "Set settings succeeded"
}
```

```javascript
var evnotify = new EVNotify(),
    settingsObj = {}; // the settings object containing all properties

// set the settings and stats for account
evnotify.setSettings('akey', 'token', 'password', settingsObj, function(err, set) {
  console.log('Settings saved: ', set);  // True, if successful
});
```

## Renew the account token
In order to be able to renew the token of your account, you will need to provide the AKey as well as the password of the account.
Changing the account token in regular periods is a good security manner.

<aside class="warning">
Renewing the token instantly changes the token of the account, so the old token is no longer usable.
</aside>

### HTTPS Request

`POST https://evnotify.de:8743/token`

### URL Parameters

Parameter | Description
--------- | -----------
akey | The AKey of the account to renew the token for
password | The password of the account

```shell
curl "https://evnotify.de:8743/token"
  -H "Content-Type: application/json"
  -X POST -d '{"akey":"akey","password":"password"}'
```

> The request returns JSON like this:

```json
{
  "message": "Token renewed",
  "token": "newtoken"
}
```

```javascript
var evnotify = new EVNotify();

// renew token of the account
evnotify.renewToken('akey', 'password', function(err, token) {
  console.log('New token: ', token);  // The new token
});
```