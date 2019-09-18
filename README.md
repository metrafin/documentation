# Metrafin Documentation
> Metrafin Platform API Documentation

# Skip to

- [Creating an Application](#creating-an-application)
- [Authorization](#authorization)
- [Endpoints](#endpoints)
	- [Token management](#token-management)
	- [Profile information](#profile-information)
	- [Reputation reading](#reputation-reading)
		- [Get Permissions](#get-v1permissions)
		- [Get HonorScore](#get-v1honorscore)
	- [Reputation contribution](#reputation-contribution)
		- [List reputation events](#get-v1reputationeventlist)
		- [Create reputation event](#post-v1reputationevent)
		- [Disable or reenable reputation event](#post-v1reputationeventsetactive)
	- [Resolving usernames and user IDs](#resolving-usernames-and-user-ids)

# Introduction

The Metrafin API provides developers with a seamless, free, and public way to keep track of individual users and their reputations more effectively than ever. Metrafin prevents individual people from creating multiple accounts and provides all of its users with a globally unique Metrafin user ID. By assigning an ID to each person, consistent across the world as Metrafin expands, Metrafin can build a better online global ID for the future and provide a powerful reputation system alongside it.

Using Metrafin's reputation platform and the Metrafin global ID, the possibilities are endless for developers - we can build safer social media platforms, petition systems with actual meaning, and more - the possibilities are limitless, really.

Metrafin's uses OAuth for user account access management and provides a straightforward and modern API for developers to easily integrate applications. Metrafin protects user accounts using mandatory SMS 2FA for all users.

## Reputation overview

Using Metrafin's reputation system, your application can preemptively block users based on their behaviour in other applications. Metrafin provides two powerful reputation APIs: the Permissions API and the HonorScore API. The Permissions API should be used to automatically block user behaviour such as making public posts, commenting, making financial transactions, or creating group chats based on past behaviour online and the HonorScore API should be used to display a reputation score for users and moderators.

HonorScore is a scaled score which is greater than 0 and less than 100. It is rounded to 1 decimal place.

## Terms

By using the Metrafin API, you certify that you have read and agree to Metrafin's [Terms of Service](https://metrafin.com/terms.html).

# Creating an application

## Registering application

Register your application through your Metrafin account in the Metrafin [Developer Center](https://metrafin.com/developerCenter.html).

Tips:
- Use an application name and homepage domain your users will recognize. 
- Specify redirect URIs - these must be on the same hostname as your application's homepage or they can be mobile app URIs. (ex. `https://example.com/auth` or `myapp://auth`)
- You can use the Metrafin [OAuth URL generator tool](https://metrafin.com/oauthUrlGenerator.html) to create OAuth URLs easily.
- Full documentation for OAuth is available [from OAuth](https://oauth.net/2/). This explains the overall concept of OAuth.

You must place a `mf_manifest.json` file in the root folder on your domain's webserver to confirm your domain and allow users to authorize your application. That will generally appear in the following format:

```json
{
	"version": 1,
	"applications": [
		{
			"id": "84ed22b0-01cf-42d5-a344-cacd1944160a",
			"authorized": true
		}
	]
}
```

## Accepting access tokens

When a user is redirected to your redirect URI following authorization, an `accesstoken` will be provided in the query string of the page. Use this along with your application's private token to access the API to access information about the user. See [Authentication](#authentication) for more information about authentication.

For example, your user may be redirected to the URL `https://example.com/auth?accesstoken=66sULabGH9AK2dqjt0SVCzBq0BKN_mB2`.

# Authorization

To authenticate to endpoints, use an `Authorization` header containing just your application's private token, a colon, and an accesstoken provided to your application. (See [Accepting Access Tokens](#accepting-access-tokens) for more info about access tokens.)

For endpoints listed in the Endpoints section labeled "Auth: app_private:accesstoken", follow the `app_private_token:accesstoken` format. ex. `Authorization: ep7c02pVdLhp9tti9z3fj5aqT0ZlvmJ_qDb1wM7Xft4mI_Kf:66sULabGH9AK2dqjt0SVCzBq0BKN_mB2`

For endpoints labeled "Auth: app_private", just use your app's private token. ex. `Authorization: ep7c02pVdLhp9tti9z3fj5aqT0ZlvmJ_qDb1wM7Xft4mI_Kf`

# Endpoints

The API base URL is `https://api.metrafin.com`.

## Token management

### GET `/v1/token`

**Description**: Fetch information about an access token.

**Auth**: `app_private:accesstoken`

Example response:

```json
{
	"error": null,
	"scopes": [
		"profile_basic",
		"profile_home",
		"profile_phone",
		"profile_age",
		"reputation_read",
		"reputation_contribute"
	],
	"userid": "da53da25-2326-481c-9511-68ba549a42ad",
	"expires": "2019-09-25T05:02:39.586Z"
}
```

## Profile information

### GET `/v1/profile`

**Description**: Fetch information about a user's profile. Some information may be missing depending on the auth scopes granted to your application.

**OPTIONAL auth scopes**: profile_basic (name and pronoun), profile_home (home address), profile_phone (phone number), profile_age (age in years)

**Auth**: `app_private:accesstoken`

Example response:

```json
{
	"error": null,
	"userId": "da53da25-2326-481c-9511-68ba549a42ad",
	"username": "jason",
	"created": "2019-09-16T02:45:19.320Z",
	"verified": {
		"firstName": "Jason",
		"middleName": "Lee",
		"lastname": "Daniels",
		"country": "US",
		"homeAddress": {
			"full": "18 Hayward Street, San Francisco, CA 94925, USA",
			"line1": "18 Hayward Street",
			"line2": null,
			"city": "San Francisco",
			"postalCode": "94103",
			"administrativeRegion": "CA",
			"countryCode": "US"
		},
		"age": 23,
		"phone": "+14151234567"
	},
	"pronoun": "male"
}
```

## Reputation reading

### GET `/v1/permissions`

**Description**: Provides a user's global permissions. Your application should utilize these and check with Metrafin's permissions endpoint when the user tries to perform an action which may be limited by a permission.

**REQUIRED auth scopes**: NONE

**Auth**: `app_private:accesstoken`

Example response:

```json
{
  "error": null,
  "permissions": {
    "public_comment": true,
    "public_post_media": true,
    "public_post_text": true,
    "private_message": true,
    "private_group_message": false,
    "video_game": true,
    "gambling": true,
    "financial_transaction": true
  }
}
```

### GET `/v1/honorScore`

**Description**: Provides a user's HonorScore for displaying to moderators or users.

**REQUIRED auth scopes**: reputation_read

**Auth**: `app_private:accesstoken`

Example response:

```json
{
  "error": null,
  "honorScore": 62
}
```

## Reputation contribution

### POST `/v1/reputationEvent`

**Description**:

Creates a new reputation event for the user. Most apps are only allowed to create reputation events with type "negative".

The reputation event's tags and contexts should describe the circumstances of the user's action which led to the event being made as accurately as possible.

Reputation event contexts:
- "public_comment" - A public comment
- "public_post_text" - The text content of a public post
- "public_post_media" - The media content of a public post
- "private_message" - A one-on-one private message
- "private_group_message" - A private group message (ex. in a video game or group conversation)
- "video_game" - Generally an online, multiplayer video game
- "gambling" - A legal gambling service, offline or online
- "financial_transaction" - A financial transaction

Reputation event tags:
- "spam" - A user is spamming in some way.
- "harassment" - A user has engaged in harassing behaviour.
- "racism" - A user has engaged in racist behaviour.
- "homophobia" - A user has engaged in homophobic behaviour.
- "sexism" - A user has used sexist language or behaviour.
- "violence" - A user has used violent language when not acceptable.
- "hate" - A user has engaged in hateful behaviour.
- "unallowed_adult_content" - A user posted adult content in an area where it is not permitted.
- "nonconsensual_content" - A user posted nonconsensual photos or videos of someone else, especially pornography.
- "intellectual_property" - A user has infringed on someone's intellectual property rights.
- "unallowed_advertising" - A user has posted advertisements in a place where they are not allowed.
- "scam" - A user is attempting to scam others.
- "illegal_activity" - General illegal activity.
- "unfair_gameplay_advantage" - A user is cheating in a video game.
- "gameplay_bad_behavior" - A user is breaking rules or being destructive in a video game.
- "chargeback" - A user has issued a chargeback for a fulfilled purchase.
- "threat" - A user has issued a threat of some type.

**REQUIRED auth scopes**: reputation_contribute

**Auth**: `app_private:accesstoken`

Example request:

```json
{
	"context": "public_comment",
	"tag": "harassment",
	"type": "negative",
	"description": "Public harassment"	
}
```

Example response:

```json
{
  "error": null,
  "eventId": "bf07694d-3e22-42f5-aa66-6f642d944c04"
}
```

### GET `/v1/reputationEvent/list`

**Description**: Provides a user's reputation events assigned by your application in a history record.

**REQUIRED auth scopes**: NONE

**Auth**: `app_private:accesstoken`

Example response:

```json
{
  "error": null,
  "reputationEvents": [
    {
      "id": "75c70e0a-a1f1-432b-97da-68980747b354",
      "created": "2019-09-18T05:25:41.290Z",
      "type": "negative",
      "tag": "harassment",
      "active": true,
      "context": "public_comment",
      "description": "Public harassment"
    },
    {
      "id": "94c4fa88-fea8-42d1-8695-52a2389f87fe",
      "created": "2019-09-18T05:28:45.136Z",
      "type": "negative",
      "tag": "scam",
      "active": true,
      "context": "private_group_message",
      "description": "Attempted to convince group members to send payments, provided false reason."
    }
  ]
}
```

### POST `/v1/reputationEvent/setActive`

**Description**: This endpoint is used to enable or disable reputation events' effects on user permissions and HonorScores.

**REQUIRED auth scopes**: NONE

**Auth**: `app_private:accesstoken`

Example request:

```json
{
	"id": "dd233391-dd15-434c-b4b9-a33eff45405c",
	"active": false
}
```

Example response:

```json
{
  "error": null
}
```

## Resolving usernames and user IDs

Most applications will want to use usernames provided through Metrafin's `/v1/profile` endpoint as the usernames of their users. Applications should not cache the usernames provided by Metrafin and should only rely on user IDs, just in case Metrafin allows users to change their usernames in the future.

Instead of caching usernames on your end for display and lookups, use the following endpoint to:
- Get usernames by user ID for display of usernames
- Get user IDs from usernames for fetching a user by username

### POST `/v1/resolveUser`

**Description**: This endpoint is used to resolve usernames or user IDs from the other.

**REQUIRED auth scopes**: NONE

**Auth**: `app_private`

Example request for username -> user ID:

```json
{
	"resolveBy": "username",
	"value": "ethan"
}
```

Example request for user ID -> username:

```json
{
	"resolveBy": "userId",
	"value": "9e2b642f-14d2-4f26-a662-d1e4b653a0d9"
}
```

Example response:

```json
{
  "error": null,
  "userId": "9e2b642f-14d2-4f26-a662-d1e4b653a0d9",
  "username": "ethan"
}
```
