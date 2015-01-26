# http-server
http-server is a simple http-server in golang. It tries to be more efficient and
disk friendly by reading documents into memory and serving them from there. It will
also search for a file on the disk if it doesn't exist in the cache, and caches it if
it has appeared. It doesn't return 404 unless the file really _really_ doesn't exist.

# Implementation
It's rather simple, really.

First we need an appropriate data structure. I chose golang's map type since it's well
suited for 'looking things up' by name. We wrap this map in a simple struct like so:
```
type DocCache struct {
        docs  map[string]string //map of documents in 'path', indexed by filename
        size  int64             //Size of the document in bytes
        count int               //Amount of documents in the cache
        path  string            //Path to the document directory
}
```

First we have the ```docs``` variable which is the map we talked about. This is a simple
name->data map. It's indexed by filename for convienence and stores a string representation of
the document we can write to a ```http.request``` writer. 

When we want to fill this cache at startup, we step through the document directory
```path``` and enumerate each of the documents, storing them in this cache.

The ```size``` and ```count``` variables are used mainly for logging, they don't really affect much else.
However, these could be used to limit the size and count of files read into the cache, but that's beyond the scope
of what I was trying to accomplish in this project

# Automatic Document Refreshing
Another great feature would be to watch the document directory for changes made to the documents themselves, and
refresh the cache. This could be done with inotify (which golang already has a great package for) and a little bit
of tinkering

This would really make this little server great, since you could use it to quickly and efficiently prototype
webpages without the need to reload the server every time you make a change. I Plan on implementing this, so stay tuned.

# Handling Requests
Another dead simple solution.
```
func RootHandle(res http.ResponseWriter, req *http.Request) {
	var reply *template.Template

	log.Println("<< GET / -", req.UserAgent())
	if req.URL.Path[1:] == "" {
		reply = cache.Docs.GetDoc("index.html")
	} else {
		reply = cache.Docs.GetDoc(req.URL.Path[1:])
	}

	data := &PageData{"http-server"}
	reply.Execute(res, data)
}
```
This simple handler can be used as a 'catch-all' route. First we check if there's even a document name in the url, if there
isn't, we simply return the ```index.html`` document. If there does happen to be a document name, we get it from the cache (or disk)
and return it instead.

html/document allows us to actually pass data to the documents for rendering, so we use it instead of a raw html file.
This is great, but not complete. Each page may need different data, and trying to figure out what data goes with what document requires a little more work which should (and will) be in its own package.

The way I see it is we can create "data getters" and register them with each document, instead of hardcoding them in the actual route handlers. This will keep things organized and will allow us to get different kinds of data in different ways (e.g redis, mongodb, sql or the server itself) making the server a little more adaptable. 

## Static Files
No website is complete without a good stylesheet, maybe some images and javascript for good measure.
This is super easy:
```
func StaticHandle(res http.ResponseWriter, req *http.Request) {
	log.Println("<< GET /static -", req.UserAgent())
	http.ServeFile(res, req, req.URL.Path[1:])
}
```
See? All we have to do is use ```http.ServeFile()``` to return the contents of the requested file or directory.
Short and sweet.

# Conclusion
This was actually a spur of the moment idea, and something I've toyed with before but never really felt I had accomplished
anything with any other language or framework. In Golang this was very easy and simple, even with my limited time in go I was
able to slap this together in just a few hours. I'm very excited about this, I definitely look forward to seeing what I can come up
with in webapp land using Go.


# Contact
Pull request? Questions? Criticism? You can hit me up on twitter [@tywkeene](https://twitter.com/tywkeene) or over email <tyrell.wkeene@gmail.com>

All feedback is greatly appreciated :)
