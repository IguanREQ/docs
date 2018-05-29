# Lightweight Web and CLI hooks

## What?

Iguan it is a light, high-performance, secure and flexible way to organize Event Driven Architecture in small or medium IT-projects with minimum intervention to current infrastructure.

## How?

Unlike others MQ \(RabbitMQ, etc\), Iguan operate with Events. Event is fired by clients, consumed by event server. Server lists all their subscriptions, i.e. hooks, and define a way in which subscriber will be invoked.

Currently, supported two ways for hooks: Web and CLI. 

Web hook means that some consumer will be invoked by HTTP on some endpoint. Signed payload will be in a request body. 

CLI means that some script will be ran by shell on node where subscription was made. Signed payload will be passed as a shell argument.

## Ok, but...

#### where is light?

* No heavy interaction between clients and server
* No persistent client-listening
* Minimum custom algorithms in API 

#### where is high-performance?

* Highly-optimized client and server code

#### where is secure?

* RBAC pushing model
* Each event can be verified for source-identical by consumer by checking sing

#### where is flexible?

* Almost each class in our client can be replaced by own implementation

## Wow!

[Let's try!](quick-start.md)

