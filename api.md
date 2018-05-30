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

### Protocol

 A basic client-server communication is based on raw sockets \(tls supported\), but quickly can be extended for HTTP\(s\).

### Client Payload

Regardless of transport way, payload format MUST match the following rules.

For socket connections, first message MUST started with auth preamble:

1. First byte - auth type byte \(id of used auth\). Now we using three lowers bits.
2. Next, if  AUTH\_TYPE\_LOGIN bit present - first byte it's a login size in bytes, next - N bytes of login \(N from 1 to 255\).
3. Next, if AUTH\_TYPE\_PASSWORD bit present - first byte it's a password size in bytes, next - N bytes of password \(N from 1 to 127\).

Next messages MUST omit the auth preamble, just:

1. Payload data.
2. LF byte.

Payload data MUST be JSON RPC string with next fields:

* `method` - remote method name
* `id` - generated message identity
* `params` - method arguments

### Data structures

We used next structures for organize our client-server communication. All fields are required.

#### EventBundle

_Contain event-related data._

* _class_ - `string` - an event source class for wrap/unwrap objects
* _name_ - `string` - event name
* _payload_ - `mixed` - event payload data, must be serializable 
* _payloadType_ - `mixed` - event payload data type in source client language

#### EventDescriptor

_Contain event meta information about emitter._

* _sourceTag_ - `string` - application/script tag that raised event
* _event_ - `array` - an EventBundle packed into assoc array
* _firedAt_ - `int` - a timestamp of event dispatching in microseconds since UNIX epoch
* _delay_ - `int` - delay before event must be caught by subscribers
* _dispatcher_ - `int` - a dispatcher language identifier. \(`PHP=1`\)

#### SubjectNotifyInfo

_Contain a description for way in which subscriber will be notified._

* _destType_ - `int` - a subject notify way identifier. \(`CLI=1`, `HTTP=2`\)
* _destPath_ - `UriPair` - file path or endpoint location where is listener located
* _sourceHash_ - `string` - a way uniq identifier \(_unused_\)

#### UriPair

_Explode URI for two parts: fluent and app-based. First one is useful for detecting same application launched on multiples domains or sub-paths. Second one is constant part of each URI which available for current app._  

_Also, pair define a way in which server take a decision of choosing candidates for invoking. If server have N same subscriptions \(same tag, same event name\), there is a some problem in choosing: which one must be invoke? One or all?  To help server, pair can bring some addition information by specifying invoke kind. If pair says that need to invoke only one, server will choose ONE subscription base on next criteria: `EventDescriptor.sourceTag`, `Event.name` matched incoming one and `UriPair.appPair`_

 _In other words, if you have TWO same applications started on TWO different domains: `example.com/some`and `some.com/example`and event handler is on `index.php` and you want to guarantee that only one handler receive same event, you have to separate URI like this structure._

* _fluentPart_- `string` -   domain-dependent part, i.e. app root. \(`example.com/some/` or `some.com/example/` depend on where is subscription made\)
* _appPart_ - `string` - app-dependent part. \(`index.php`\)
* _invokeKind_ - `int` - invoking strategy. There is two constants:
  * `UriPair.INVOKE_KIND_ONCE = 1` - only one subscriber will be notified
  * `UriPair.INVOKE_KIND_EACH = 2` - each subscriber will be notified

### Server JSON RPC Methods

{% api-method method="post" host="" path="" %}
{% api-method-summary %}
Event.Register
{% endapi-method-summary %}

{% api-method-description %}
Register subscriber on server.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-body-parameters %}
{% api-method-parameter name="subjects" type="array" required=true %}
`SubjectNotifyInfo[]`
{% endapi-method-parameter %}

{% api-method-parameter name="eventMask" type="string" required=true %}
`Event.name` mask for wich subscriber will listening
{% endapi-method-parameter %}

{% api-method-parameter name="sourceTag" type="string" required=true %}
application source tag
{% endapi-method-parameter %}
{% endapi-method-body-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```
{"id": <source_id>, "error": null, "reason": null}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

{% api-method method="post" host="" path="" %}
{% api-method-summary %}
Event.Fire
{% endapi-method-summary %}

{% api-method-description %}
Emit event.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="event" type="object" required=true %}
`EventDescriptor` to emit.
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```
{"id": <source_id>, "error": null, "reason": null}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

