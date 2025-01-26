# Access Token

The _access token_ is a string which contains the credentials and permissions that can be used to access a given resource (e.g artists, albums or tracks) or user's data (e.g your profile or your playlists).

To use the _access token_ you must include the following header in your API calls:

| Header        | Parameter Value                                                  |
| ------------- | ---------------------------------------------------------------- |
| Authorization | Valid access token following the format: `Bearer <Access Token>` |

Note that the _access token_ is valid for 1 hour (3600 seconds). After that time, the token expires and you need to request a new one.

<br>

## Examples

The following example uses `cURL` to retrieve information about a track using the [Get a track](https://developer.spotify.com/documentation/web-api/reference/get-track) endpoint:

```bash linenums="1"
curl --request GET \
 	'https://api.spotify.com/v1/tracks/2TpxZ7JUBn3uw46aR7qd6V' \
 	--header "Authorization: Bearer NgCXRK...MzYjw"
```

The following code implements the `getProfile()` function which performs the API call to the [Get Current User's Profile](https://developer.spotify.com/documentation/web-api/reference/get-current-users-profile) endpoint to retrieve the user profile related information:

```js linenums="1"
async function getProfile(accessToken) {
	let accessToken = localStorage.getItem('access_token');

	const response = await fetch('https://api.spotify.com/v1/me', {
		headers: {
			Authorization: 'Bearer ' + accessToken,
		},
	});

	const data = await response.json();
}
```
