---
hide:
    - toc
---

# Getting started with Web API

This tutorial will help you to make your first Web API call by retriving an artist's metadata. The steps to do so are the following:

1. [Create an app](#create-an-app), if you haven't done so.
2. [Request an access token](#request-an-access-token).
3. Use the access token to [request the artist data](#request-artist-data).

Here we go, let's rock & roll!

<br>

## Prerequisites

-   This tutorial assumes you have a Spotify account (free or premium).
-   We will use cURL to make API calls. You can install it from [here](https://curl.se/download.html) our using the package manager of your choice.

<br>

## Set Up Your Account

Login to the [Spotify Developer Dashboard](https://developer.spotify.com/dashboard). If necessary, read the latest [Developer Terms of Service](https://developer.spotify.com/terms) to complete your account set up.

<br>

## Create an app

An app provides the _Client ID_ and _Client Secret_ needed to request an _access token_ by implementing any of the [authorization](./concepts/Authorization.md) flows.

To create an app, go to [your Dashboard](https://developer.spotify.com/dashboard), click on the _Create an app_ button and enter the following information:

-   App Name: _My App_
-   App Description: _This is my first Spotify app_
-   Redirect URI: You won't need this parameter in this example, so let's use `http://localhost:3000`.

Finally, check the _Developer Terms of Service_ checkbox and tap on the _Create_ button.

<br>

## Request an access token

The _access token_ is a string which contains the credentials and permissions that can be used to access a given resource (e.g artists, albums or tracks) or user's data (e.g your profile or your playlists).

In order to request the _access token_ you need to get your _Client_ID_ and _Client Secret_:

1. Go to the [Dashboard](https://developer.spotify.com/dashboard)
2. Click on the name of the app you have just created (My App)
3. Click on the _Settings_ button

The _Client ID_ can be found here. The _Client Secret_ can be found behind the _View client secret_ link.

With our credentials in hand, we are ready to request an access token. This tutorial uses the [Client Credentials](./tutorials/Client-credentials.md), so we must:

-   Send a POST request to the token endpoint URI.
-   Add the Content-Type header set to the application/x-www-form-urlencoded value.
-   Add a HTTP body containing the Client ID and Client Secret, along with the grant_type parameter set to client_credentials.

```bash linenums="1"
curl -X POST "https://accounts.spotify.com/api/token" \
 	 -H "Content-Type: application/x-www-form-urlencoded" \
 	 -d "grant_type=client_credentials&client_id=your-client-id&client_secret=your-client-secret"
```

The response will return an access token valid for 1 hour:

```json linenums="1"
{
	"access_token": "BQDBKJ5eo5jxbtpWjVOj7ryS84khybFpP_lTqzV7uV-T_m0cTfwvdn5BnBSKPxKgEb11",
	"token_type": "Bearer",
	"expires_in": 3600
}
```

<br>

## Request artist data

For this example, we will use the [Get Artist](https://developer.spotify.com/documentation/web-api/reference/get-an-artist) endpoint to request information about an artist. According to the API Reference, the endpoint needs the Spotify ID of the artist.

An easy way to get the Spotify ID of an artist is using the Spotify Desktop App:

1. Search the artist
2. Click on the three dots icon from the artist profile
3. Select _Share > Copy link to artist_. The Spotify ID is the value that comes right after the `open.spotify.com/artist` URI.

Our API call must include the _access token_ we have just generated using the `Authorization` header as follows:

```bash linenums="1"
curl "https://api.spotify.com/v1/artists/4Z8W4fKeB5YxbusRsdQVPb" \
 	 -H "Authorization: Bearer BQDBKJ5eo5jxbtpWjVOj7ryS84khybFpP_lTqzV7uV-T_m0cTfwvdn5BnBSKPxKgEb11"
```

If everything goes well, the API will return the following JSON response:

```json linenums="1"
{
	"external_urls": {
		"spotify": "https://open.spotify.com/artist/4Z8W4fKeB5YxbusRsdQVPb"
	},
	"followers": {
		"href": null,
		"total": 7625607
	},
	"genres": ["alternative rock", "art rock", "melancholia", "oxford indie", "permanent wave", "rock"],
	"href": "https://api.spotify.com/v1/artists/4Z8W4fKeB5YxbusRsdQVPb",
	"id": "4Z8W4fKeB5YxbusRsdQVPb",
	"images": [
		{
			"height": 640,
			"url": "https://i.scdn.co/image/ab6761610000e5eba03696716c9ee605006047fd",
			"width": 640
		},
		{
			"height": 320,
			"url": "https://i.scdn.co/image/ab67616100005174a03696716c9ee605006047fd",
			"width": 320
		},
		{
			"height": 160,
			"url": "https://i.scdn.co/image/ab6761610000f178a03696716c9ee605006047fd",
			"width": 160
		}
	],
	"name": "Radiohead",
	"popularity": 79,
	"type": "artist",
	"uri": "spotify:artist:4Z8W4fKeB5YxbusRsdQVPb"
}
```

Congratulations! You made your first API call to the Spotify Web API.

<br>

## Summary

-   The Spotify Web API provides different endpoints depending on the data we want to access. The API calls must include the `Authorization` header along with a valid access token.

-   This tutorial makes use of the [client credentials](./tutorials/Client-credentials.md) grant type to retrieve the access token. That works fine in scenarios where you control the API call to Spotify, for example where your backend is connecting to the Web API. It will not work in cases where your app will connect on behalf of a specific user, for example when getting private playlist or profile data.

<br>

## What's next?

-   The tutorial used the Spotify Desktop App to retrieve the Spotify ID of the artist. The ID can also be retrieved using the [Search endpoint](https://developer.spotify.com/documentation/web-api/reference/search). An interesting exercise would be to extend the example with a new API call to the /search endpoint. Do you accept the challenge?

-   The [authorization](./concepts/Authorization.md) guide provides detailed information about which authorization flow suits you best. Make sure you read it first!

-   You can continue your journey by reading the [API calls](./concepts/API-calls.md) guide which describes in detail the Web API request and responses.

-   Finally, if you are looking for a more practical documentation, you can follow the [Display your Spotify Profile Data in a Web App](./how-tos/display.md) how-to which implements a step-by-step web application using [authorization code flow](./concepts/Authorization.md) to request the _access token_.
