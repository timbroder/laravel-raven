Laravel Raven
=============

> Sentry (Raven) error monitoring for Laravel 4 & 5 with send in background using queues

[![Build Status](http://img.shields.io/travis/clowdy/laravel-raven/master.svg?style=flat-square)](https://travis-ci.org/clowdy/laravel-raven)
[![Scrutinizer Code Quality](http://img.shields.io/scrutinizer/g/clowdy/laravel-raven/master.svg?style=flat-square)](https://scrutinizer-ci.com/g/clowdy/laravel-raven/)
[![Coverage Status](https://img.shields.io/scrutinizer/coverage/g/clowdy/laravel-raven/master.svg?style=flat-square)](https://scrutinizer-ci.com/g/clowdy/laravel-raven/code-structure/master)
[![License](http://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](http://www.opensource.org/licenses/MIT)
[![Latest Version](http://img.shields.io/packagist/v/clowdy/laravel-raven.svg?style=flat-square)](https://packagist.org/packages/clowdy/laravel-raven)
[![Total Downloads](https://img.shields.io/packagist/dt/clowdy/laravel-raven.svg?style=flat-square)](https://packagist.org/packages/clowdy/laravel-raven)

Sentry (Raven) error monitoring for Laravel 5 with send in background using queues. This will add a listener to Laravel's existing log system. It makes use to Laravel's queues to push messages into the background without slowing down the application.

![rollbar](https://www.getsentry.com/_static/getsentry/images/hero.png)

## Installation

Add the package to your `composer.json` and run `composer update`.

```js
{
    "require": {
        "clowdy/laravel-raven": "2.1.*"
    }
}
```

Add the service provider in `config/app.php`:

```php
Clowdy\Raven\RavenServiceProvider::class,
```

Register the Raven alias:

```php
'Raven' => Clowdy\Raven\Facades\Raven::class,
```

### Looking for a Laravel 4 compatible version?

Checkout the [1.0 branch](https://github.com/clowdy/laravel-raven/tree/1.0)

## Configuration

Publish the included configuration file:

```bash
$ php artisan vendor:publish
```

Change the Sentry DSN by using the `RAVEN_DSN` env variable or changing the config file:

```php
RAVEN_DSN=your-raven-dsn
```

This library uses the queue system, make sure your `config/queue.php` file is configured correctly. You can also specify the connection and the queue to use in the raven config. The connection and queue must exist in `config/queue.php`. These can be set using the `RAVEN_QUEUE_CONNECTION` for connection and `RAVEN_QUEUE_NAME` for the queue.

```php
RAVEN_QUEUE_CONNECTION=redis
RAVEN_QUEUE_NAME=error
```

**The job data can be quite large, ensure you are using a queue that can support large data sets like `redis` or `sqs`**.
If the job fails to add into the queue, it will be sent directly to sentry, slowing down the request, so its not lost.

## Usage

To monitor exceptions, simply use the `Log` facade or helper. This can be done in ```app/Exceptions/Handler.php```:

```php
public function report(Exception $e)
{
    Log::error($e);
    
    // or

    logger()->error($exception);
    
    return parent::report($e);
}
```

You can change the logs used by changing the log level in the config by modifying the env var.

```php	
RAVEN_LEVEL=error
```

### Event Id

There is a method to get the **latest** event id so it can be used on things like the error page.

```php
Raven::getLastEventId();
```

### Context information

You can pass additional information as context like this:

```php
Log::error('Oops, Something went wrong', [
    'user' => ['name' => $user->name, 'email' => $user->email]
]);
```

User data can alternatively be added to the context of any messages sent via a processor, such as the included `UserDataProcessor`; see below.

#### Included optional processors

##### `UserDataProcessor`

This processor attaches data about the current user to the error report.

```php
'processors' => [
    Clowdy\Raven\Processors\UserDataProcessor::class,
],
```

Or, to configure the `UserDataProcessor` object:

```php
'processors' => [
    new Clowdy\Raven\Processors\UserDataProcessor([
        'only' => ['id', 'username', 'email'], // This is ['id'] by default; pass [] or null to include all fields
        'appends' => ['normallyHiddenProperty'],
    ]),
],
```

See the `UserDataProcessor` source for full details.

##### `LocaleProcessor`

This processor attaches to the error report the current application locale as a tag.

```php
'processors' => [
    Clowdy\Raven\Processors\LocaleProcessor::class,
],
```

## Credits

This package was inspired [rcrowe/Raven](https://github.com/rcrowe/Raven).
