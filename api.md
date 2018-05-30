# IguanAPI

Our server API is build on JsonRPC 1.0. It's little overhead, but very easy to use.

Here is overview, description of methods and data structures in communication.

## Event

 Event is a some action in any part of system. 

An event has two principal parts: name and payload.

### Name

Name may consist of different pats. Each part can represent own domain or level. Composition of event name is defined by system architect. Typical event name is like `domain.entity.action` or `auth.user.password_change`, or `app.log.error`.

Token can be separated on domains by dot \(`.`\) only.

### Payload

Payload is piece of data wich must be caught by consumers. It can contain ID of DB entity which was changed or entire entity. Structure of payload is defined by emitter. You can automatically wrap/unwrap payload into own object of some class.

### Emitter

 A part of system that produce events.

### Subscriber 

A part of system that consume events _\(also known like listener or consumer, both the same\)_.

Subscribers can listen for multiple events by event name mask. Listeners will be notified in case of:

1. Full matching with excepted name and incoming event token.
2. Partial matching with wildcards \(`*`\). Wildcard are can be any of one event name domain. For example, a name like `entity.*` means, that all subscriber for `entity.<any_word>` and even`entity.<any_word>.action` will receive event.
3. Partial matching with sharp \(`#`\). Unlike wildcard, sharp will replace all remains name domain on right side. For example, a name like `entity.#` means, that all subscriber for `entity.attribute.action` or `entity.attribute` will receive event.

## Auth

Before client can interact with server, client must provide auth info. 

Server can restrict some client action using own RBAC system.

{% hint style="danger" %}
RBAC is not implemented yet.
{% endhint %}

There is 4 types of auth:

1. **No auth** _\(id=1\)_ - allows public access
2. **Login auth** _\(id=2\)_ - check login part only for authorization, login can contain some secret token
3. **Password auth** _\(id=4\)_ - check password part only for authorization, like login, can contain some secret token
4. **Login+password auth** _\(id=6\)_ - check login + password combination

ID is counting as a bitwise `or` operation with presented auth component types:

* `AUTH_TYPE_NO_AUTH = 0x1`
* `AUTH_TYPE_LOGIN = 0x2`
* `AUTH_TYPE_PASSWORD = 0x4`

## Signing

Each event by server can be signed using RSA SHA256 with server private key. Each client that have public key can verify incoming event that it was from trusted source.

We categorically against for secret token that shared between server and clients, because any subscriber is free to pass same token to another subscriber with broken or malware data.

### Web Hook

Each web hook event content are signed. For signing we use concatenation of `Iguan-Dest-Host` header and whole request body. Sign is on `Iguan-Sign` header. 

## Interaction

{% api-method method="get" host="" path="" %}
{% api-method-summary %}

{% endapi-method-summary %}

{% api-method-description %}

{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="" type="string" required=false %}

{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```

```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

