# Quick Start

## Setup Server

{% hint style="danger" %}
Under development
{% endhint %}

## Setup client

{% hint style="warning" %}
_Currently, we have only one client for PHP. But, if you want, we glad to get some help for growing._
{% endhint %}

### PHP

{% hint style="info" %}
[_https://github.com/IguanREQ/Iguan-client-php_](https://github.com/IguanREQ/Iguan-client-php)
{% endhint %}

Create and navigate to new folder.

```bash
$ mkdir IguanTest && cd IguanTest
```

Initialize project via [composer](https://getcomposer.org/).

```text
$ composer init
```

Add Iguan client lib to project.

```bash
  $ composer require iguan-req/iguan-client-php
```

## Emit!

Emit your first event and see what happens.

### PHP

Create Iguan config file.

```text
common:  tag: 'First Event App'  remote:    client:      socket:        host: <IguanServerIp>        port: 8081
```

Create event handler

```text
<?phpuse Iguan\Common\Data\EncodeDecodeException;use Iguan\Event\Builder\Builder;use Iguan\Event\Builder\Config;use Iguan\Event\Common\CommunicateException;use Iguan\Event\Common\EventDescriptor;use Iguan\Event\Event;use Iguan\Event\Subscriber\Subject;use Iguan\Event\Subscriber\SubjectHttpNotifyWay;use Iguan\Event\Subscriber\UriPair;use Iguan\Event\Subscriber\Verificator\InvalidVerificationException;require_once dirname(__DIR__, 2) . '/vendor/autoload.php';//load config from file$config = Config::fromFile(__DIR__ . '/' . 'config.yml');$builder = new Builder($config);//build emitter with config values$emitter = $builder->buildEmitter();try {    //build subscriber with config values.    //because subscriber can revoke previous subscriptions, there is may be a server    //communication issues    $subscriber = $builder->buildSubscriber();    //create HTTP-subscription (WebHook) subject for some man creating event    //assuming we have localhost web root in "src" folder    $subject = new Subject('man.create', new SubjectHttpNotifyWay(new UriPair('http://localhost/', 'event.php')));    //add some handler for this subject    $subject->addHandler(function (EventDescriptor $descriptor) {        //a $man now is source array with event data inside        $man = $descriptor->raisedEvent->getPayload();        //just store each new person in separated files        file_put_contents('/tmp/event_man_' . $man['id'], json_encode($man));    });    //after adding handlers we must subscribe subject for registering in    //backend event server and for being ready for receiving incoming events    //handlers will be notified right here    $subscriber->subscribe($subject);    //fire event when some person created with person data inside    $emitter->dispatch(Event::create('man.create', ['name' => 'John', 'age' => 28, 'id' => 1199]));} catch (CommunicateException $e) {    die('Some server communication error: ' . $e->getMessage());} catch (EncodeDecodeException $e) {    die('Some data decoding error: ' . $e->getMessage());} catch (InvalidVerificationException $impossible) {    //verifying disabled}
```

Run  local PHP server

```bash
$ php -S localhost:8000
```

Navigate to [http://localhost:8000/src/event.php](http://localhost:8000/src/event.php). Now, you can check `tmp`location and look inside generated files.

{% hint style="info" %}
More examples in [repository](https://github.com/IguanREQ/Iguan-client-php/tree/master/examples).
{% endhint %}



