---
layout: post
title: "Delta Service for Mobile Apps"
date: 2014-02-20 20:59:10 -0700
comments: false
published: true
categories: 
---

A typical RESTful API is simple and intuitive to consume, but requires a lot of service calls in order to download a large dataset. This can be a problem in mobile when an app simply doesn’t have the time or the bandwidth to wait for all the service calls to finish collecting the dataset.

<!-- more -->

There are a few options to try to improve on this:

  1. Create a search service to return just the data that the client needs at the time
  2. Download the entire dataset with a single service call
  3. Pre-install the data on the client (mobile apps only)

### Option #1 - Search

The search option is the only option for extremely large datasets; you simply can’t download all the data from Google Maps onto a mobile device. With search, there is the risk that repeated searches will download some of the same content, wasting bandwidth. Also, every search needs a network connection and takes time to complete, introducing delays to the user experience. In some cases, such as identical search terms, caching can improve upon these issues.

### Option #2 - One-shot

Downloading the entire dataset in a single service call can be significantly faster than the many requests necessary to do so using a RESTful API. A typical RESTful API requires one request to get a list of resources and then n requests to fetch all the resources individually (n+1 requests). The gain with downloading the entire set at once is the removal of the request overhead (and startup delay) for n requests. Additionally, compressing larger response payloads has a bigger benefit than compressing small ones. However, this “download everything” option has the greatest impact on UX as the client has to download and process the entire payload before the user can benefit from the data.

### Option #3 - Pre-install

Pre-installing the data on the client has the biggest UX benefit: immediate availablility of the data. Note that this benefit is only realized after the mobile app has been downloaded and installed from the app store. The initial download takes longer since the app bundle is larger due to the included data. A big issue with packaging the data in the app is that you have to release an update to the app in order to get updated data into the hands of your users. This is a big delay as development, testing and Apple’s review all have to complete before the update is available. The upside to this delay is that during the entire release process users have a copy of the data and can continue using it, but it’s growing stale. Additionally, users have to actually install the update. There are always some users that don’t update their apps. Depending on how critical the data is, this can be a showstopper for some apps.
Both RESTful and search APIs have the downside of requiring a network connection in order to get any data. Previously requested data can be served up from a cache, but that doesn’t help if the user is asking for data that hasn’t been cached. While the “download everything” and pre-installed data options require network connections to get the data into the app, once that is complete the network is no longer required. Thus, these latter options provide the best offline experience.

So, it would seem that we have to choose between the following experiences:

   * Delayed, on-demand data
   * Can’t do anything for a while, but then it’s fast
   * Slow app install, everything’s snappy, stale data

I propose a mix of all three in order to provide the ultimate experience.

## Solution

At a high level, here’s the idea: pre-install a snapshot of the dataset in the app at build time (Option #3). At some point after app launch, the app calls a “delta" API asking for all changes to the dataset since a given point in time (Option #1, search by time). The service responds with either an empty response, or with a payload which includes only the resources that have changed. Where the entire dataset download comes into play (Option #2) is that if there are ever issues with applying the delta, the client has an option to reset its data to a pristine state. Hopefully, this never happens, but if it does it’s better to temporarily degrade the UX rather than let the app continue using stale data forever.

Before I get any further into the details, I want to call out the pros and cons of this delta API approach.

### Pros

   * Data is immediately available after app install
   * Data is available offline
   * Data is updated every time the app is used (provided server is reachable)
   * Server outage or maintenance has little to no impact on client
   * Updates are smaller so less potential for user confusion due to looking at data which changes after the update
   * Searching is fast

### Cons

   * Longer app download times due to larger app binary
   * Not for large datasets (~50MB+)
   * Other than offline, benefits will not be that different from a simple RESTful or search API for small datasets (<100KB)
   * Complex implementation, requiring coordination between server and client
   * Server needs to track and record dates for data changes over time
   * Extra build step to update the pre-installed data

## Design

Let’s start with the “download everything” call. This is just a service endpoint that a GET request invokes a response which contains the entire dataset in question.

```
GET /api/widgets
Accept: */*

HTTP/1.1 200 OK
Date: Tue, 04 Feb 2014 16:30:30 GMT
Last-Modified: Tue, 04 Feb 2014 16:30:30 GMT
Content-Type: application/json

{
    "widgets": [
        {
            "name": "Widget 1",
            "color": "blue"
        }, {
            "name": "Widget 2",
            "color": "yellow"
        }, {
            "name": "Widget 3",
            "color": "red"
        }, {
            "name": "Widget 4",
            "color": "green"
        }, {
            "name": "Widget 5",
            "color": "gray"
        }, {
            "name": "Widget 6",
            "color": "white"
        }
    ]
}
```

This service call will be used by the client whenever there is a problem applying a delta update. It also provides an interface for the build prep process to grab a current copy of the data to be included in the app bundle.

In order to stay as close to the principles of REST, the delta functionality comes into play when the client does a conditional GET and includes an If-Modified-Since header in the request.

```
GET /api/widgets
Accept: */*
If-Modified-Since: Wed, 05 Feb 2014 19:08:16 GMT

HTTP/1.1 200 OK
Date: Thu, 06 Feb 2014 13:31:30 GMT
Last-Modified: Thu, 06 Feb 2014 13:31:30 GMT
Content-Type: application/json

{
    "widgets": [
        {
            "name": "Widget 2",
            "color": "yellow"
        }, {
            "name": "Widget 6",
            "color": "white"
        }
    ]
}
```

The records included in the response body are upserted into the client’s data; any that don’t exist are created, those that do are updated if any of the attributes have changed.

This takes care of creating and updating records, but what about removing records? The simplest approach I’ve been able to come up with is to include a “hidden” attribute with a value of true which instructs the client to stop displaying that record. This approach allows for bringing back records in the future.

```
GET /api/widgets
Accept: */*
If-Modified-Since: Thu, 06 Feb 2014 13:31:30 GMT

HTTP/1.1 200 OK
Date: Fri, 07 Feb 2014 20:04:06 GMT
Last-Modified: Fri, 07 Feb 2014 20:04:06 GMT
Content-Type: application/json

{
    "widgets": [
        {
            "name": "Widget 1",
            "color": "blue",
            "hidden": true
        }
    ]
}
```

The last example is when a client asks for changes, but nothing has changed. In this case the response will have a 304 status code and an empty body. This tells the client it is up-to-date.

```
GET /api/widgets
Accept: */*
If-Modified-Since: Fri, 07 Feb 2014 20:04:06 GMT

HTTP/1.1 304 Not Modified
Date: Sat, 08 Feb 2014 12:01:06 GMT
Last-Modified: Sat, 08 Feb 2014 12:01:06 GMT
```

## Date Handling

With the myriad of problems that can arise with interpreting dates, it is very important for the client to treat the value of the Last-Modified response header as an opaque string and return the exact same value in the If-Modified-Since request header of the next request to the service.

On the server side it’s also important to pay attention to the exact time that the “last modified” attribute gets set on the records. If there is any delay between when this value is set and that record is available to clients through the delta service, any clients calling in during this gap will miss changes. Say you decide to update records in a staging database and the "last modified” timestamp gets set there (Sat, 08 Feb 2014 17:00:00 GMT), meanwhile a client hits the service and receives a response with some changes (Last-Modified header Sat, 08 Feb 2014 17:15:00 GMT) then after the changes have been validated in the staging area they are moved out to the production database (Sat, 08 Feb 2014 18:00:00 GMT). At this point the “last modified” attributes on the recently changed records are an hour old. The client that last called in at 17:15 will ask for changes since 17:15 the next time, but those changes that were applied with a 17:00 timestamp will be skipped for that client. This leads to very strange stale data issues that are difficult to debug.

An alternative to using dates is to use an integer version number. This strays from REST and requires the server to keep track of these numeric versions. But, the server implementation could be as simple as a mapping of version numbers to dates which are then used to query against the “last modified” attribute of the records. Note that the point of version numbers would not be to support returning old version of the data. This is far more work as it requires storing snapshots of every version of every record, at least those versions that have changed. The point of this delta service is to quickly and efficiently update clients to the latest version of the data.

