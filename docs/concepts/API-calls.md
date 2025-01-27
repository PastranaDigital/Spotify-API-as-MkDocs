# API calls

The Spotify Web API is a restful API with different endpoints which return JSON metadata about music artists, albums, and tracks, directly from the Spotify Data Catalogue.

<br>

## Base URL

The base address of Web API is `https://api.spotify.com`.

<br>

## Authorization

All requests to Spotify Web API require authorization. Make sure you have read the [authorization](./Authorization.md) guide to understand the basics.

To access private data through the Web API, such as user profiles and playlists, an application must get the user’s permission to access the data.

<br>

## Requests

Data resources are accessed via standard HTTP requests in UTF-8 format to an API endpoint. The Web API uses the following HTTP verbs:

| Method | Action                                           |
| ------ | ------------------------------------------------ |
| GET    | Retrieves resources                              |
| POST   | Creates resources                                |
| PUT    | Changes and/or replaces resources or collections |
| DELETE | Deletes resources                                |

<br>

## Responses

Web API normally returns JSON in the response body. Some endpoints (e.g [Change Playlist Details](https://developer.spotify.com/documentation/web-api/reference/change-playlist-details)) don't return JSON but the HTTP status code

<br>

### Response Status Codes

Web API uses the following response status codes, as defined in the [RFC 2616](https://www.ietf.org/rfc/rfc2616.txt) and [RFC 6585](https://www.ietf.org/rfc/rfc6585.txt):

| Status Code | Description                                                                                                                                                                                                            |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 200         | OK - The request has succeeded. The client can read the result of the request in the body and the headers of the response.                                                                                             |
| 201         | Created - The request has been fulfilled and resulted in a new resource being created.                                                                                                                                 |
| 202         | Accepted - The request has been accepted for processing, but the processing has not been completed.                                                                                                                    |
| 204         | No Content - The request has succeeded but returns no message body.                                                                                                                                                    |
| 304         | Not Modified. See Conditional requests.                                                                                                                                                                                |
| 400         | Bad Request - The request could not be understood by the server due to malformed syntax. The message body will contain more information; see Response Schema.                                                          |
| 401         | Unauthorized - The request requires user authentication or, if the request included authorization credentials, authorization has been refused for those credentials.                                                   |
| 403         | Forbidden - The server understood the request, but is refusing to fulfill it.                                                                                                                                          |
| 404         | Not Found - The requested resource could not be found. This error can be due to a temporary or permanent condition.                                                                                                    |
| 429         | Too Many Requests - Rate limiting has been applied.                                                                                                                                                                    |
| 500         | Internal Server Error. You should never receive this error because our clever coders catch them all ... but if you are unlucky enough to get one, please report it to us through a comment at the bottom of this page. |
| 502         | Bad Gateway - The server was acting as a gateway or proxy and received an invalid response from the upstream server.                                                                                                   |
| 503         | Service Unavailable - The server is currently unable to handle the request due to a temporary condition which will be alleviated after some delay. You can choose to resend the request again.                         |

<br>

## Response Error

Web API uses two different formats to describe an error:

-   Authentication Error Object
-   Regular Error Object

<br>

#### Authentication Error Object

Whenever the application makes requests related to authentication or authorization to Web API, such as retrieving an access token or refreshing an access token, the error response follows [RFC 6749](https://tools.ietf.org/html/rfc6749) on the OAuth 2.0 Authorization Framework.

| Key               | Value Type | Value Description                                                                                                                         |
| ----------------- | ---------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| error             | string     | A high level description of the error as specified in [RFC 6749 Section 5.2](https://tools.ietf.org/html/rfc6749#section-5.2).            |
| error_description | string     | A more detailed description of the error as specified in [RFC 6749 Section 4.1.2.1](https://tools.ietf.org/html/rfc6749#section-4.1.2.1). |

Here is an example of a failing request to refresh an access token.

```bash linenums="1"
$ curl -H "Authorization: Basic Yjc...cK" -d grant_type=refresh_token -d refresh_token=AQD...f0 "https://accounts.spotify.com/api/token"

{
	"error": "invalid_client",
	"error_description": "Invalid client secret"
}
```

<br>

#### Regular Error Object

Apart from the response code, unsuccessful responses return a JSON object containing the following information:

| Key     | Value Type | Value Description                                                                                                                                                                                                      |
| ------- | ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| status  | integer    | The HTTP status code that is also returned in the response header. For further information, see [Response Status Codes](https://developer.spotify.com/documentation/web-api/concepts/api-calls#response-status-codes). |
| message | string     | A short description of the cause of the error.                                                                                                                                                                         |

Here, for example is the error that occurs when trying to fetch information for a non-existent track:

```bash linenums="1"
$ curl -i "https://api.spotify.com/v1/tracks/2KrxsD86ARO5beq7Q0Drfqa"

HTTP/1.1 400 Bad Request
{
	"error": {
		"status": 400,
		"message": "invalid id"
	}
}
```

<br>

## Conditional Requests

Most API responses contain appropriate cache-control headers set to assist in client-side caching:

-   If you have cached a response, do not request it again until the response has expired.
-   If the response contains an ETag, set the If-None-Match request header to the ETag value.
-   If the response has not changed, the Spotify service responds quickly with **304 Not Modified** status, meaning that your cached version is still good and your application should use it.

<br>

## Timestamps

Timestamps are returned in [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601) format as [Coordinated Universal Time (UTC)](http://en.wikipedia.org/wiki/Offset_to_Coordinated_Universal_Time) with a zero offset: `YYYY-MM-DDTHH:MM:SSZ`. If the time is imprecise (for example, the date/time of an album release), an additional field indicates the precision; see for example, `release_date` in an [Album Object](https://developer.spotify.com/documentation/web-api/reference/get-an-album).

<br>

## Pagination

Some endpoints support a way of paging the dataset, taking an offset and limit as query parameters:

```bash linenums="1"
$ curl
https://api.spotify.com/v1/artists/1vCWHaC5f2uS3yhpwWbIA6/albums?album_type=SINGLE&offset=20&limit=10
```

In this example, in a list of 50 (`total`) singles by the specified artist : From the twentieth (`offset`) single, retrieve the next 10 (`limit`) singles.
