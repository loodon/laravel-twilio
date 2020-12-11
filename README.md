laravel-twilio
===============
Laravel Twilio API Integration

[![Build Status](https://img.shields.io/travis/okay/laravel-twilio.svg?style=flat-square)](https://travis-ci.org/okay/laravel-twilio)
[![Total Downloads](https://img.shields.io/packagist/dt/okay/twilio.svg?style=flat-square)](https://packagist.org/packages/okay/twilio)
[![Latest Stable Version](https://img.shields.io/packagist/v/okay/twilio.svg?style=flat-square)](https://packagist.org/packages/okay/twilio)
[![License](https://img.shields.io/github/license/okay/laravel-twilio?style=flat-square)](#license)

## Installation

Begin by installing this package through Composer. Run this command from the Terminal:

```bash
composer require okay/twilio
```

This will register two new artisan commands for you:

- `twilio:sms`
- `twilio:call`

And make these objects resolvable from the IoC container:

- `Okay\Twilio\Manager` (aliased as `twilio`)
- `Okay\Twilio\TwilioInterface` (resolves a `Twilio` object, the default connection object created by the `Manager`).

There's a Facade class available for you, if you like. In your `app.php` config file add the following
line to the `aliases` array if you want to use a short class name:

```php
'Twilio' => 'Okay\Twilio\Support\Laravel\Facade',
```

You can publish the default config file to `config/twilio.php` with the artisan command

```shell
php artisan vendor:publish --tag=config --provider=Okay\Twilio\Support\Laravel\ServiceProvider
```

#### Facade

The facade has the exact same methods as the `Okay\Twilio\TwilioInterface`. First, include the `Facade` class at the top of your file:

```php
use Twilio;
```

To send a message using the default entry from your `twilio` [config file](src/config/config.php):

```php
Twilio::message($user->phone, $message);
```

One extra feature is that you can define which settings (and which sender phone number) to use:

```php
Twilio::from('call_center')->message($user->phone, $message);
Twilio::from('board_room')->message($boss->phone, 'Hi there boss!');
```

Define multiple entries in your `twilio` [config file](src/config/config.php) to make use of this feature.

### Usage

Creating a Twilio object. This object implements the `Okay\Twilio\TwilioInterface`.

```php
$twilio = new Okay\Twilio\Twilio($accountId, $token, $fromNumber);
```

Sending a text message:

```php
$twilio->message('+18085551212', 'Pink Elephants and Happy Rainbows');
```

Creating a call:

```php
$twilio->call('+18085551212', 'http://foo.com/call.xml');
```

Generating a call and building the message in one go:

```php
$twilio->call('+18085551212', function (\Twilio\TwiML\VoiceResponse $message) {
    $message->say('Hello');
    $message->play('https://api.twilio.com/cowbell.mp3', ['loop' => 5]);
});
```

or to make a call with _any_ Twiml description you can pass along any Twiml object:

```php
$message = new \Twilio\TwiML\VoiceResponse();
$message->say('Hello');
$message->play('https://api.twilio.com/cowbell.mp3', ['loop' => 5]);

$twilio->call('+18085551212', $message);
```

Access the configured `Twilio\Rest\Client` object:

```php
$sdk = $twilio->getTwilio();
```

You can also access this via the Facade as well:

```php
$sdk = Twilio::getTwilio();
```

##### Pass as many optional parameters as you want

If you want to pass on extra optional parameters to the `messages->sendMessage(...)` method [from the Twilio SDK](https://www.twilio.com/docs/api/messaging/send-messages), you can do so
by adding to the `message` method. All arguments are passed on, and the `from` field is prepended from configuration.

```php
$twilio->message($to, $message, $mediaUrls, $params);
// passes all these params on.
```

The same is true for the [call method](https://www.twilio.com/docs/api/voice/call#post-parameters).

```php
$twilio->call($to, $message, $params);
// passes all these params on.
```

#### Dummy class

There is a dummy implementation of the `TwilioInterface` available: `Okay\Twilio\Dummy`. This class
allows you to inject this instead of a working implementation in case you need to run quick integration tests.

#### Logging decorator

There is one more class available for you: the `Okay\Twilio\LoggingDecorator`. This class wraps any
`TwilioInterface` object and logs whatever Twilio will do for you. It also takes a `Psr\Log\LoggerInterface` object
(like Monolog) for logging, you know.

By default the service providers don't wrap objects with the `LoggingDecorator`,
but it is at your disposal in case you want it. A possible use case is to construct a
`TwilioInterface` object that logs what will happen, but doesn't actually call Twilio (using the Dummy class):

```php
if (getenv('APP_ENV') === 'production') {
    $twilio = $container->make(\Okay\Twilio\Manager::class);
} else {
    $psrLogger = $container->make(\Psr\Log\LoggerInterface::class);
    $twilio = new LoggingDecorator($psrLogger, new \Okay\Twilio\Dummy());
}

// Inject it wherever you want.
$notifier = new Notifier($twilio);
```

## Credits

- [Hannes Van De Vreken](https://twitter.com/hannesvdvreken)
- [Travis Ryan](https://twitter.com/nayrsivart)
- [All Contributors](../../contributors)

### License

laravel-twilio is open-sourced software licensed under the [MIT license](http://opensource.org/licenses/MIT)
