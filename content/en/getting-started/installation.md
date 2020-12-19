---
title: Installation
description: ''
position: 30
category: Getting Started
---

## Requirements

- Laravel 8
- MySQL 5.7
- Either the [BC Math](https://secure.php.net/manual/en/book.bc.php) or [GMP](https://secure.php.net/manual/en/book.gmp.php) extension.
- Elasticsearch 7.9

The default search implementation uses Elasticsearch, although this is configurable via the config and can be swapped out for another driver search will only work with the initial configuration using Elasticsearch.

The GetCandy API is a Laravel package, designed to be easily added to any new or existing Laravel application.

> If you're starting a brand new Laravel application, install by [following their documentation](https://laravel.com/docs/8.x) along with Sanctum as [per the documentation](https://laravel.com/docs/8.x/sanctum)


## Installations Steps

Require the API package

```bash
$ composer require getcandy/candy-api:^0.11-beta
```

Or add this to your `composer.json` file

```javascript
"getcandy/candy-api": "^0.11-beta"
```

#### A note on Vimeo uploads
<!-- (**might be able to remove this**) -->
GetCandy supports Vimeo as an asset driver, however to [due to an issue in the repo](https://github.com/vimeo/laravel/issues/74), this is disabled by default. If you want to support Vimeo uploads, add the following to your composer file:

```javascript
"ankitpokhrel/tus-php": "dev-symfony-5 as 1.10.0",
"vimeo/laravel": "^5.5"
```

### Publish config

Once the package has been added, publish the base config

```bash
php artisan vendor:publish --tag=config
```

### Preflight configuration

Add the `HasCandy` trait to your user model.

```php
<?php

namespace App\Models;

use GetCandy\Api\Core\Traits\HasCandy;
use Illuminate\Notifications\Notifiable;
use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use HasFactory, Notifiable, HasCandy;
//...
```

### Enable GetCandy routes

Update your RouteServiceProvider.php boot method as follows (these routes assume you are using Sanctum but you can update them after installation if not).

```php
use GetCandy;
//...

public function boot()
{
    $this->configureRateLimiting();

    $this->routes(function () {
        GetCandy::router([
            'prefix' => 'api/v1',
           'template' => 'santum',
        ]);
        // ...
     });
}
```

### Additional Sanctum configuration

Laravel Sanctum requires that the API and the consumer run off the same TLD (Top Level Domain). This means that if you wanted your storefront to run on `myonlinestore.com` your API would need to be `api.myonlinestore.com`. This is also true when developing your storefront locally OR if you are using the Admin Hub.

The easiest way to do this is to run `php artisan serve --host=localhost` during development.

#### Enable credentials in your CORS

Open up `config/cors.php` and change `supports_credentials` to `true`

#### Set your session driver
Set your SESSION_DRIVER to cookie in your API env file
`SESSION_DRIVER=cookie`

#### Sanctum route prefix
Depending on how you added your GetCandy routes i.e. with the api/v1 prefix, you may need to add this to the sanctum configuration:
`'prefix' => 'api/v1/sanctum',`

### Database strict mode

Currently, for reporting to function correctly you should set `strict` to false in your `config/database.php`. This workaround will be addressed in a future release.

### Run the installer

```
php artisan candy:install
```

Lastly, if you are using Elasticsearch we need to create the initial indexes, we don't have any categories or products yet, which is fine, we just need to get it going.

```
php artisan candy:categories:reindex
php artisan candy:products:reindex
```
[block:callout]
{
  "type": "info",
  "title": "Congratulations, you've installed the GetCandy API"
}
[/block]
### Passport Authentication

GetCandy was originally built with Passport integrated into the core. [From the Laravel website](https://laravel.com/docs/8.x/passport)

>  Laravel makes API authentication a breeze using Laravel Passport, which provides a full OAuth2 server implementation for your Laravel application in a matter of minutes.

Install Passport as per their documentation and update your RouteServiceProvider.php to use the below.

```php
use GetCandy;
//...

public function boot()
{
    $this->configureRateLimiting();

    $this->routes(function () {
        GetCandy::router([
            'prefix' => 'api/v1'
        ], function ($registrar) {
            Route::group([
                'middleware' => ['auth:api', 'api']
            ], function () use ($registrar) {
                $registrar->auth();
            });
            Route::group([
                'middleware' => ['api']
            ], function () use ($registrar) {
                $registrar->guest();
            });
        });
        // ...
     });
}
```

#### Guest routes and client credentials
Previously, GetCandy used an altered version of the `Laravel\Passport\Http\Middleware\CheckClientCredentials` middleware, but since removing Passport from the core, we found this would have been too opinionated.

The issue with the original middleware was this allowed access tokens created via the `client_credentials` grant to access the API, but also meant that even if a user with an authenticated access token pinged the API, they wouldn't be bound to the request.

There seems to be some debate on this, [which you can see here](https://github.com/laravel/passport/issues/898), we currently do not have a solution that wouldn't require opinionated changes to the core GetCandy middleware, so for now we suggest reading the thread linked above and make a conscious decision based on your own needs.

If we figure out a solid solution we will post it either here or in the [forum](https://community.getcandy.io/) for discussion.

## Sanctum configuration

### Using localhost
Add localhost to your `SANCTUM_STATEFUL_DOMAINS` env value. Note we should be adding the port number.
`SANCTUM_STATEFUL_DOMAINS = 'localhost:3000,localhost,127.0.0.1,127.0.0.1:8000,::1'`

A common gotcha is the `SESSION_DOMAIN` env variable, for localhost this should be an empty string
`SESSION_DOMAIN=""`

### Using a custom domain
You will need to add the custom domain to the Sanctum configuration, remember to add the port number.
`SANCTUM_STATEFUL_DOMAINS = 'hub.example-storefront.test:3000,localhost,127.0.0.1,127.0.0.1:8000,::1'`

Update your session driver to use cookies and your session domain.
`SESSION_DOMAIN=".example-storefront.test"`

### Basic auth
You will need to implement a way of logging users into your Laravel app, a basic example of this is provided below, this can be added to your `api.php` routes file.

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;
//...
Route::post('/login', function (Request $request) {
    if (Auth::attempt([
        'email' => $request->email,
        'password' => $request->password
    ])) {
        return response()->json('', 204);
    }
    return response()->json([
        'error' => 'invalid_credentials'
    ], 403);
});

Route::post('/logout', function (Request $request) {
    Auth::logout();
    return response()->json('', 204);
});
```
