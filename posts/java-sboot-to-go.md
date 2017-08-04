### Java and Spring Boot to __just__ Go

## Why ?
Well, not exactly slack... It's something I wanted to write down for quite some time and I can also correlate many points that this article outlines, with some of my answers and comments on random chats on slack.

## Let's get started
Even if you're not familiar with Java, you should be able to get something out of this, even if you have a different language background (e.g. PHP, C#).

I'll first outline what made it hard (or sometimes, easier), to learn to write web apps with golang. Java, wether your framework is compliant with the enterprise standard (EE), or prefers to drift away a bit (like Spring), in order to make some stuff easier - like less boilerplate code - is a feature packed monster. Lot's of stuff happen under the hood. This may be confusing to new developers but as soon as you perform a good dive in with a test project, all these features make you productive and very, very fast to implement new features. I'll outline some of these very useful, usually plug & play features right now :

- ORM (Including libraries like retrofit)
- Templating Engines
- REST-Exposed repositories
- Running profiles (ex. prod-mysql, prod-mongo, testing, staging)
- Global & Easy exception handling
- HATEOAS (Integrates super easy with templating engines)
- Form Validation and object mapping
- Migrations and Schema Generation
- Easy Cloud Native apps (Distributed tracing, Discovery, Configuration Servers, etc)


> Remember : Due to the structure and the metaprogramming involved in the architecture of the applications, there is no need to change almost none of your code; for example, turning on csrf protection on par with a server-side templating engine, just works out of the box because components are detected automatically, no middleware chain additions are needed. Such middleware chain(s) are not something you actually write in Java - components are multiplexed together automatically based on priorities, demands and dependencies.

I won't say something about speed and memory requirements here; I'll just write down that I was super pissed off when my new rapsberry pi took 80 seconds to bootstrap a spring-boot app. Memory requirements are also higher than go and perhaps, speed (unless you're performing batch operations? I'm not sure though).

Enough with Java. After taking the tour of the go language and getting used to the conventions, I was still under the effects of tunnel vision due to these feature packed frameworks that Java has given me; I was expecting some similar 'package-thing' being advertised in many places (like Spring-Boot is), but I was wrong. Go's simplicity, functions as first class citizens and concurrency made it feel different.

- No single framework/package to do everything. There are some stuff like `go-kit` and others but they are not exactly idiomatic, nor take in perspective that go is not an Object Oriented Language. Some are influenced by other languages like ruby or python, which leads to many nasty workarounds.

- The community doesn't like under-the-hood proccesses and I am super happy about that.

- Everybody typically starts by making a hello rest world example by using the built-in http server, which is super good and fast. Then, pick your components and mix & match them. No automatic detection of dependencies etc. Pick your router, pick your middleware (maybe a middleware chain-helper) and deploy your app. 

- Write your own repository code. You can use some sort of library to get an implementation from an `interface`, but not everybody does that. Elegant error handling makes writing custom sql statements easy. You could take it one step further and get rid of the boilerplate code (`sqlx` I think is the name) - but thats it.

- Same thing for the mocks. Write it yourself and stay safe(r) or generate them with some fluent API.


