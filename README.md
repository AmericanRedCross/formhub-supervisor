# Disaster Asset Manager

**American Red Cross and Kevin Lustig**

A Node.js server that enables storing and maintaining assets for disaster relief such as situation reports and maps. Assets can be accessed for display and use via a robust API.

## Installation

**Install via NPM**

```console
npm install
```

**Configure the application by modifying config.js in the project directory.**

* **siteName**: The name of the site as it will be displayed to your users in the site header.
* **description**: The description of the site as it will be displayed to your users on the homepage.
* **db**: The name of the MongoDB database you will use for this application (will store users, assets, and asset files.) When you start the application for the first time, this database will be created if it doesn't already exist.
* **port**: The port at which to run the application's public-facing server. If you do not have any other HTTP activity on your server, use port **80**. 
* **asset_opts > extents**: Available geographic area tags that can be applied to your assets for filtering and sorting. 
* **asset_opts > sectors**: Available keyword tags that can be applied to your assets for filtering and sorting.

**Start the application**

```console
node server
```

**Visit the application in your browser**

If you used port 80, the URL should simply be the IP address or URL of the server where you're running the application

```console
http://www.mydomain.com
```

If you used any other port, specify the port along with the IP address/URL (or set up a Virtual Host as appropriate to redirect traffic to the correct port)

```console
http://www.myassetmanager.com:myport
```

e.g.,

```console
http://www.redcross.org:8888
```

**Log in with the default super user**

```console
defaultUser@redcross.org/pa$$w0rd
```

**Go to /users and create a new user**

***Very Important:*** **Delete the default super user**

## API

The API methods provide access to the data in the asset manager in JSON format from any domain. Assets that are not marked as public are only accessible to users with ownership or super user permissions via the use of access tokens.

### Methods

All methods are accessible via GET request. 

Successful requests are responded to with the property **success: true** and a **response** property containing the data payload.

Failed requests or requests that produce no data are responded to with the property **success: false** and a human readable error message. 

**/api/asset/[id]**

Returns the data for a single asset identified by its MongoDB ID. If an asset cannot be found with the provided ID or the user doesn't have permission to view that asset, a failure response is provided.

**/api/assets**

Returns the data for all public assets by default. If an access token is provided, additional assets will be included as appropriate. 

Query parameters can also be included in the request, and results will be filtered to match these criteria. Parameters that do not match the asset schema will be ignored.

*Examples* 

```console
/api/assets?type=sitrep 
```

Limits the results to assets tagged as Situation Reports. 

```console
/api/assets?extent=World&extent=Nepal 
```

Limits the results to assets tagged as relevant to the World **or** to Nepal. 

```console
/api/assets?foo=bar 
```

Does nothing.

### Authentication

All authentication methods use the authentication gateway at **/api/authenticate**.

#### Pass-Through Method

To enable users of a different site or application to authenticate for API requests, use the pass-through method. 

Direct your users to the authentication gateway with your site's URL appended as the "from" value, like so:

```console
http://www.myassetmanager.com:myport/api/authenticate?from=www.myotherapplication.com
```

***Note:*** *Do not use "http://" or "https://" in the "from" URL.*

Once users provide their credentials, they will be redirected back to the URL you provided with a token parameter appended to the URL. 

```console
http://www.myotherapplication.com?token=12345.67890.12345
```

Use this token in your code to make API requests on behalf of the authenticated user.

#### AJAX Method

To avoid having to direct users to the authentication gateway, you can also use the AJAX method to submit user credentials to the asset manager to get an access token. 

Send the user's **email** and **password** via a POST request to the authentication gateway at:

```console
http://www.myassetmanager.com:myport/api/authenticate
```

If the credentials are valid, you will receive a success response that looks like:

```console
{
	success: true,
	response: {
		token: 12345.67890.12345
	}
}
```

#### Using the Access Token

Regardless of which method you use, tokens are valid for 24 hours from issue. 

To use an access token, append the token to your API requests in one of two ways:

**As a GET query parameter**

```console
http://www.myassetmanager.com:myport/api/assets?token=12345.67890.12345
```

**As a request header**

```console
x-access-token: 12345.67890.12345
```
