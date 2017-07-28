# Why some things can't be part of a library
I'll explain it by using [gorilla/handlers](https://github.com/gorilla/handlers) as an example, but first

## Why
Everything happens for a reason, and this post due to slack discussions __again__. (Yeah, join **gophers.slack.com** already!)

## cors.go
Gorilla Handlers provide some middleware to configure Cross Origin Resource Sharing. It's very handy and easy to use. If you take a look into the source, you'll see that you can either configure specific domains, have a plain wildcard ('*') in order to allow any origin, or specify a validator function that returns a bool based on wether the provided url fulfils whatever conditions you programmed.

You might think that a plain wildcard would also mean that there could be some regex support, but it's not! The foundation is already laid down for this, with the validator function. Assuming the type is
```go
type OriginValidator func(string) bool
```
...the regex matching function should look like this:
```go
func AllowedOriginRegexValidator(rgx string) OriginValidator {
  r := regexp.MustCompile(rgx)
  return func(url string) bool {
    return r.MatchString(url)
  }
}
```
> We could also use regexp.Compile and handle the panic as an error

Assume this gets integrated into the library. People are going to start using it. Untested regex patterns will pass the **go build** phase and maybe the **go test** phase as well, just because 

1. Regex gets compiled at runtime
2. You can't unit test the actual regex easily, without using some global constant, or having some code duplication, because it's wrapped around the library's code.

Libraries shouldn't really panic that way, on runtime, just because the user provided wrong data/functions; libraries should return errors that their users can handle (or ignore and have their program crash __:)__ ). Libraries shouldn't log stuff either. Panics should happen only due to programmer errors, if that programmer is the creator of the library. In this specific case, I could recover() from the panic and return an error, but there is no room to do that due to the design of the library; errors are not supposed to happen (and really, they don't - you just pass string contstants as a slice...) on that stage.

OK, one could claim that it's not that bad of a choice because it would just fail to start. Well, not everyone has a staging environment and many people just treat CI builds that pass tests as good to go. Nobody wants stuff breaking in production.

## On the other side
Moving the testing code (which could be regex, checking prefixes or whatever) from the library to your application's code gives you more responsibility regarding testing these parts or, making sure that wrongly crafted validators fail at compile time.

### Conclusion
This may not be the best example because allowed Origins is not something that you change that frequently (if ever, after your code makes it into production), but you get the point...

Due to how trivial this example is, I created the PR including the [Add AllowedOriginRegexValidator PR](https://github.com/gorilla/handlers/pull/109) and I'm currently awaiting for feedback, from the community or the gorilla team.
