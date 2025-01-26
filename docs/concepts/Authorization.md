# Authorization

Authorization refers to the process of granting a user or application access permissions to Spotify data and features (e.g your application needs permission from a user to access their playlists).

Spotify implements the [OAuth 2.0](https://datatracker.ietf.org/doc/html/rfc6749) authorization framework:

![Auth Intro Framework](../assets/authorization_auth_intro.png)

Where:

-   _End User_ corresponds to the Spotify user. The _End User_ grants access to the protected resources (e.g. playlists, personal information, etc.)
-   _My App_ is the client that requests access to the protected resources (e.g. a mobile or web app).
-   _Server_ hosts the protected resources and provides authentication and authorization via OAuth 2.0.

The access to the protected resources is determined by one or several _scopes_. Scopes enable your application to access specific functionality (e.g. read a playlist, modify your library or just streaming) on behalf of a user. The set of scopes you set during the authorization, determines the access permissions that the user is asked to grant. You can find detailed information about scopes in the [scopes documentation](Scopes.md).

The authorization process requires valid _client credentials_: a client ID and a client secret. You can follow the [Apps guide](Apps.md) to learn how to generate them.

Once the authorization is granted, the authorization server issues an access token, which is used to make API calls on behalf the user or application.

The OAuth2 standard defines four grant types (or flows) to request and get an access token. Spotify implements the following ones:

-   [Authorization code](../tutorials/Authorization-code.md)
-   [Authorization code with PKCE extension](../tutorials/Authorization-code-PKCE.md)
-   [Client credentials](../tutorials/Client-credentials.md)
-   [Implicit grant](../tutorials/Implicit-grant.md)

<br>

## Which OAuth flow should I use?

Choosing one flow over the rest depends on the application you are building:

-   If you are developing a long-running application (e.g. web app running on the server) in which the user grants permission only once, and the client secret can be safely stored, then the [authorization code flow](../tutorials/Authorization-code.md) is the recommended choice.
-   In scenarios where storing the client secret is not safe (e.g. desktop, mobile apps or JavaScript web apps running in the browser), you can use the [authorization code with PKCE](../tutorials/Authorization-code-PKCE.md), as it provides protection against attacks where the authorization code may be intercepted.
-   For some applications running on the backend, such as CLIs or daemons, the system authenticates and authorizes the app rather than a user. For these scenarios, [Client credentials](../tutorials/Client-credentials.md) is the typical choice. This flow does not include user authorization, so only endpoints that do not request user information (e.g. user profile data) can be accessed.
-   The [implicit grant](../tutorials/Implicit-grant.md) has some important downsides: it returns the token in the URL instead of a trusted channel, and does not support refresh token. Thus, we don't recommend using this flow.

The following table summarizes the flows' behaviors:

| Flow                         | Access User Resources | Requires Secret Key (Server-Side) | Access Token Refresh |
| ---------------------------- | --------------------- | --------------------------------- | -------------------- |
| Authorization code           | Yes                   | Yes                               | Yes                  |
| Authorization code with PKCE | Yes                   | No                                | Yes                  |
| Client credentials           | No                    | Yes                               | No                   |
| Implicit grant               | Yes                   | No                                | No                   |
