---
title: How Javascript Service Workers Increased Our Site Speed by 97.5%
description: Here’s how we made our documentation site load 97.5% faster by
  using javascript service workers by reducing file sizes and improving
  bundling.
author: yonatankra
published: true
published_at: 2023-08-03T10:20:28.256Z
updated_at: 2023-08-03T10:20:28.299Z
category: engineering
tags:
  - javascript
  - vivid
  - performance
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
Here’s how we made our website load 97.5% faster by using service workers, how we ensure the users will get the newest version every time, and how you can do it too.

[Vivid’s design system’s website](https://vivid.deno.dev/) has come a long way in just one year. It started with a few basic pages and has now grown into a full-fledged documentation site, showcasing live examples of how to use our product. 

We use code splitting extensively in our code, and our product includes various static resources like icons and CSS files. To top it off, our documentation uses iFrames to present our components, each calling its dependencies separately (yes, like a Microfrontend architecture).

But as time passed, things began to break.

It started with our dev environment. After a few reloads, it would just get stuck.

![Vivid Docs Loading Error](https://yonatankra.com/wp-content/uploads/2023/06/Docs-error.gif "vivid-docs-loading-error.gif")

That, we could live with. What we couldn’t live with was a ticket coming from our users showing us how bad the situation truly was.

![Chrome Performance User Ticket](/content/blog/how-javascript-service-workers-increased-our-site-speed-by-97-5/user-ticket-vivid.png "vivid-user-ticket.png")

We knew we had to do something about it. We had to speed up our website. Here’s how we debugged and fixed our website’s performance.

## The Performance Boost

Let’s start from the end. Our main goal was to reduce the number of calls to the server. A secondary goal (or bonus, if you will) was speeding up the load time of pages in our app. You can see the results and judge for yourself:

**Before**

![Vivid Performance Before Service Workers](/content/blog/how-javascript-service-workers-increased-our-site-speed-by-97-5/vivid-performance-before.png "vivid-performance-before.png")

\
In the image above, we can see the profile of the network requests before the change. At some points, the page load took 860ms – a huge amount of time! The average load time of a page was around 400ms. In the rightmost section, you can see the gray bar of the site trying to load the `all.css` file, which keeps going forever. This causes the site to stop loading and if the user tries to refresh, it actually makes it worse!

![Vivid Performance Before Service Workers](/content/blog/how-javascript-service-workers-increased-our-site-speed-by-97-5/vivid-performance-after.png "vivid-performance-after.png")

So, how did we do it?

## Problem: The Chrome Connection Limit

Chrome offers a limit of 6 concurrent connections to a server. What surprised us was the long timeout and that the requests remained “live” even when refreshing or browsing a different page.

Because we had almost a hundred requests on every page (we code-split to an extreme), Chrome would just shut down the connection to our website for every user after just a few minutes of browsing. So we were left with a mission: reduce the number of files that Chrome needed to load.

## Problem: Too Many Files

### Attempt #1: Remove Prefetch

How were we sure that we had too many files? When we opened chrome inspector, our code looked like this: 

![Chrome Inspector Loading Too Many Files](/content/blog/how-javascript-service-workers-increased-our-site-speed-by-97-5/vivid-prefetch.png "chrome-inspector-before-service-workers.png")

These files are just a small part of our files manifesto. See that we also prefetch the files to speed up the load time of the following pages.

Our first step was to stop the prefetch. Although this reduced the number of requests, it did not help because the prefetch happened only when the network was idle, so there wasn’t much effect. We needed to remove the number of scripts requested.

**Attempt Number 2: Bundle All of the Files Into One Big Bundle**

In our project, we use [`rollup`](https://rollupjs.org/) to bundle our files. Our configuration is meant to code split everything and let the consumers use their own bundlers to code split, bundle, and tree shake.

In this step, we just went over all of the components, created a barrel file, and bundled all of them into one big \`vivid-components.js\` file we used instead of all the other script tags. Check out this [commit](https://github.com/Vonage/vivid-3/pull/1208/commits/83d9b8fe1a2ce9452bcabb40fdc6123efef4c777) to see how we did it.

It helped on most pages but – remember the iFrames? They still brought a lot of stuff — duplicates of the files already brought by the top page.

So this brought us to our final and most effective solution: *service workers*.

## **Solution: Service Workers**

A service worker is a layer between our app and the network. It can listen to all requests coming in and out of the app and handle them.

In our case, we wanted to handle the requests, cache the response, and return the response on consequent requests.

### **How to Register a Service Worker**

The first step is to register the service worker in the client:

```javascript
(async function() {
	const registration = await navigator.serviceWorker.register(
		'/sw.js',
		{
			scope: '/',
		}
	);
})();
```

The service worker can fetch requests according to its containing folder. This is why I put it at the root of my project.

In our project, the actual file is not in the root. It moves there in our build process, hence giving us a nice development experience while still allowing us to fetch requests from the root. You could use the [service-worker-allowed http-header](https://www.w3.org/TR/service-workers/#service-worker-allowed) to serve it in a different folder, but using the “build to root” trick, we had no need for it.

### Service Worker Functionality

Our service worker looks like this:

```javascript
const addResourcesToCache = async (resources) => {
	const cache = await caches.open('vivid-cache');
	await cache.addAll(resources);
};

const putInCache = async (request, response) => {
	const cache = await caches.open('vivid-cache');
	await cache.put(request, response);
};

const cacheFirst = async ({ request, preloadResponsePromise, fallbackUrl }) => {
	const responseFromCache = await caches.match(request);
	if (responseFromCache) {
		return responseFromCache;
	}

	const preloadResponse = await preloadResponsePromise;
	if (preloadResponse) {
		console.info('using preload response', preloadResponse);
		await putInCache(request, preloadResponse.clone());
		return preloadResponse;
	}

	try {
		const responseFromNetwork = await fetch(request);
		await putInCache(request, responseFromNetwork.clone());
		return responseFromNetwork;
	} catch (error) {
		const fallbackResponse = await caches.match(fallbackUrl);
		if (fallbackResponse) {
			return fallbackResponse;
		}
		return new Response('Network error happened', {
			status: 408,
			headers: { 'Content-Type': 'text/plain' },
		});
	}
};

const enableNavigationPreload = async () => {
	if (self.registration.navigationPreload) {
		await self.registration.navigationPreload.enable();
	}
};

self.addEventListener('activate', (event) => {
	event.waitUntil(enableNavigationPreload());
});

self.addEventListener('install', (event) => {
	event.waitUntil(
		addResourcesToCache([
			'./',
			'./index.html',
			'/assets/styles/core/all.css',
			'/assets/scripts/vivid-components.js',
			'/assets/scripts/live-sample.js',
		])
	);
});

self.addEventListener('fetch', (event) => {
	event.respondWith(
		cacheFirst({
			request: event.request,
			preloadResponsePromise: event.preloadResponse,
			fallbackUrl: './assets/images/vivid-logo.jpeg',
		})
	);
});
```

We have two utility functions: `addResourcesToCache` to add a resource to the cache and `putInCache` to put a request and its response into the cache.

They both use the [caches](https://developer.mozilla.org/en-US/docs/Web/API/caches) global object that gives us access to the [CacheStorage](https://developer.mozilla.org/en-US/docs/Web/API/CacheStorage) object.

`cacheFirst` is where the magic happens. It tries to get the response from the cache. If it finds it, it returns the cached response. (Lines 12-15)

If not, it tries to get the response from a preload. If it works, we’re good – we cache it and return the preloaded response. (Lines 17-22)

If this fails, we move on to request from the network (e.g., the server), get the response, and cache it. (Lines 24-27)

If all fails, we just return an error with a picture. (Lines 29-35)

### Service Worker Lifecycle

The service worker lifecycle:

1. Registration (we’ve been through that)
2. Installation
3. Activation

Our service worker listens to the installation phase and adds our main resources to the cache. 

```javascript
self.addEventListener('install', (event) => {
	event.waitUntil(
		addResourcesToCache([
			'./',
			'./index.html',
			'/assets/styles/core/all.css',
			'/assets/scripts/vivid-components.js',
			'/assets/scripts/live-sample.js',
		])
	);
});
```

Notice the utility `waitUntil` we get on the event object. This utility helps us avoid race conditions as it awaits for async operations to finish.

Then, in the `activate` phase, we enable content pre-loading (with `waitUntil`).

```javascript
const enableNavigationPreload = async () => {
	if (self.registration.navigationPreload) {
		await self.registration.navigationPreload.enable();
	}
};

self.addEventListener('activate', (event) => {
	event.waitUntil(enableNavigationPreload());
});
```

The final step is to add a listener to `fetch`. This listener intercepts the requests and allows us to handle them using our `cacheFirst` function:

```javascript
self.addEventListener('fetch', (event) => {
	event.respondWith(
		cacheFirst({
			request: event.request,
			preloadResponsePromise: event.preloadResponse,
			fallbackUrl: './assets/images/vivid-logo.jpeg',
		})
	);
});
```

Notice the `respondWith` utility. It literally does exactly what it says – given the request, we can return any response. In this case, we return the result of `cacheFirst`.

## How to Handle Versions in a Service Worker

There might be a time in which you’d like to update a service worker’s version. In our case, it is needed for our library’s update. For this, we must state the version in our Service Worker’s file, create a versioned cache and delete the old cache.

### How to Create a Versioned Cache in a Service Worker

Adding a version is straightforward: `const VERSION = ‘3.17.0’;`

This can be changed manually on every release.

In our project, for example, we use rollup to bundle, so we did the following “trick”:

1. We set the version this way: `const VERSION = ‘SW_VERSION’;`
2. During the build, we extract the version from our `package.json`
3. We use the rollup’s replace plugin to set the version on every build.

You can see our setup [here](https://github.com/Vonage/vivid-3/blob/main/apps/docs/rollup.config.js#L64).

### How to Create a Versioned Cache in a Service Worker

If you noticed, the `caches.open` method accepts a string:

`const cache = await caches.open(VERSION);`

It expects a cache name or id we can later reference.

You’ll often need to update the service worker or your website’s cache. For instance, when you bump a library version, add a new section to the page, and every change you make to the response, and want it to show without waiting for cache expiration.

Using the version as the id allows us to address the cache of each version and thus display the current version and delete old ones.

### How to Delete Obsolete Cache in a Service Worker

Now that we have versioned cache, we can delete it. 

Let’s create a function `removeOldCache`:

```javascript
async function removeOldCache(event) {
	await caches.keys().then(function (keys) {
		return Promise.all(keys.filter(function (key) {
			return key !== VERSION;
		}).map(function (key) {
			return caches.delete(key);
		}));
	}).then(function () {
		return self.clients.claim();
	});
}
```

The function goes over all the cache keys (line 2), and for every key that is not the new activated version, we delete it (line 6). Once this is done, we use the `self.clients.claim` method to tell the browser our new Service Worker controls all tabs now (line 9).

We call this function during activation:

```javascript
self.addEventListener('activate', (event) => {
	event.waitUntil(removeOldCache(event));
	event.waitUntil(enableNavigationPreload());
});
```

Notice that the cache is the global cache for all the service workers. In our case, we can safely remove them, but in your case, you may have more than one cache and thus might consider using a suffix to the version to remove only the cache you want.

An example is a case in which you’d want to separate HTML, and server requests caches. The HTML cache will clear when you change clientside-related areas, while the server requests cache will vary on some other term.

### Updating Service Workers

New Service Workers have a waiting period. It might take a few hours to replace them. We can skip this waiting period by adding `self.skipWaiting()` in our install phase.

Now our Service Worker will update immediately when we change it (e.g., upload a new version of the library).

## Summary

Service Workers are a very powerful tool for web developers. They allow us to control the communication between the server and the client. Here we saw the classic example of caching content, but many more use cases exist.

For instance, if we cache the server’s responses and return them, the user can keep using our application while offline.

This caching example shows our solution to our problem – and it indeed solves it. But Service Workers answer many other needs in development. I’d be happy to hear yours 🙂Let me know what you are building with Service Workers on [Twitter](<https://twitter.com/VonageDev>) or the [Vonage Community Slack](https://developer.vonage.com/en/community/slack)! 

*Thanks a lot to [Oria Biton](https://www.linkedin.com/in/oria-biton-2930a522b/) and [Miki Ezra Stanger](https://www.linkedin.com/in/miki-stanger-153bb365/) for the kind and thorough review of this article*