# gorilla/csrf Explained

## Why ?
I saw a guy who posted the same question 2 days in a row, on the Gophers Slack #golang-newbies channel and decided to jump in and help!

## Overview
[gorilla/csrf](https://github.com/gorilla/csrf) does quite a lot of stuff to protect your application from csrf attacks, but the whole source is about 5 go source files (and 5 test ones), so it wasn't that hard to dive into the source code and inspect the project further.

Official docs present us with 2 use cases: **Serverside HTML Render** and **JavaScript Frontend**.

# Serverside HTML Render
In this case, just embed the provided csrf token into the form as a hidden field. That token is then sent back to the server where validation takes place. This is something that is done automatically. Here is the corresponding code snippet: 
```golang
// TemplateField is a template helper for html/template that provides an <input> field
// populated with a CSRF token.
//
// Example:
//
//      // The following tag in our form.tmpl template:
//      {{ .csrfField }}
//
//      // ... becomes:
//      <input type="hidden" name="gorilla.csrf.Token" value="<token>">
//
func TemplateField(r *http.Request) template.HTML {
	if name, err := contextGet(r, formKey); err == nil {
		fragment := fmt.Sprintf(`<input type="hidden" name="%s" value="%s">`,
			name, Token(r))

		return template.HTML(fragment)
	}

	return template.HTML("")
}
```

# JavaScript Frontend
This is a bit different case. You can't inject the token into the form as a hidden field, but you can render it as a meta tag, which you can then retrieve with js and send back it's contents as a request header.

Sometimes you just can't serve the fronted from your backend code. Thus, you are not able to render anything on the html via templates. A different approach would be to send a __X-CSRF-Token__ header from the server to the client, which is then going to be passed back as the same header, on the __subsequent__ post! 
Remember: 
> This library generates unique-per-request (masked) tokens as a mitigation against the [BREACH](http://breachattack.com/) attack.

* You shouldn't really care about BREACH if you use https and also send HSTS headers (which, ehm, you should - even with self-signed certs).

How does this server-side validation take place? 

While exchanging requests with the server, every time a secure session cookie (specifically [gorilla/securecookie](https://github.com/gorilla/securecookie)) gets sent through the wire. This cookie is of course, encrypted and only the server can decrypt. This piece of data contains the masked csrf token that can be unmasked from the client. It is easily possible to override this and just have a regular session cookie without embedded information, while storing the tokens on some key/value store (redis perhaps?) on the server side. To do that, you'd have to fork gorilla/csrf and provide a kv store-backed implementation of the following interface at [store.go](https://github.com/gorilla/csrf/blob/master/store.go):

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
... but that's something out of scope for this blog post ...

## Digging deeper
All the actual middleware work is done at [csrf.go](https://github.com/gorilla/csrf/blob/master/csrf.go), lines 126 ~ end.

I will now proceed to outlining what happens upon each request:

First of all, if any request encounters any error while going through this middleware, an error handler is engaged. By default, 403 Forbidden is returned. You can change this and many other things if you provide these so-called __options__ :

```golang
type options struct {
	//...
}
```

The Protect function:

```golang
func Protect(authKey []byte, opts ...Option) func(http.Handler) http.Handler {
	//...
}
```

usually receives a mux.Router as a parameter. It sets up defaults and/or overrides them if options are passed. It also sets up the secure cookie instance:

```golang
// Create an authenticated securecookie instance.
if cs.sc == nil {
	cs.sc = securecookie.New(authKey, nil)
	// Use JSON serialization (faster than one-off gob encoding)
	cs.sc.SetSerializer(securecookie.JSONEncoder{})
	// Set the MaxAge of the underlying securecookie.
	cs.sc.MaxAge(cs.opts.MaxAge)
}
```
...which we are probably going to discuss in a different blog post.

Now, upon each request, an attempt is being made to extract the token from the session cookie. If the token is absent (or the cookie is), a new one will be generated, but the current request is going to return 403 Forbidden. Supposing the operation was completed successfully, this extracted token is the __real__ csrf token. The next step would be to check the headers for the __X-CSRF-Token__. That token is masked by XORing a one-time-pad and the base csrf toke. It should be unmasked and then compared to our previously extracted csrf token. (<-- That one time pad protects from Breach as well, because in order for someone to perform a breach attack, he would need to send quite a lot of requests) 

There's other magic going on towards the bottom of the function:
```golang
	// Set the Vary: Cookie header to protect clients from caching the response.
	w.Header().Add("Vary", "Cookie")
	
	//and
	// Clear the request context after the handler has completed.
	contextClear(r)
```

Especially, this contextClear on the end is why not a lot of new variables are declared. I believe this is a micro-optimization in order to avoid extra memory allocations; this middleware is supposed to run upon each request... it is a requirement to be efficient!

Lastly, this middleware consumes the request body in order to defend against the attack. You need to store it in the request context with another layer of middleware if you intend to use it in your own handler. You can still read the form data in the following handlers.

Hopefully you now understand a bigger fraction of the what's-going-on-under-the-hood part of your go web services :) .
