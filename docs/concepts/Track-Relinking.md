# Track Relinking

The availability of a track depends on the country registered in the user’s Spotify profile settings. Often Spotify has several instances of a track in its catalogue, each available in a different set of markets. This commonly happens when the track the album is on has been released multiple times under different licenses in different markets.

These tracks are linked together so that when a user tries to play a track that isn’t available in their own market, the Spotify mobile, desktop, and web players try to play another instance of the track that is available in the user’s market.

<br>

## Track Relinking in the Web API

If your application supplies a `market` parameter in its call to the following track endpoints, the Web API will attempt to return information about alternative tracks that are available in the specified market:

-   [Get a Track](https://developer.spotify.com/documentation/web-api/reference/get-track)
-   [Get Several Tracks](https://developer.spotify.com/documentation/web-api/reference/get-several-tracks)
-   [Get an Album](https://developer.spotify.com/documentation/web-api/reference/get-an-album)
-   [Get Several Albums](https://developer.spotify.com/documentation/web-api/reference/get-multiple-albums)
-   [Get an Album’s Tracks](https://developer.spotify.com/documentation/web-api/reference/get-an-albums-tracks)
-   [Get a Playlist’s Tracks](https://developer.spotify.com/documentation/web-api/reference/get-playlists-tracks)
-   [Get a User’s Saved Tracks](https://developer.spotify.com/documentation/web-api/reference/get-users-saved-tracks)

When using the `market` query parameter, the response will contain another instance of the tracks only if the original track is unavailable and other instances of the track are available.

For example, the track “Heaven and Hell” by William Onyeabor is not available in the United States (market code US), as shown by the request to retrieve the track’s metadata:

```bash linenums="1"
curl -X GET "https://api.spotify.com/v1/tracks/6kLCHFM39wkFjOuyPGLGeQ"`
```

```json linenums="1"
{
  ...
  available_markets: [ "AT", "AU", "BE", "DK", "ES", "FI",
                       "FR", "HU", "IT", "PL", "PT", "SE",
                       "SK", "TR", "TW" ],
  ...
  uri: "spotify:track:6kLCHFM39wkFjOuyPGLGeQ"
}
```

If a `market` query parameter specifying the US market is appended to the call, the Web API recognizes that the specified track is unplayable and instead returns information about a track that is playable in the specified market. In addition it returns information about the original track:

```bash linenums="1"
curl -X GET "https://api.spotify.com/v1/tracks/6kLCHFM39wkFjOuyPGLGeQ?market=US"`
```

```json linenums="1"
{
	...
	is_playable: true
	linked_from: {
		external_urls: {
			spotify: "https://open.spotify.com/track/6kLCHFM39wkFjOuyPGLGeQ"
		},
		href: "https://api.spotify.com/v1/tracks/6kLCHFM39wkFjOuyPGLGeQ",
		id: "6kLCHFM39wkFjOuyPGLGeQ",
		type: "track",
		uri: "spotify:track:6kLCHFM39wkFjOuyPGLGeQ"
	},
	...
	uri: "spotify:track:6ozxplTAjWO0BlUxN8ia0A"
}
```

There are a number of important differences between the response you get with and without the `market` query parameter.

When the `market` parameter is supplied:

-   The `available_markets` property in the [Track](https://developer.spotify.com/documentation/web-api/reference/get-track) object is replaced by the `is_playable` property. (Since the request contains the `market` query parameter, there’s no need for the `available_markets` property to determine if the user can play the track or not.)
-   If the track has been relinked, the response contains a `linked_from` object containing information about the original track. In the example above, the track that was requested had the Spotify URI `spotify:track:6kLCHFM39wkFjOuyPGLGeQ`. Since it’s been relinked, this original [track URI](Spotify-URIs-and-IDs.md) can be found in the `linked_from` object. The parent track object now contains metadata about the relinked track with URI `spotify:track:6ozxplTAjWO0BlUxN8ia0A`.
-   If the `is_playable` property is `false`, the original track is not available in the given market, and Spotify did not have any tracks to relink it with. The track response will still contain metadata for the original track, and a `restrictions` object containing the reason why the track is not available: `"restrictions" : {"reason" : "market"}`.
-   If the `is_playable` property is `true`, the `linked_from` object may or may not exist depending on whether the original track was available in the market. If the `linked_from` object exists, the original track has been relinked.

!!! info

    IMPORTANT: If you plan to do further operations on tracks (for example, removing the track from a playlist or saving it to “Your Music”), it is important that you operate on the **original** track id found in the `linked_from` object. Using the ID of the linked track returned at the root level will likely return an error or other unexpected result.

Valid `market` query parameter values are [ISO 3166-1 alpha-2 codes](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2), as well as `from_token`. If `from_token` is used, an access token tied to a user must be supplied; `from_token` is the same thing as setting the market parameter to the [user’s](https://developer.spotify.com/documentation/web-api/reference/get-current-users-profile) country.
