go-signin-with-apple
======

![](https://img.shields.io/badge/golang-1.13-blue.svg?style=flat) [![codecov](https://codecov.io/gh/Timothylock/go-signin-with-apple/branch/master/graph/badge.svg)](https://codecov.io/gh/Timothylock/go-signin-with-apple)
 [![Build Status](https://travis-ci.com/Timothylock/go-signin-with-apple.svg?branch=master)](https://travis-ci.com/Timothylock/go-signin-with-apple) [![Codacy Badge](https://api.codacy.com/project/badge/Grade/b54cafe3d1884d9cbe9748839739265e)](https://www.codacy.com/manual/Timothylock/go-signin-with-apple?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=Timothylock/go-signin-with-apple&amp;utm_campaign=Badge_Grade)

go-signin-with-apple verifies the `sign-in with apple` token.

## TODO
- Validate Refresh Tokens
- Validate App Tokens (currently this only validates web tokens)

## Installation
```
go get github.com/Timothylock/go-signin-with-apple
import "github.com/Timothylock/go-signin-with-apple/apple"

```

## Usage
### Example
An example file can be found at [example/example_test.go](example/example_test.go)

It is as simple as doing this (remember to actually handle errors of course!): 

``` golang
import "github.com/Timothylock/go-signin-with-apple/apple"

...


// Your 10-character Team ID
teamID := "XXXXXXXXXX"

// ClientID is the "Services ID" value that you get when navigating to your "sign in with Apple"-enabled service ID
clientID := "com.your.app"

// Find the 10-char Key ID value from the portal
keyID := "XXXXXXXXXX"

// The contents of the p8 file/key you downloaded when you made the key in the portal
secret := `-----BEGIN PRIVATE KEY-----
YOUR_SECRET_PRIVATE_KEY
-----END PRIVATE KEY-----`

// Generate the client secret used to authenticate with Apple's validation servers
secret, _ := apple.GenerateClientSecret(secret, teamID, clientID, keyID)

// Generate a new validation client
client := apple.New()

var resp apple.ValidationResponse
vReq := apple.ValidationRequest{
	ClientID:     clientID,
	ClientSecret: secret,
	Code:         "the_token_to_validatte",
	RedirectURI:  "https://example.com", // This URL must be validated with apple in your service
	GrantType:    "authorization_code",
}

// Do the verification
_ = client.VerifyNonAppToken(context.Background(), vReq, &resp)
unique, _ := apple.GetUniqueID(resp.IDToken)

// Voila!
fmt.Println(unique)
```

### Generating Client Secret
Apple requires a JWT token along with your validation request to authenticate your request. A token can be generated by 
calling the `GenerateClientSecret` function included. Check [secret.go](secret.go) to see exactly how to obtain the 
parameters required by the function. Note that your account might not have permissions to view/create `service IDs` and 
`keys` required by this function. 

```
import "github.com/Timothylock/go-signin-with-apple/apple"

...

// Your 10-character Team ID
team_id := "XXXXXXXXXX"

// Your Services ID, e.g. com.aaronparecki.services
client_id := "come.change.me"

// Find the 10-char Key ID value from the portal
key_id := "XXXXXXXXXX"

secret := `Your key that starts in -----BEGIN PRIVATE KEY-----`

secret, _ := apples.GenerateClientSecret(secret, team_id, client_id, key_id)
fmt.Println(secret)
```

### Validating Token
To validate a token, you must create a new validation `Client` then call the respective `Verify` function.

```
import "github.com/Timothylock/go-signin-with-apple/apple"

...

// Generate a new validation client
client := apple.New()

vReq := apple.ValidationRequest{
	ClientID:     client_id,
	ClientSecret: secret,                    // generated from the above step
	Code:         "the_token_to_validatte",
	RedirectURI:  "https://example.com",     // This URL must be validated with apple in your service
	GrantType:    "authorization_code",
}

var resp apple.ValidationResponse

// Do the verification
err = client.VerifyNonAppToken(context.Background(), vReq, &resp)
if err != nil {
	fmt.Println(err.Error())
	return
}

fmt.Println(resp)

```

### Obtaining Unique Subject ID
Apple as of right now unfortunately does not return an email or name that you can use. If youw ant to use those, the 
clients have access to those.

A subject ID is however included in the `id_token` field of the response which when decoded, has a subject that can 
uniquely identify the user. A helper function is included to obtain this subject ID: `GetUniqueID`

```
import "github.com/Timothylock/go-signin-with-apple/apple"

...

... Code to validate token ...

reflect.TypeOf(response)         // ValidationResponse
reflect.TypeOf(response.IdToken) // String


id := apple.GetUniqueID(response.IdToken)
fmt.Println(id)
```

## Tests
Coming soon ._.

## Contributing
Make sure tests pass, submit a PR, and lets get going! 

## License
go-signin-with-apple verifies is licensed under the MIT.
