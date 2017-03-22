# Oauth Sample Apps 

Each sample app has a README with instructions on running it and a Procfile for use with Foreman.

1. Python - uses the sanction OAuth library and web.py.
2. Ruby - uses Sinatra and RestClient, does not use an OAuth library.
3. Clojure - Uses Compojure, http-kit and hiccup. Does not use an OAuth library. 


## Client Ids
To create a new Oauth integration, you will need to create a client id for your new application. You can create/modify clients at `https://rally1.rallydev.com/login/accounts/index.html#/clients`  The page will ask you to enter a name and redirect_uri for your client. After saving it will give you a `client_id` and `client_secret` to use in your app. Zuul's oauth URLs are:

* Authentication `https://rally1.rallydev.com/login/oauth2/auth`
* Token `https://rally1.rallydev.com/login/oauth2/token`

## API Key
API keys are strings that you can use in place of a zuul session token or username/password combination. You can generate and name as many keys as needed at `https://rally1.rallydev.com/login/accounts/index.html#/keys` To use one, you can make calls to alm with a cookie or header called `zsessionid` set to the `secret_key` value of your API key. You can delete or regenerate an existing API Key to revoke access to keys you have previously issued.

## Revoking Grants
After you agree to let an oauth client access your data, you can remove the grant by visiting `https://rally1.rallydev.com/login/accounts/index.html#/apps` and clicking remove on the application you do not want to have access any longer.

## Workflow 

Go to `https://rally1.rallydev.com/login/accounts/index.html#/clients` and create a new Client for your application. Save the Client ID and Client Secret. 

To gain an access token for a user from your app first

redirect to `https://rally1.rallydev.com/login/oauth2/auth`

Encode the following parameters onto the URL

* `state` a key to use to validate the auth token, typically a UUID. 
* `response_type` set to 'code'.
* `redirect_uri` this must match the URL you specified when creating your client id and secret. 
* `client_id` is the client_id that was created in Rally. 
* `scope` set to 'alm'.

For example: `https://rally1.rallydev.com/login/oauth2/auth?state=e347b102-6029-49b0-81d7-5089d846812e&response_type=code&redirect_uri=http%3A%2F%2Flocalhost%3A4567%2Foauth-redirect&client_id=817b4273628c415cddfd657ab7224582&scope=alm`

In the handler for the URL you redirect to you need to get the code from the parameters and POST to the token endpoint. 

You will recieve a JSON body with 'state' and 'code'. 

* `state` must match the 'state' you specified earlier.
* `token` is the auth token that you can exchange for an access token. 

To exchange the auth token for an access token:

`POST to https://rally1.rallydev.com/login/oauth2/token`

The BODY must contain the following parameters in URL/form encoded format.

* `code` the 'code' you recieved.
* `redirect_uri` must match the redirect_uri you specified earlier.
* `grant_type` set to 'authorization_code'.
* `client_id` set to your client id.
* `client_secret` set to your client secret.

Ensure you set the Content-Type header to "application/x-www-form-urlencoded".

The response will either contain an error body in JSON such as

`{"client_id":null,	 "client_secret":null, "error_description":"Invalid value for grant_type params - must be authorization_code", "error":"invalid_grant"}`

or on success a JSON body of 

`{"id_token":"???","refresh_token":"8b829ee0a4cb45f6884f580433c98","expires_in":86400,"token_type":"Bearer","access_token":"Ovp8AQ4pRMADXUmzR44V5IFb8ViYUdxhYJkd123aQs"}`

You can then use the access_token to make requests using the Rally Web Services API. On each request set the zsessionid header to the access token.  
