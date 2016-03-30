## Proposals
This is a new area of exploration so all proposals are correspondingly rough.

### Manifest URL
The manifest format could be extended to include an `offline_url` attribute:

```json
"offline_url": "/enumerate-cache"
```

The service worker would intercept requests to this URL, inspect its cache, and respond with a JSON object specifying its contents, along with the URL necessary to view those contents:

```javascript
[
    {
 		    url: "sw-contents.example.com/videos",
 		    content-type: "video",
 		    sources: ["cached-video-1.mp4", "cached-video-2.mp4"]	
 	  },
 	  {
 		    url: "sw-contents.example.com/articles",
 		    content-type: "text",
 		    sources: ["cached-article-1.html", "cached-article-2.html"]
 	  }
]
```

Instead of returning strings, the sources array could return JSON objects specifying additional meta data like preview thumbnails, explainer text, etc. 

A browser (or any site collaborating via foreign fetch) would poll the `offline_url` when constructing its offline content viewer.

__Pros:__
- Minimal API surface area additions to manifest or SW
- Works with foreign fetch
- Robust against content deletion or changes
- Code simplicity: developer only has to maintain one method and does not have to explicitly track resources.

__Cons:__
- Browsers that want to track offline content for many sites would have to poll several service workers

### Service worker event
Instead of having an `offline_url` in the manifest, we could add a `getContent` event to service workers:

```javascript
this.addEventListener('getcontent', function(event) {
    event.respondWith(
		    // inspect cache and return available content
 	  );
});
```

The response to this method could be the same JSON array specified above.

__Pros:__
- `getcontent` is an explicit event rather than a special case of the `fetch` event
- Code simplicity: developer only has to maintain one method and does not have to explicitly track resources.

__Cons:__
- Doesn’t work with foreign fetch
- Requires browsers to poll multiple service workers

### Navigator method
Another approach is to explicitly register/unregister cached content with the browser:

```javascript
navigator.pageIndexing.add({
 	  url: "sw-contents.example.com/video", 
 	  offline: true, 
 	  meta: { 
 	  	title: "example", 
   		description: "...", 
 		  type: "video"
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
- Code complexity: Developer has to track keys and explicitly add/remove content at multiple points in the code base.
- Browser needs to do extra work to clean up registrations when it clears content
- Doesn’t work with foreign fetch
