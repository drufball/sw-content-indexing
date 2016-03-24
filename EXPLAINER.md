# Offline service worker contents
## Problem
A registered SW could be a fully interactive offline experience, an app shell, or a skeleton that only enables push notifications. Users have no easy way of identifying and navigating to sites that have full offline experiences when they have no network connection. Many users don’t have an expectation that offline web experiences exist at all.

As service workers gain adoption, users will be interacting with more offline web experiences. Currently those experiences are opaque to browsers, meaning that there is little that can be done to support them. This repo explores ideas that enable service workers to broadcast the contents of their offline experience.

## Use Cases
With more knowledge about the contents of service workers, browsers could take steps to promote offline sites and content. When offline, a browser could only autocomplete URLs that have content to show the user. Browsers could also offer an “Offline” page that lists URLs and content that the user can explore.

Transparent SW contents could be used with [foreign fetch] to create robust ecosystems of collaborating sites.

## Proposals
This is a new area of exploration, with many proposals.

### Manifest URL
The manifest format could be extended to include an `offline_url` attribute:

```json
"offline_url": "sw-contents.example.com/enumerate-cache"
```

The service worker would intercept requests to this URL, inspect its cache, and respond with a JSON object specifying its contents, along with the URL necessary to view those contents:

```javascript
[
    {
 		    url: "sw-contents.example.com/videos",
 		    content-type: "video",
 		    sources: ["example1", "example2"]	
 	  },
 	  {
 		    url: "sw-contents.example.com/articles",
 		    content-type: "text",
 		    sources: ["example3", "example4"]
 	  }
]
```

The sources array could be extended to specify things like preview thumbnails, explainer text, etc. 

A browser (or any site collaborating via foreign fetch) would poll the `offline_url` when constructing its offline content viewer.

__Pros:__
- Minimal API surface area additions to manifest or SW
- Works with foreign fetch
- Robust by default against content deletion or changes

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

__Cons:__
- Doesn’t work with foreign fetch
- Requires browsers to poll multiple service workers

### Navigator method
Another approach is to go with a strictly JavaScript approach:

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

__Cons:__
- Developer has to track keys and explicitly add/remove content
- Browser needs to do extra work to clean up registrations when it clears content
- Doesn’t work with foreign fetch
