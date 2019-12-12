# Laravel Socialite Azure Active Directory Plugin

## Installation

`composer require metrogistics/laravel-azure-ad-oauth`

If you are using Laravel 5.5 or greater, the service provider will be detected and installed by Laravel automatically. Otherwise you will need to add the service provider and the facade (optional) to the `config/app.php` file:

```
Metrogistics\AzureSocialite\ServiceProvider::class,
// ...
'AzureUser' => Metrogistics\AzureSocialite\AzureUserFacade::class,
```

Publish the config and override any defaults:

```
php artisan vendor publish
```

Add the necessary env vars:

```
AZURE_AD_CLIENT_ID=XXXX
AZURE_AD_CLIENT_SECRET=XXXX
```

The only changes you should have to make to your application are:

* You will need to make the password field in the users table nullable.
* You will need to have a `VARCHAR` field on the users table that is 36 characters long to store the Azure AD ID for the user. The default name for the field is `azure_id` but that can be changed in the config file: `'user_id_field' => 'azure_id',`.

## Usage

All you need to do to make use of Azure AD SSO is to point a user to the `/login/microsoft` route (configurable) for login. Once a user has been logged in, they will be redirect to the home page (also configurable).

After login, you can access the basic Laravel authenticate user as normal:

```
auth()->user();
```

If you need to set additional user fields when the user model is created at login, you may provide a callback via the `UserFactory::userCallback()` method. A good place to do so would be in your AppServiceProvider's `boot` method:

```
\Metrogistics\AzureSocialite\UserFactory::userCallback(function($new_user){
	$new_user->api_token = str_random(60);
});
```

## Azure AD Setup


### New version of the Application Registration Portal (post October 2019)

1. Open your browser to https://portal.azure.com/
2. Navigate to `Azure Active Directory` -> `App registrations`.
3. Click on the blue button `Register an application`
4. Provide a human readable name for the application.
5. Select the type of accounts under `Supported account types`.
6. Under `Redirect URI (optional)` select `Web` and add the url in the form

https://your.domain.name/login/microsoft/callback

Note: This must be a https address, not http.

7. Click on the `Register` button
8. In the right pane, top section, locate the `Application (client) ID` and copy it's value to AZURE_AD_CLIENT_ID in your .env file.
9. In the left menu bar under `Manage` click on "Certificates & Secrets". 
10. In the pane on the right that just opened, under "Client secrets", click on the "+ New client secret" button.
11. Enter a description for the key in the "Description" box
12. Select the expiry term you require (this authentication mechnism stops once this key expires)
13. Click on the "Add" button. The key will be displayed once only
14. Copy the key and add it to your .env file as the AZURE_AD_CLIENT_SECRET value.
15. Click on "API permissions" and add any permissions that may be required
16. Click on "Manifest" in the menu
17. Add the following to the end of the exiting manifest. (Don't forget to add a comma to the last item that's in the manifest before pasting)
```
    "appRoles": [
        {
            "allowedMemberTypes": [
                "User"
            ],
            "displayName": "Manager Role",
            "id": "123459e3-8d88-4d99-b630-b9642a70f51e",
            "isEnabled": true,
            "description": "Manage stuff with this role",
            "value": "manager"
        }
    ]
```
18. Change the "id" value to a unique GUID of your choosing (ie don't use the one in the example here).



### Legacy version of the Application Registration Portal. (Pre October 2019)

1. Navigate to `Azure Active Directory` -> `App registrations`.
2. Create a new application
  1. Choose a name
  2. Select the "Web app / API" Application Type
  3. Add the "Sign-on URL". This will typically be `https://domain.com/auth/login`
  4. Click "Create"
3. Click into the newly created app.
4. The "Application ID" is what you will need for your `AZURE_AD_CLIENT_ID` env variable.
5. Click into "Reply URLs". You will need to whitelist the redirection path for your app here. It will typically be `https://domain.com/login/microsoft/callback`. Click "Save"
6. Select the permissions required for you app in the "Required permissions" tab.
7. Add any necessary roles to the manifest:
  1. Click on the "Manifest" tab.
  2. Add roles as necessary using the following format:

		```
		"appRoles": [
		    {
		      "allowedMemberTypes": [
		        "User"
		      ],
		      "displayName": "Manager Role",
		      "id": "08b0e9e3-8d88-4d99-b630-b9642a70f51e",// Any unique GUID
		      "isEnabled": true,
		      "description": "Manage stuff with this role",
		      "value": "manager"
		    }
		  ],
		```
  3. Click "Save"
8. In the "Keys" tab, enter a description (something like "App Secret"). Set Duration to "Never Expires". Click "Save". Copy the whole key. This will not show again. You will need this value for the `AZURE_AD_CLIENT_SECRET` env variable.
9. Click on the "Managed application" link (It will be the name of the application);
10. Under the "Properties" tab, enable user sign-in. Make user assignment required. Click "Save".
11. Under the "Users and groups" tab, add users and their roles as needed.

