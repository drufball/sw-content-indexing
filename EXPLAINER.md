## Proposals
This is a new area of exploration so all proposals are correspondingly rough.

### Manifest URL
The manifest format could be extended to include an `offline_url` attribute:

```json
"offline_url": "/getcontent"
```

The service worker would intercept fetch requests to `/getcontent`, inspect its cache, and respond with a JSON array specifying available content. The JSON array would contain an object for each URL with offline content, along with a `sources` key specifying metadata about content viewable at that URL.

For example, suppose a user has saved two videos to view offline at [bbc.co.uk](bbc.co.uk). BBC has also downloaded a few breaking stories automatically. BBC has also cached the app shells necessary to view videos and articles. BBC's service worker would respond to a fetch request for `/getcontent` with the following JSON array:

```javascript
[
    {
 		    url: "bbc.co.uk/videos",
 		    content-type: "video",
 		    sources: ["presidential-interviews.mp4", "g8-summit.mp4"]	
 	  },
 	  {
 		    url: "bbc.co.uk/articles",
 		    content-type: "text",
 		    sources: ["supertuesday-results.txt", "heatwave-in-prague.txt"]
 	  }
]
```

Instead of strings, the `sources` array could return JSON objects specifying human readable metadata like title, preview thumbnails, explainer text, etc.

A browser (or any site collaborating via foreign fetch) would poll the `offline_url` when constructing its offline content viewer.

__Pros:__
- No API additions required since the manifest is extensible
- Works with foreign fetch
- Robust against content deletion or changes
- Code simplicity: developer only has to maintain one method and does not have to explicitly track resources.

__Cons:__
- Browsers that want to track offline content for many sites would have to poll several service workers
- Browser would not be aware of the moment when the content was added

### Service worker event
Instead of having an `offline_url` in the manifest, we could add a `getcontent` event to service workers:

```javascript
this.addEventListener('getcontent', function(event) {
    event.respondWith(
		    // inspect cache and return available content
 	);
});
```

The response to this method would be the same JSON array in the manifest proposal.

__Pros:__
- `getcontent` is an explicit event rather than a special case of the `fetch` event
- Robust against content deletion or changes
- Code simplicity: developer only has to maintain one method and does not have to explicitly track resources.

__Cons:__
- Doesn’t work with foreign fetch
- Requires browsers to poll multiple service workers
- Browser would not be aware of the moment when the content was added

### Navigator method
Another approach is to explicitly register/unregister cached content with the browser:

```javascript
navigator.pageIndexing.add({
 	  url: "sw-contents.example.com/video", 
 	  offline: true, 
 	  meta: { 
 	  	title: "example", 
   		description: "...", 
 		type: "video",
        expiration: new Date().getTime() + expirationTime
 	  }
});
```

`navigator.pageIndexing.add()` could be called whenever content is successfully cached to register it. The method would return a key which could be used to unregister content when it was no longer available or relevant:

```javascript
navigator.pageIndexing.remove(key);
```

__Pros:__
- Explicit registration eliminates need for browser poll
- Browser knows when offline content is added and can take appropriate actions like granting [persistent storage](https://storage.spec.whatwg.org/#persistence), etc.

__Cons:__
- Code complexity: Developer has to track keys, expirations, etc. and explicitly add/remove content at multiple points in the code base.
- Browser needs to do extra work to clean up registrations when it clears content
- Doesn’t work with foreign fetch
