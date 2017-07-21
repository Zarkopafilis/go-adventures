### gorilla/csrf Explained
WIP

## Why ?
I saw a guy who posted the same question 2 days in a row, on the Gophers Slack #golang-newbies channel and decided to jump in and help

## Overview
[gorilla/csrf](https://github.com/gorilla/csrf) does quite a lot of stuff to protect your application from csrf attacks,
but the whole source is about 5 go source files (and 5 test ones), so it wasn't that hard to dive into the source code and inspect the project further.

Official docs present us with 2 use cases: **Serverside HTML Render** and **JavaScript Frontend**.

# Serverside HTML Render
In this case, just embed the provided csrf token into the form as a hidden field.
That token is then sent back to the server where validation takes place.

# JavaScript Frontend
This is a bit different case. You can't inject the token into the form as a hidden field, but you can render it as a meta tag,
which you can then retrieve with js and send back it's contents as a request header.

How does this server-side validation take place? While exchanging requests with the server, every time a secure session cookie 
(specifically [gorilla/securecookie](https://github.com/gorilla/securecookie)) gets sent through the wire. This cookie is of course,
encrypted and only the server can decrypt. This piece of data contains the masked csrf token that can be unmasked from the client.
It is easily possible to override this and just have a regular session cookie without embedded information, while storing the tokens
on some key/value store (redis perhaps?) on the server side. To do that, you'd have to fork gorilla/csrf and provide a kv store-backed
implementation of the following interface at [store.go](https://github.com/gorilla/csrf/blob/master/store.go):

```golang
// store represents the session storage used for CSRF tokens.
type store interface {
	// Get returns the real CSRF token from the store.
	Get(*http.Request) ([]byte, error)
	// Save stores the real CSRF token in the store and writes a
	// cookie to the http.ResponseWriter.
	// For non-cookie stores, the cookie should contain a unique (256 bit) ID
	// or key that references the token in the backend store.
	// csrf.GenerateRandomBytes is a helper function for generating secure IDs.
	Save(token []byte, w http.ResponseWriter) error
}
```

## Digging deeper
