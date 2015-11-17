# Obtaining OAuth 2.0 access tokens

***

This flow has the following steps:

## Request an access token

> Note: Requests to CDS's authorization server must use https instead of http because the server is only accessible over SSL (HTTPs) and refuses HTTP connections.

When a user first tries to perform an action that requires API authentication, you need to direct the user to CDS's authorization server at `https://api.collectivedigitalstudio.com/auth/authorize`. The table below identifies the request parameters that you need to (or can) include in the URL. Note that the request URI that you construct must contain properly URL-escaped parameter values.

| Parameters | Explanation |
| ---------- | ----------- |
| client_id     | **Required.** The OAuth 2.0 client ID for your application. You are provided this value by CDS. |
| redirect_uri  | **Required.** A registered redirect_uri for your client ID. Register valid redirect URIs for your application with CDS. |
| response_type | **Required.** Determines whether the endpoint returns an authorization code. Set the parameter's value to code. |
| state         | *Optional.* Allows for passing of unique data to the redirect_uri, arrives as `state` GET parameter. |

```
 https://api.collectivedigitalstudio.com/auth/authorize?
   client_id=FY320dzWyvRN0o79rZ&
   redirect_uri=http%3A%2F%2Flocalhost%2Foauth2callback&
   response_type=code
```


## User Login

In this step, if the user is not currently logged into CDS's system, they are required to login. In this case, the `/auth/authorize` endpoint will redirect them to `https://api.collectivedigitalstudio.com/auth/oauth/login` and provide them a HTML login page. Following a successful login, they will be re-directed to `/auth/authorize`

## Handle response from CDS

After the user consents or refuses to grant access to your application, CDS redirects the user to the redirect_uri that you specified in first step.

CDS will append a code parameter to the redirect_uri. This value is a temporary authorization code that you can exchange for an access token as discussed in following step.

`http://localhost/oauth2callback?code=4/ux5gNj-_mIu4DOD_gNZdjX9EtOFf`


## Exchange authorization code for access token

Assuming the user has granted access to your application, exchange the authorization code obtained for an access token. To do so, send a POST request to `https://api.collectivedigitalstudio.com/auth/token` that includes the following key-value pairs in the request body:

| Parameter | Explanation |
| --------- | ----------- |
| code          | **Required.** The authorization code that CDS returned to your redirect_uri in previous step. |
| client_id     | **Required.** The OAuth 2.0 client ID for your application. This value is provided by CDS. |
| client_secret | **Required.** The client secret associated with your client ID. This value is provided by CDS. |
| redirect_uri  | **Required.** A registered redirect_uri for your client ID. This much match what you provided to CDS. |
| grant_type    | **Required.** Set this value to `authorization_code`. |

A sample request is displayed below:

```
POST /auth/token HTTP/1.1
Host: api.collectivedigitalstudio.com
Content-Type: application/x-www-form-urlencoded

code=4/ux5gNj-_mIu4DOD_gNZdjX9EtOFf&
client_id=FY320dzWyvRN0o79rZ&
client_secret=hDBmMRhz7eJRsM9Z2q1oFBSe&
redirect_uri=http://localhost/oauth2callback&
grant_type=authorization_code
```

## Process response and store tokens

CDS will respond to your POST request by returning a JSON object that contains a short-lived access token and a refresh token.

```
{
  "access_token" : "ya29.AHES6ZTtm7SuokEB-RGtbBty9IIlNiP9-eNMMQKtXdMP3sfjL1Fc",
  "token_type" : "Bearer",
  "expires_in" : 3600
}
```

**Note:** To account for legacy clients, we also return the same token as `access-token`. This field is deprecated and will be removed soon. Use `access_token` moving forward.

> Important: Your application should store both values in a secure, long-lived location that is accessible between different invocations of your application.

***

# Making an authorized API request

After obtaining an access token for a user, your application can use that token to submit authorized API requests on that user's behalf.

#### Specify the access token as the value of the Authorization: Bearer HTTP request header.

* This example is for a CDS Backend API request:

```
GET /me HTTP/1.1
Host: api.collectivedigitalstudio.com
Authorization: Bearer ACCESS_TOKEN
...
```

You can test this using cURL with the following command:

`curl -H "Authorization: Bearer ACCESS_TOKEN" https://api.collectivedigitalstudio.com/me`

* This example is for a CDS Backend API request:

```
GET /channel/list?apikey=API_KEY
Host: api.collectivedigitalstudio.com
Authorization: Bearer ACCESS_TOKEN

{
  "pagination": {
    "page": 1,
    "per": 10,
    "total": 1
  },
  "items": [
    {
      "description": null,
      "thumb_url_high": "https://yt3.ggpht.com/-DUtWsATGBZA/AAAAAAAAAAI/AAAAAAAAAAA/RB3j_zG6CA8/s240-c-k-no/photo.jpg",
      "subscribers": 3,
      "comments": 0,
      "videos": 4,
      "cms": 8,
      "vertical": "Comedy",
      "thumb_url_default": "https://yt3.ggpht.com/-DUtWsATGBZA/AAAAAAAAAAI/AAAAAAAAAAA/RB3j_zG6CA8/s88-c-k-no/photo.jpg",
      "title": "CollectiveDS Engineering",
      "created_on": "2014-07-22T23:59:34Z",
      "thumb_url_medium": "https://yt3.ggpht.com/-DUtWsATGBZA/AAAAAAAAAAI/AAAAAAAAAAA/RB3j_zG6CA8/s240-c-k-no/photo.jpg",
      "id": "UCGuTn01c_NLBPIDc9dx9oig",
      "added_cms": "2015-07-08T01:03:57Z",
      "read_permission": null,
      "views": 26
    }
  ]
}
```

You can test this using cURL with the following command:

`curl -H "Authorization: Bearer ACCESS_TOKEN" https://api.collectivedigitalstudio.com/channel/list?apikey=API_KEY`
