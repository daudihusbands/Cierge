# PwdLess
<img src="http://pwdless.biarity.me/images/PwdLessLogo.svg" width="150">

PwdLess is a free, open-source authentication server that allows you to register/login users without a password. This is achieved by sending a "magic link" containing a time-based one-time password (TOTP), possibly in the form of a URL. Once the user opens the link (or manually types the TOTP into your app), a JWT is generated for the user, authenticating their identity. PwdLess operates without a database (cache only) and only requires simple configuration to run. This makes it platform-agnostic so you can easily integrate it into any tech-stack.

For more information, visit the official website: http://pwdless.biarity.me/.

# Getting Started
Getting started with PwdLess is easy:

1. [Download a PwdLess release](https://github.com/PwdLess/PwdLess/releases) for your OS of choice
 > if you don't find a build for your OS, consider [building from source](#building From Source)

2. [Add configuration](#configuration) to the included `appsettings.json` file

3. Run PwdLess & [test it](#http Endpoints) to see if it works 

# Basic process

Here's an overview of how you can use PwdLess to authenticate a user (this is very similar to OAuth2 grants):
_Note: TOTP == "Time-based One-Time Password"_

1. Users provide their email address & are sent a TOTP
A user provides their email address to your website (ie. JS client). In turn, it makes an API call to PwdLess's `/auth/sendtotp?identifier=USER_EMAIL`. This will cause PwdLess to send the email a TOTP. The email server settings are easily configurable.

2. The user opens the TOTP URL or enters the TOTP into your app
Once your website recieves the TOTP the user recieved (by letting the user enter it manually or through query strings), you will begin requesting a JWT for the user. To do this, your website makes an API call to PwdLess's `/auth/totpToToken?totp=SUPPLIED_TOTP`. PwdLess will then respond with a signed JWT containing the user's email address.

3. You use the JWT to authenticate the user into your APIs
Since it is not possible to change the contents of a signed JWT (given that you validate it in your APIs), you can now be certain of the user's identity & proceed by including the JWT in the authorization header of all subsequent requests made by your website.

# HTTP Endpoints
PwdLess exposes the following HTTP API:

Arguments could be sent in a `GET` query string (as shown below), or as `POST` body values.

* `GET /auth/sendtotp?identifier=[IDENTIFIER]` where `[IDENTIFIER]` is the user's email
  * creates a TOTP/token pair, stores it in cache, and sends the TOTP to `[IDENTIFIER]`
  * responds `200` once email has been sent
  * responds `400` on any failiure (wrong email server settings, etc.)

* `GET /auth/totpToToken?totp=[TOTP]` where `[TOTP]` is the TOTP to exhcnage for a token 
  * searches cache for a token associated with given totp
  * responds `200` with the JWT (plaintext) if token found
  * responds `404` with if the token wasn't found (ie. expired)
  * responds `400` on any failiure

* `GET /auth/echo?echo=[TEXT]` where `[TEXT]` is some text to echo for testing
  * use to check if server is running properly

# Configuration
The configration is in present in the root folder, in `appsettings.json`. This tells PwdLess about everything it needs to know to start working.

A description of each configuration item:
```
  "PwdLess": {
    "Totp": {
      "Expiry": `int: the number of minutes to pass before a TOTP expires`,
      "Length": `int: the maximum length of a TOTP (cutoff at 36)`
    },
    "Jwt": {
      "SecretKey": `string: the key used to sign the JWTs to prevent it from being tampered, should only be present here and in your API for JWT validation`,
      "Issuer": `string: "iss" claim in generated JWTs`,
      "Expiry": `string: "exp" claim in generated JWTs, leave empty for 30 days`,
      "Audience": `string: "aud" claim in generated JWTs`
    },
    "EmailAuth": {
      "From": `string: email address to send emails from`,
      "Server": `string: SMTP mail server address`,
      "Port": `int: mail server port`,
      "SSL": `bool: should ssl/tls be used for email server?`,
      "Username": `string: email username`,
      "Password": `string: email password`
    },
    "EmailContents": {
      "Subject": `string: the subject of sent emails`,
      "Body": `string: the body of sent emails, you add here a string "{{totp}}" that will be replaced by the TOTP once the email is sent, see wiki entry on TOTPs in emails`,
      "BodyType":  `string: type of message body (ie. "plain" for plaintext and "html" for HTML)`
    }
  },
  ...
}
```
This configuration could also be provided in the form of environment variables, where nesting is acheived by using colons (ie. "EmailContents:Subject").

To change the url/port at which the server runs (default of http://localhost:5000), supply a command line argument of `--url` (ie. `--url http://localhost:9538`)

# Misc

* By default, an in-memory distributed ASP.NET Core cache used. This could easily be replaced by another one such as Redis by changing the injected caching service in the ASP.NET Core IoC container & building from source.
* For more information on the included templaing of TOTPs: https://github.com/PwdLess/PwdLess/wiki/Templating-&-TOTPs-in-emails
* For the JSON Schema of the configuration file: https://github.com/PwdLess/PwdLess/wiki/Configuration-JSON-Schema

# Building from source

This project is built on top of ASP.NET Core, which supports a variety of operating systems. Follow this guide for more information: https://docs.microsoft.com/en-us/dotnet/articles/core/deploying/.

# License, Contributions, & Support

This project is licensed under the permissive open source [MIT license](https://opensource.org/licenses/MIT). Feel free to contribute to this project in any way, any contributions are highly appreciated.










