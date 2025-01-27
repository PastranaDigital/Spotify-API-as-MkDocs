# Authorization Code with PKCE

The authorization code flow with PKCE is the recommended authorization flow if you’re implementing authorization in a mobile app, single page web apps, or any other type of application where the client secret can’t be safely stored.

The implementation of the PKCE extension consists of the following steps:

-   [Code Challenge](#code-challenge) generation from a [Code Verifier](#code-verifier).
-   [Request authorization](#request-user-authorization) from the user and retrieve the authorization code.
-   [Request an access token](#request-an-access-token) from the authorization code.
-   Finally, use the access token to make API calls.

<br>

#### Pre-requisites

This guide assumes that:

-   You have read the [authorization guide](../concepts/Authorization.md).
-   You have created an app following the [apps guide](../concepts/Apps.md).

<br>

#### Example

You can find an example app implementing Authorization Code flow with PKCE extension on GitHub in the [web-api-examples](https://github.com/spotify/web-api-examples/tree/master/authorization/authorization_code_pkce) repository.

<br>

## Code Verifier

The PKCE authorization flow starts with the creation of a code verifier. According to the [PKCE standard](https://datatracker.ietf.org/doc/html/rfc7636#section-4.1), a code verifier is a high-entropy cryptographic random string with a length between 43 and 128 characters (the longer the better). It can contain letters, digits, underscores, periods, hyphens, or tildes.

The code verifier could be implemented using the following JavaScript function:

```js linenums="1"
const generateRandomString = (length) => {
	const possible = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
	const values = crypto.getRandomValues(new Uint8Array(length));
	return values.reduce((acc, x) => acc + possible[x % possible.length], '');
};

const codeVerifier = generateRandomString(64);
```

<br>

## Code Challenge

Once the code verifier has been generated, we must transform (hash) it using the SHA256 algorithm. This is the value that will be sent within the user authorization request.

Let's use [window.crypto.subtle.digest](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/digest) to generate the value using the SHA256 algorithm from the given data:

```js linenums="1"
const sha256 = async (plain) => {
	const encoder = new TextEncoder();
	const data = encoder.encode(plain);
	return window.crypto.subtle.digest('SHA-256', data);
};
```

Next, we will implement a function `base64encode` that returns the `base64` representation of the digest we just calculated with the `sha256` function:

```js linenums="1"
const base64encode = (input) => {
	return btoa(String.fromCharCode(...new Uint8Array(input)))
		.replace(/=/g, '')
		.replace(/\+/g, '-')
		.replace(/\//g, '_');
};
```

Let's put all the pieces together to implement the code challenge generation:

```js linenums="1"
const hashed = await sha256(codeVerifier);
const codeChallenge = base64encode(hashed);
```

<br>

## Request User Authorization

To request authorization from the user, a `GET` request must be made to the `/authorize` endpoint. This request should include the same parameters as the [authorization code flow](Authorization-code.md), along with two additional parameters: `code_challenge` and `code_challenge_method`:

| Query Parameter       | Relevance                            | Value                                                                                                                                                                                                                                                                                                                                                                                                                         |
| --------------------- | ------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| client_id             | _Required_                           | The Client ID generated after registering your application.                                                                                                                                                                                                                                                                                                                                                                   |
| response_type         | _Required_                           | Set to `code`.                                                                                                                                                                                                                                                                                                                                                                                                                |
| redirect_uri          | _Required_                           | The URI to redirect to after the user grants or denies permission. This URI needs to have been entered in the Redirect URI allowlist that you specified when you registered your application (See the [app guide](../concepts/Apps.md)). The value of `redirect_uri` here must exactly match one of the values you entered when you registered your application, including upper or lowercase, terminating slashes, and such. |
| state                 | _Optional, but strongly recommended_ | This provides protection against attacks such as cross-site request forgery. See [RFC-6749](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1).                                                                                                                                                                                                                                                                       |
| scope                 | _Optional_                           | A space-separated list of [scopes](../concepts/Scopes.md). If no scopes are specified, authorization will be granted only to access publicly available information: that is, only information normally visible in the Spotify desktop, web, and mobile players.                                                                                                                                                               |
| code_challenge_method | _Required_                           | Set to `S256`.                                                                                                                                                                                                                                                                                                                                                                                                                |
| code_challenge        | _Required_                           | Set to the code challenge that your app calculated in the previous step.                                                                                                                                                                                                                                                                                                                                                      |

The code for requesting user authorization looks as follows:

```js linenums="1"
const clientId = 'YOUR_CLIENT_ID';
const redirectUri = 'http://localhost:8080';

const scope = 'user-read-private user-read-email';
const authUrl = new URL('https://accounts.spotify.com/authorize');

// generated in the previous step
window.localStorage.setItem('code_verifier', codeVerifier);

const params = {
	response_type: 'code',
	client_id: clientId,
	scope,
	code_challenge_method: 'S256',
	code_challenge: codeChallenge,
	redirect_uri: redirectUri,
};

authUrl.search = new URLSearchParams(params).toString();
window.location.href = authUrl.toString();
```

The app generates a PKCE code challenge and redirects to the Spotify authorization server login page by updating the `window.location` object value. This allows the user to grant permissions to our application

Please note that the code verifier value is stored locally using the `localStorage` JavaScript property for use in the next step of the authorization flow.

<br>

### Response

If the user accepts the requested permissions, the OAuth service redirects the user back to the URL specified in the `redirect_uri` field. This callback contains two query parameters within the URL:

| Query Parameter | Value                                                            |
| --------------- | ---------------------------------------------------------------- |
| code            | An authorization code that can be exchanged for an access token. |
| state           | The value of the state parameter supplied in the request.        |

We must then parse the URL to retrieve the code parameter:

```js linenums="1"
const urlParams = new URLSearchParams(window.location.search);
let code = urlParams.get('code');
```

The code will be necessary to request the access token in the next step.

If the user does not accept your request or if an error has occurred, the response query string contains the following parameters:

| Query Parameter | Value                                                         |
| --------------- | ------------------------------------------------------------- |
| error           | The reason authorization failed, for example: "access_denied" |
| state           | The value of the state parameter supplied in the request.     |

<br>

## Request an access token

After the user accepts the authorization request of the previous step, we can exchange the authorization code for an access token. We must send a `POST` request to the `/api/token` endpoint with the following parameters:

| Body Parameters | Relevance  | Value                                                                                                                                                                                                    |
| --------------- | ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| grant_type      | _Required_ | This field must contain the value `authorization_code`.                                                                                                                                                  |
| code            | _Required_ | The authorization code returned from the previous request.                                                                                                                                               |
| redirect_uri    | _Required_ | This parameter is used for validation only (there is no actual redirection). The value of this parameter must exactly match the value of `redirect_uri` supplied when requesting the authorization code. |
| client_id       | _Required_ | The client ID for your app, available from the developer dashboard.                                                                                                                                      |
| code_verifier   | _Required_ | The value of this parameter must match the value of the `code_verifier` that your app generated in the previous step.                                                                                    |

The request must include the following HTTP header:

| Header Parameter | Relevance  | Value                                       |
| ---------------- | ---------- | ------------------------------------------- |
| Content-Type     | _Required_ | Set to `application/x-www-form-urlencoded`. |

The request of the token could be implemented with the following JavaScript function:

```js linenums="1"
const getToken = async (code) => {
	// stored in the previous step
	let codeVerifier = localStorage.getItem('code_verifier');

	const payload = {
		method: 'POST',
		headers: {
			'Content-Type': 'application/x-www-form-urlencoded',
		},
		body: new URLSearchParams({
			client_id: clientId,
			grant_type: 'authorization_code',
			code,
			redirect_uri: redirectUri,
			code_verifier: codeVerifier,
		}),
	};

	const body = await fetch(url, payload);
	const response = await body.json();

	localStorage.setItem('access_token', response.access_token);
};
```

<br>

### Response

On success, the response will have a `200 OK` status and the following JSON data in the response body:

| key           | Type   | Description                                                                                        |
| ------------- | ------ | -------------------------------------------------------------------------------------------------- |
| access_token  | string | An access token that can be provided in subsequent calls, for example to Spotify Web API services. |
| token_type    | string | How the access token may be used: always "Bearer".                                                 |
| scope         | string | A space-separated list of scopes which have been granted for this `access_token`                   |
| expires_in    | int    | The time period (in seconds) for which the access token is valid.                                  |
| refresh_token | string | See [refreshing tokens](Refreshing-tokens.md).                                                     |

<br>

## What's next?

-   Great! We have the access token. Now you might be wondering: _what do I do with it_? Take a look at to the [access token](../concepts/Access-Token.md) guide to learn how to make an API call using your new fresh access token.

-   If your access token has expired, you can learn how to issue a new one without requiring users to reauthorize your application by reading the [refresh token](Refreshing-tokens.md) guide.
