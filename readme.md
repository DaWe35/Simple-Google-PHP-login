
# Login with Google API using PHP

  

This is a mirror of [Login with Google API using PHP](http://usefulangle.com/post/9/google-login-api-with-php-curl).
Find the [original documentation & code here!](http://usefulangle.com/post/9/google-login-api-with-php-curl).



**Documentation backup**
This article was updated on March 15th 2019 and is edited as per the new Google API process.

The Javascript version of this tutorial is located at [Google Login with Javascript API](http://usefulangle.com/post/55/google-login-javascript-api).

You can implement Login with Google feature in your web application using its API. In this tutorial I will discuss its implementation.

Google also provides a PHP client library — it takes care of all the APIs that Google provides. But personally I find that client library a little too overwhelming. It is bulky in size and I don't understand the implementation clearly.

Writing custom code for Google login and getting the details of the user amounts to only 40 lines of PHP code, which is discussed in this tutorial.

#### Demo

[Click here for the demo application](https://usefulangle.com/demos/9/)

**Sample codes for download are provided at the end**.

#### Is Verification Required ?

Google is now requiring verification to use some privacy sensitive APIs. **However if you are looking just to get profile and email information from the user, you don't need verification.**

The API scopes used in this tutorial need no verification, and your application can be made public immediately.

#### Is HTTPS Required ?

To get only user profile infomation and email, Google does not require your application to be hosted over HTTPS. **It will work with HTTP also.**

#### Creating a Google App

The first step would be to create a Google App :

-   Go to [Google API Console](https://console.developers.google.com/)
-   If you have not created a project, create a project by clicking "**Select a project**" (at the top), and then clicking on the "+" button in the dialogbox. In the next screen enter your project name. ![](http://usefulangle.com/img/posts/9-select-project.png) ![](http://usefulangle.com/img/posts/9-new-project.png)
-   After the project is created, select the created project from the top dropdown.
-   Now click on **Credentials** tab on the left sidebar. In the next screen click on "**OAuth consent screen**". ![](http://usefulangle.com/img/posts/9-google-oauth-consent.png)  
    Fill out **Application Name** with your application name. When a user signs in, he will see this application name.  
    Fill **Authorized domains** with the domain from where you intend to run the application. If you are just testing it out on localhost, you can leave it blank.
-   Now click on the "**Credentials**" tab (just beside "**OAuth consent screen**"). In the screen, click on "**Create credentials**". Choose "**OAuth Client ID**" as the type. ![](http://usefulangle.com/img/posts/9-create-app-id.png)
-   In the next screen fill out the name. The Application type should be "**Web application**" ![](http://usefulangle.com/img/posts/9-app-details.png)
-   Add a redirect url in the section **Authorised redirect URIs**. This url should point to your redirect url script. (gauth.php in the attached codes). You can add a localhost url if you want.  
    Example of redirect url can be :  
    http://website.com/gauth.php  
    http://website.com/folder/gauth.php  
    http://localhost/folder-name/gauth.php
-   You can leave out **Authorised JavaScript origins** as blank. Click on the **Create** button.
-   On success you will get the App Client ID and App Secret. Save those as they will be required later.

#### Basic Understanding of the Login Process

Google implements OAuth2 for the login process. Google has done all the hard work and adding the login API to your code is fairly simple.

1.  You place a url in your HTML code that will redirect to Google's servers for login (called the OAuth login url).
2.  You also provide a redirect url, Google will redirect the user to this url after he does the login. Google will also pass an authorization code to this redirect url. Something like :
    
    ```markup
    http://website.com/redirect.php?code=546546554465
    ```
    
3.  It is in this redirect url script that you must use the passed authorization code (which is available as a GET parameter). This authorization code can be exchanged for an access token from Google (you have to implement an API call to get the access token from the authorization code).
4.  You can then use the access token to get user information such as id, name, picture, email etc.

#### Step 1 : Save the App Client ID and Client Secret

Use a settings.php file to save Client ID, Client Secret and Redirect Url

```php
<?php

/* Google App Client Id */
define('CLIENT_ID', 'xxxxxxxxxxxxxxxxxxxx');

/* Google App Client Secret */
define('CLIENT_SECRET', 'xxxxxxxxxxxxxxxxxxxx');

/* Google App Redirect Url */
define('CLIENT_REDIRECT_URL', 'xxxxxxxxxxxxxxxxxxxx');

?>

```

#### Step 2 : Add the Link in your code

Add the link to Google OAuth login url in your HTML code. If there is a need you can use PHP header function or Javascript document.location also to redirect to the Google login url.

```php
<?php

require_once('settings.php');

$login_url = 'https://accounts.google.com/o/oauth2/v2/auth?scope=' . urlencode('https://www.googleapis.com/auth/userinfo.profile https://www.googleapis.com/auth/userinfo.email') . '&redirect_uri=' . urlencode(CLIENT_REDIRECT_URL) . '&response_type=code&client_id=' . CLIENT_ID . '&access_type=online';

?>
<html>
<head>....</head>

<body>
	.....
	
	<a href="<?= $login_url ?>">Login with Google</a>

	.....
</body>
</html>

```

The login url is basically https://accounts.google.com/o/oauth2/v2/auth with five parameters :

-   scope : Scope is basically what you are looking to do or get from the user. Google provides a scope for each of its [API services](https://developers.google.com/identity/protocols/googlescopes). For just the login and getting user information (including email) you need 2 scopes —  
    i) https://www.googleapis.com/auth/userinfo.profile  
    ii) https://www.googleapis.com/auth/userinfo.email
-   redirect_uri : Your redirect url
-   response_type : Set it to the default value of "code"
-   client_id : Your Google App Client ID
-   access_type : Set it to "online"

Clicking on this link will redirect the user to Google where he does the login. After login Google redirects the user to the redirect url that you have provided. The redirect url script will handle the next steps.

#### Step 3 : Preparing the Redirect Url Script

On redirecting the user to your given redirect url Google passes an authorization code as a GET parameter named "code". A sample redirect url can look like :

```markup
http://website.com/redirect.php?code=546546554465
```

You must use this code and make an API call to get an access token. Below are the codes for the **API call to get the access token using the authorization code :**

```php
// $client_id, $redirect_uri & $client_secret come from the settings
// $code is the code passed to the redirect url
function GetAccessToken($client_id, $redirect_uri, $client_secret, $code) {	
	$url = 'https://www.googleapis.com/oauth2/v4/token';			

	$curlPost = 'client_id=' . $client_id . '&redirect_uri=' . $redirect_uri . '&client_secret=' . $client_secret . '&code='. $code . '&grant_type=authorization_code';
	$ch = curl_init();		
	curl_setopt($ch, CURLOPT_URL, $url);		
	curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);		
	curl_setopt($ch, CURLOPT_POST, 1);		
	curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE);
	curl_setopt($ch, CURLOPT_POSTFIELDS, $curlPost);	
	$data = json_decode(curl_exec($ch), true);
	$http_code = curl_getinfo($ch,CURLINFO_HTTP_CODE);		
	if($http_code != 200) 
		throw new Exception('Error : Failed to receieve access token');
	
	return $data;
}

```

After you get the access token you can make another API call to get the user profile information. Below are the codes for the **API call to get user profile information using the access token :**

```php
// $access_token is the access token you got earlier
function GetUserProfileInfo($access_token) {	
	$url = 'https://www.googleapis.com/oauth2/v2/userinfo?fields=name,email,gender,id,picture,verified_email';	
	
	$ch = curl_init();		
	curl_setopt($ch, CURLOPT_URL, $url);		
	curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
	curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE);
	curl_setopt($ch, CURLOPT_HTTPHEADER, array('Authorization: Bearer '. $access_token));
	$data = json_decode(curl_exec($ch), true);
	$http_code = curl_getinfo($ch, CURLINFO_HTTP_CODE);		
	if($http_code != 200) 
		throw new Exception('Error : Failed to get user information');
		
	return $data;
}

```

The overall code would look something like :

```php
<?php
// Holds the Google application Client Id, Client Secret and Redirect Url
require_once('settings.php');

// Holds the various APIs functions
require_once('google-login-api.php');

// Google passes a parameter 'code' in the Redirect Url
if(isset($_GET['code'])) {
	try {
		// Get the access token 
		$data = GetAccessToken(CLIENT_ID, CLIENT_REDIRECT_URL, CLIENT_SECRET, $_GET['code']);

		// Access Token
		$access_token = $data['access_token'];
		
		// Get user information
		$user_info = GetUserProfileInfo($access_token);
	}
	catch(Exception $e) {
		echo $e->getMessage();
		exit();
	}
}

?>

```

Previously Google+ API was used to get profile information. But that is deprecated now.

#### Sample Codes with Instructions

[Download](http://usefulangle.com/downloads/9-1.zip)

#### Need the Login in a New Window or Popup ?

#### Need Offline Access to Google APIs ?

This section deals with offline access — sometimes your application might need to access a Google API when the user is not present. However if your sole aim is to just implement a simple Google login you may stop at this point.

To get offline access you need to change the access_type parameter to "offline" in the Google login url.

```php
<?php

$login_url = 'https://accounts.google.com/o/oauth2/v2/auth?scope=' . urlencode('https://www.googleapis.com/auth/userinfo.profile https://www.googleapis.com/auth/userinfo.email https://www.googleapis.com/auth/plus.me') . '&redirect_uri=' . urlencode(CLIENT_REDIRECT_URL) . '&response_type=code&client_id=' . CLIENT_ID . '&access_type=offline';

?>

```

When access type is offline Google sends a refresh token in addition to an access token. Access token is valid only for an hour or so. The point of the refresh token is to refresh the access token.

The redirect url script :

```php
// Get the access token 
$data = GetAccessToken(CLIENT_ID, CLIENT_REDIRECT_URL, CLIENT_SECRET, $_GET['code']);

// Access Token
$access_token = $data['access_token'];

// Refresh Token
$refresh_token = $data['refresh_token'];

```

Ideally you should save the refresh token in your database because Google does not pass the refresh every time the API is called. Refresh token is passed only for the first API call.

However if you don't want to save the refresh token, you will have to re-prompt the user for consent before obtaining another refresh token. Re-prompting can done by adding a prompt parameter to the login url.

```php
<?php

$login_url = 'https://accounts.google.com/o/oauth2/v2/auth?scope=' . urlencode('https://www.googleapis.com/auth/userinfo.profile https://www.googleapis.com/auth/userinfo.email https://www.googleapis.com/auth/plus.me') . '&redirect_uri=' . urlencode(CLIENT_REDIRECT_URL) . '&response_type=code&client_id=' . CLIENT_ID . '&access_type=offline&prompt=consent';

?>

```

When you find that the access token has expired you should get a new access token using the refresh token. Below are the codes for the **The API call to get a new access token from the refresh token :**

```php
function GetRefreshedAccessToken($client_id, $refresh_token, $client_secret) {	
	$url_token = 'https://www.googleapis.com/oauth2/v4/token';			

	$curlPost = 'client_id=' . $client_id . '&client_secret=' . $client_secret . '&refresh_token='. $refresh_token . '&grant_type=refresh_token';
	$ch = curl_init();		
	curl_setopt($ch, CURLOPT_URL, $url_token);		
	curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);		
	curl_setopt($ch, CURLOPT_POST, 1);		
	curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE);
	curl_setopt($ch, CURLOPT_POSTFIELDS, $curlPost);	
	$data = json_decode(curl_exec($ch), true);	//print_r($data);
	$http_code = curl_getinfo($ch,CURLINFO_HTTP_CODE);		
	if($http_code != 200) 
		throw new Exception('Error : Failed to refresh access token');
	
	return $data;
}

```

See [Google APIs : Handling Access Token Expiration](http://usefulangle.com/post/11/php-google-apis-handling-access-token-expiration) to know more about handling the expiry time of the access token.

The new access token can be fetched in a manner like :

```php
$data = GetRefreshedAccessToken(CLIENT_ID, $_SESSION['refresh_token'], CLIENT_SECRET);

// New Access Token
$access_token = $data['access_token'];

```
