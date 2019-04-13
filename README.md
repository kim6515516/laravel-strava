# Laravel Strava Package

A laravel package to access data from the Strava API.

## Table of Contents

- [Strava Access Credentials](https://github.com/RichieMcMullen/strava#strava-access-credentials)
- [Installation](https://github.com/RichieMcMullen/strava#installation)
- [Publish Strava Config File](https://github.com/RichieMcMullen/strava#publish-strava-config-file)
- [Auto Discovery](https://github.com/RichieMcMullen/strava#auto-discovery)
  - [Provider](https://github.com/RichieMcMullen/strava#provider)
  - [Facade](https://github.com/RichieMcMullen/strava#alias--facade)
- [Usage](https://github.com/RichieMcMullen/strava#usage)
  - [Initialise Facade](https://github.com/RichieMcMullen/strava#use-strava-facade)
  - [Authenticate User](https://github.com/RichieMcMullen/strava#authenticate-user)
  - [Get Access Token](https://github.com/RichieMcMullen/strava#get-access-token)
  - [Access Token Expiry](https://github.com/RichieMcMullen/strava#access-token-expiry)
  - [Example Usage](https://github.com/RichieMcMullen/strava#example-usage)
  - [Unauthenticate User](https://github.com/RichieMcMullen/strava#unauthenticate-user)
- [Available Methods](https://github.com/RichieMcMullen/strava#usage)
  - [Athlete Data](https://github.com/RichieMcMullen/strava#athelete-data)
  - [User Activities Data](https://github.com/RichieMcMullen/strava#user-activities-data)
  - [User Single Activity](https://github.com/RichieMcMullen/strava#user-single-activity)
  - [Activity Comments](https://github.com/RichieMcMullen/strava#activity-comments)
  - [Activity Kudos](https://github.com/RichieMcMullen/strava#activity-kudos)
  - [Activity Laps](https://github.com/RichieMcMullen/strava#activity-laps)
  - [Activity Zones](https://github.com/RichieMcMullen/strava#activity-zones)
  - [Athlete Zones](https://github.com/RichieMcMullen/strava#athlete-zones)
  - [Athlete Stats](https://github.com/RichieMcMullen/strava#athlete-stats)
  - [Club](https://github.com/RichieMcMullen/strava#club)
  - [Club Members](https://github.com/RichieMcMullen/strava#club-members)
  - [Club Activities](https://github.com/RichieMcMullen/strava#club-activities)
  - [Club Admins](https://github.com/RichieMcMullen/strava#club-admins)
  - [Athlete Clubs](https://github.com/RichieMcMullen/strava#athlete-clubs)
  - [Gear](https://github.com/RichieMcMullen/strava#gear)
  - [Route](https://github.com/RichieMcMullen/strava#route)
  - [Athlete Routes](https://github.com/RichieMcMullen/strava#athlete-routes)
  - [Segment](https://github.com/RichieMcMullen/strava#segment)
  - [Segment Effort](https://github.com/RichieMcMullen/strava#segment-effort)
  - [Starred Segments](https://github.com/RichieMcMullen/strava#starred-segments)
- [Parameter Types](https://github.com/RichieMcMullen/strava#parameter-types)

## Strava Access Credentials

In order to use this package you will need to create an app from within your strava account [Create Strava App](https://www.strava.com/settings/api) to access your API credentials. Click here for more information on the [Strava API](https://developers.strava.com/).


## Installation

To install the package within your laravel project use the following composer command:

```shell
composer require codetoad/strava
```


## Publish Strava Config File

```shell
php artisan vendor:publish --provider="CodeToad\Strava\StravaServiceProvider"
```

This commmand will publish a file named `ct_strava.php` within your laravel project `config/ct_strava.php`. Edit this file with your Strava API credentials, generated from the Strava app you created.

```php
'client_id' => env('CT_STRAVA_CLIENT_ID', 'ADD STRAVA CLIENT ID HERE')
'client_secret' => env('CT_STRAVA_SECRET_ID', 'ADD STRAVA SECRET HERE')
'redirect_uri' => env('CT_STRAVA_REDIRECT_URI', 'ADD STRAVA REDIRECT URI HERE')
```

Alternatively you can ignore the above publish command and add this following variables to your `.env` file. Make sure to add your Strava App credentials

```shell
CT_STRAVA_CLIENT_ID=ADD STRAVA CLIENT ID HERE
CT_STRAVA_SECRET_ID=ADD STRAVA SECRET HERE
CT_STRAVA_REDIRECT_URI=ADD STRAVA REDIRECT URI HERE
```


## Auto Discovery

If you're using Laravel 5.5+ you don't need to manually add the service provider or facade. This will be Auto-Discovered. For all versions of Laravel below 5.5 add the ServiceProvider & StravaFacade to the appropriate arrays within your Laravel project `config/app.php`


#### Provider

```php
CodeToad\Strava\StravaServiceProvider::class,
```

#### Alias / Facade

```php
'Strava' => CodeToad\Strava\StravaFacade::class,
```


## Usage

#### Use Strava Facade

Add the Strava facade to the top of your controller so you can access the Strava class methods.

```php
use Strava;

class MyController extends Controller
{
  // Controller functions here...
}
```

#### Authenticate User

Redirect user to Strava to authenticate. If authentication is successful the user will be redirected to the `redirect_uri` that you added to the `config` file or your `.env` file

```php
Strava::authenticate();
```

#### Get Access Token

When returned to the redirected uri, use the following method within one of your controllers to generate the users Strava access token. The access token is generated via the `code` parameter within the redirected url. This token should be stored in a database or likewise for the related user so it can be used to access their Strava data using the methods below. A refresh token is also supplied in the response, be sure to save that too, as it will be needed to automatically generate new access tokens once the current token expires.

```php
$code = request()->code;
$token = Strava::token($code);
```

Example Response

```php
"token_type": "Bearer"
"expires_at": 1555165838
"expires_in": 21600 // 6 Hours
"refresh_token": "671129e56b1ce64d7e0c7e594cb6522b239464e1"
"access_token": "e105062b153da39f81a439b90b23357c741a40a0"
"athlete": ...
```

#### Access Token Expiry

Access tokens will now expire after 6 hours under the new flow that Strava have implemented and will need to be updated using a refresh token. A refresh token is retrieved from the initial access token request made in the previous step. When calling any of the methods below you might want to use a `try catch` block to check the current access token expiry, if expired you need to call the refresh token method. To refresh an access token take a look at the example code below.

```php
Strava::refreshToken($refreshToken);
```

#### Example Usage

This is an example of how you might fetch Strava data for a user using their authenticated `access token` that you have stored in your database. If theres an error while retrieving the data, it may be that the users access token has expired. In this case, the example below will use the user `refresh token` to automatically generate and store a new access token.

```php
try {

  # Get Athlete Data
  Strava::athlete($token); // Pass user access token

} catch (\Exception $e) {

  # If theres a problem getting the data from the method above,
  # it maybe that your access token has expired.

  # Lets use the users stored 'refresh_token' to obtain a new access token
  $data = Strava::refreshToken($refreshToken); // Pass user refresh token

  # Check the '$data' array contains both tokens
  if($data->access_token && $data->refresh_token)
  {
    # Update users Strava access and refresh tokens
    # Database update code here...

    # Call Athlete Data Method with New Token, for this request!
    Strava::athlete($data->access_token);

  }else{
    // log the error
    \Log::info($e->getMessage());
  }
}
```

All subsequent requests will use the token stored in your database, as long as it hasn't expired. If it has, it will go through the `catch` block and once again and refresh the users tokens.


#### Unauthenticate User

You can allow users to unauthenticate their Strava account with you Strava app. Simply allow users to call the following method, passing the access token that has been stored for their account.

```php
Strava::unauthenticate($token);
```

## Available Methods

All methods require an access token, some require additional parameters mentioned below. All "perPage" parameters default to 10 and it is an optional parameter.

#### Athlete Data

Returns the currently authenticated athlete.

```php
Strava::athlete($token);
```

#### User Activities Data

Returns the activities of an athlete.

```php
Strava::activities($token, $perPage);
```

#### User Single Activity

Returns the given activity that is owned by the authenticated athlete.

```php
Strava::activity($token, $activityID);
```

#### Activity Comments

Returns the comments on the given activity.

```php
Strava::activityComments($token, $activityID, $page, $perPage);
```

#### Activity Kudos

Returns the athletes who kudoed an activity.

```php
Strava::activityKudos($token, $activityID, $page, $perPage);
```

#### Activity Laps

Returns the laps data of an activity.

```php
Strava::activityLaps($token, $activityID);
```

#### Activity Zones

Summit Feature Required. Returns the zones of a given activity.

```php
Strava::activityZones($token, $activityID);
```

#### Athlete Zones

Returns the the authenticated athlete's heart rate and power zones.

```php
Strava::athleteZones($token);
```

#### Athlete Stats

Returns the activity stats of an athlete.

```php
Strava::athleteStats($token, $athleteID, $page, $perPage);
```

#### Club

Returns a given club using its identifier.

```php
Strava::club($token, $clubID);
```

#### Club Members

Returns a list of the athletes who are members of a given club.

```php
Strava::clubMembers($token, $clubID, $page, $perPage);
```

#### Club Activities

Retrieve recent activities from members of a specific club. The authenticated athlete must belong to the requested club in order to hit this endpoint. Pagination is supported. Athlete profile visibility is respected for all activities.

```php
Strava::clubActivities($token, $clubID, $page, $perPage);
```

#### Club Admins

Returns a list of the administrators of a given club.

```php
Strava::clubAdmins($token, $clubID, $page, $perPage);
```

#### Athlete Clubs

Returns a list of the clubs whose membership includes the authenticated athlete.

```php
Strava::athleteClubs($token, $page, $perPage);
```

#### Gear

Returns equipment data using gear ID.

```php
Strava::gear($token, $gearID);
```

#### Route

Returns a route using its route ID.

```php
Strava::route($token, $routeID);
```

#### Athlete Routes

Returns a list of the routes created by the authenticated athlete using their athlete ID.

```php
Strava::athleteRoutes($token, $athleteID, $page, $perPage);
```

#### Segment

Returns the specified segment.

```php
Strava::segment($token, $segmentID);
```

#### Segment Effort

Returns a segment effort from an activity that is owned by the authenticated athlete.

```php
Strava::segmentEffort($token, $segmentID);
```

#### Starred Segments

List of the authenticated athlete's starred segments.

```php
Strava::starredSegments($token, $page, $perPage);
```


## Parameter Types

```php
$token = String
$activityID = integer
$athleteID = integer
$clubID = integer
$gearID = integer
$routeID = integer
$segmentID = integer
$page = integer
$perPage = integer
```