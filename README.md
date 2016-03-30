# Offline service worker contents
Allowing service workers to provide details about the content of their offline experiences.

See the [EXPLAINER](https://github.com/drufball/sw-content-indexing/blob/master/EXPLAINER.md) for proposed solutions.

## Problem
A registered SW could be a fully interactive offline experience, an app shell, or a skeleton that only enables push notifications. Users have no easy way of identifying and navigating to sites that have full offline experiences when they have no network connection. Many users don’t have an expectation that offline web experiences exist at all.

As service workers gain adoption, users will be interacting with more offline web experiences. Currently those experiences are opaque to browsers, meaning that there is little that can be done to support them. This repo explores ideas that enable service workers to broadcast the contents of their offline experience.

## Use Cases
With more knowledge about the contents of service workers, browsers could take steps to promote offline sites and content. When offline, a browser could only autocomplete URLs that have content to show the user. Browsers could also offer an “Offline” page that lists URLs and content that the user can explore.

Transparent SW contents could be used with [foreign fetch](https://github.com/mkruisselbrink/ServiceWorker/blob/foreign-fetch/foreign_fetch_explainer.md) to create robust ecosystems of collaborating sites.
