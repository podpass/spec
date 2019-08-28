# PodPass Technical Specification DRAFT 0.2

## Status of this Document

This is an early, incomplete draft meant to spark conversation and raise questions and potential areas for improvement. It is not bound for submission as a standard, but responses to it will shape a future standards-bound document.

> Paragraphs in blockquotes offer thinking around a particular approach, future enhancements, or other text which will likely not appear in a complete technical spec.

## Purpose

PodPass is a simple API implemented by podcast hosts and an interaction flow implemented by podcast client apps. The goal of PodPass is to introduce a lightweight, web-like identity layer to podcast subscriptions, on an opt-in basis by listeners, which can then be used by publishers to support new and better experiences for podcasts where an identity can enhance the listening experience. Examples include services which offer ad-free listening experiences for logged-in listeners, podcasts which release episodes on a schedule which is determined by the date of subscription, and podcasts with a closed back-catalog.

## Design Goals

 * **DO NOT break current pseudo-anonymous listening semantics.** Completing a PodPass flow requires a direct action on the part of the listener, and must always be 100% reversible with a simple action. PodPass feeds require a public component.
 * **Be transparent about listener identity state.** At a glance, a listener should be able to tell whether they have completed the PodPass flow or not.
 * **Avoid assumptions about the purpose of identity.** Building a system which offers a tool without use-case-specific design affordances will result in a system which can support multiple future use cases.
 * **Everything flows from the public feed.** To avoid the potential for hijacking, private feed adoption must be initiated based on metadata found in a public feed. This has a few side-effects, including that all private feeds must have a corresponding public one.
 * **Design for third-party private feed platforms.** While the public feed needs to advertise its support for PodPass, it should be possible for third-party services to be advertised in the discovery mechanism without requiring significant involvement from the original host.

## Prior Art

There are currently three major ways podcasts with an identity requirement work:

 1. **Offered obscurely, shared.** These are podcast feeds which are kept behind a login gate or secret URL, but all authorized listeners subscribe to the same RSS URL. Thus, it is not possible to revoke one listener’s access without impacting all listeners. Leaking this feed URL is very easy and likely very common. As a benefit to listeners, listening is pseudo-anonymous, depending on the number of listeners accessing the feed.
    _Examples: Maximum Fun, Radiotopia Bonus Content_
 2. **Offered obscurely, personally.** These are podcast feeds kept behind login gates but which are personal to each individual listener. They can be shared, but revoking one listener’s access only impacts that listener and anyone their feed URL leaked to. While these can still be leaked, it’s possible to try to identify leaks.
    _Examples: Supporting Cast, Glow, Patreon.com, Cafe Insider, Acast Access_
 3. **Non-podcast.** These are “podcasts” which are not available via RSS. They vary widely enough to defy generalization, but they are not the target of this spec.
    _Examples: Stitcher Premium, Luminary, Emails with links to MP3s._

## Overview

### Identify Flow

The Podcast Host adds a Discovery Tag to their Public Feed. The Client App of a subscribed Listener detects the discovery tag and begins displaying the Disconnected Badge in conjunction with the podcast within the app.

The listener decides to complete the Identity Flow, taps the Identity UI Element in the client app, and is presented with the Identity Flow Page indicated in the discovery tag loaded in their mobile browser. After completing some task presented by the identity flow page (e.g. username & password credentials), the page uses the App Communication API to send an Identity Payload to the client app. After confirming with the listener, the client app will store the payload for the podcast and immediately download the podcast feed, but this time will include the Identity Token in the request headers and receives a Private Feed.

If the identity payload included Compatible Feeds, the listener is presented with the option to quick-connect their identity to the compatible feeds. The Adopt Flow is automatically completed for each feed the listener chooses.

The Identified Badge is shown in conjunction with all now-connected podcast. All future requests for the podcasts and their Enclosures will include the token as part of the request headers.

### Adopt Flow
> It came up several times during my early conversations about podpass that we should offer a way to log in to several feeds at once. Because one of the things we want to avoid is recreating the cookie environment we have on the web, it was important to me that we didn’t do anything automatically without prompting the user. This flow also makes it possible to advertise on-network shows to the listener that they are not yet following. 
Importantly, it’s still completely opt-in (that is, it is possible for me to listen to one in-network program with an identity and still remain pseudo-anonymous on other programs in the same network).
We will need to think through how we handle potential edge cases bordering on abuse, such as a major shared host identifying all feeds as compatible.

Given an active identity token, Source Public Feed URL, and Destination Public Feed URL, client app fetches and parses destination public feed url, finds the Adopt Endpoint URL in the discovery tag if present, and POSTs the token and source public feed URL to the adopt endpoint URL.

The adopt endpoint returns an identity payload if the incoming token is valid. Client app stores necessary information and completes the identity flow for destination. The compatible feeds portion of the returned identity payload is ignored if present (the adopt flow is non-recursive).

The identified Badge is shown in conjunction with all now-connected podcasts. All future requests for the podcasts and their enclosures will include the token as part of the request headers.

### Manage Flow

The Identity Flow has already been completed, and the client app has the appropriate data stored to display content from the private feed. The podcast hosts offers a Manage Page URL in the discovery tag on the private feed.

The listener decides to change their membership level, so they select the Manage Component of the identity UI element and are presented with the manage page in their mobile browser. After making the changes they like, the manage page optionally uses the app communication API to send data back to the app, which automatically updates the token if sent.

On return to the client app, the feed is refreshed to ensure the latest information is shown.

### Disconnect Flow

The Identity Flow has already been completed, and the client app has the appropriate data stored to display content from the private feed.

The listener selects the Disconnect Component of the identity UI element and, after confirming, the client app deletes any stored token for that feed. It reverts to the public feed, stored at the beginning of the Identity Flow. The feed is refreshed to ensure the latest information is shown.

## Term Definitions

* Adopt Endpoint: an optional API endpoint for allowing tokens for one feed to be converted automatically into tokens for another feed when advertised by the source feed.
* App Communication API: the mechanism by which data is passed between the client app and identity flow page or manage page, defined fully in this document.
* Client App: a PodPass compliant application which can be used to consume PodPass compliant podcasts.
* Compatible Feeds: additional feeds referenced in the identity payload which can be shortcut-connected using the token in the identity payload.
* Destination Public Feed: during the adopt flow, the public feed for which the target identity payload is meant. Receives a token from the source feed.
* Disconnected Badge: a UI element which indicates that, while the podcast in question supports PodPass, the Identify Flow has not yet been completed or, if it has, the Disconnect Flow has been completed more recently.
* Disconnect Component: a UI element which is always displayed in conjunction with the identified badge to allow the listener to revert to the disconnected state via the disconnect flow.
* Discovery Tags: XML tags indicating that the containing RSS feed supports PodPass and required metadata for its operation.
* Enclosure and Linked Enclosure: media files included in the RSS feed as links in enclosure XML tags.
* Identified Badge: a UI element which indicates that the podcast in question supports PodPass and the client app currently has identity information stored for inclusion in requests for the feed and linked enclosures. Optionally, includes podcast-provided information about the state of the linked identity (e.g. membership level).
* Identity Flow Page: a web page referenced from the public feed discovery tag with the goal of generating an identity payload and sending it to the client app via the app communication API.
* Identity Payload: Information sent via app communication API to the client app, including an identity token, alternate private feed URL, and other compatible feeds.
* Identity Token: an opaque token included as a Bearer token on requests for feeds and linked enclosures.
* Identity UI Element: a standard UI element for initiating PodPass flows.
* Listener: the person controlling the client app, and dictating the operation of the PodPass defined flows.
* Manage Component: a UI element which is displayed to launch the manage page when available.
* Manage Page: a web page referenced from the private feed discovery tag with the goal of allowing the listener to manage some aspect of their connected identity. Optionally, can use the app communication API to send a new identity payload which is automatically accepted.
* Podcast Host: party responsible for managing the RSS feed and its content.
* Private Feed: the version of the feed served when the identity token is included on the request and, if present, the alternate private feed URL is requested.
* Public Feed: unobscured, publicly available and indexed version of the RSS feed. This is a required component of any PodPass compliant podcast.
* Source Public Feed: during the adopt flow, the feed for which the original token was issued.

## API Definitions

### Identity Payload

The Identity Payload is made up of the following data, stored as a map data structure.

* auth, which is a string to be included as a bearer token on future requests for enclosures and feeds. This MUST NOT be blank, but MAY be ignored by the podcast host on subsequent requests if it is not required for the working of the podcast host’s system.
* url, which is a string representation of the URL which should be considered the canonical feed for the duration of the connected state. This MUST NOT be blank, but if no alternate URL is used, this MAY be the same as the public feed url.
* compatible, an optional array structure of feeds which can also be accessed using the same token, in the Compatible Feed data structure format.

### Compatible Feed

Other feeds which share the same identity system can be cross linked to minimize the number of Identity Flows required, for example in cases where a full network of feeds use the same system. Compatible feeds are included in the Identity Payload, as members of the list data structure stored as compatible. They have the following map structure:

* url, string, the canonical public feed URL of the compatible feed
* imageUrl, string, URL for a square image to display for the referenced podcast
* title, string, referenced podcast title


Therefore, an example complete Identity Payload may look like: (whitespace added for clarity)

```json
{
    "auth": "f382b506-eaff-4f8d-af5d-3366576c6249",
    "url": "https://example.com/rss/authenticated.xml",
    "compatible": [
        {
            "url": "https://example.com/altrss/public.xml",
            "imageUrl": "https://example.com/altrss/image.png",
            "title": "Example.com Altcast"
        }
    ]
}
```

### App Communication API

The Identity Payload will be communicated to the client app by invoking the javascript function window.opener.postMessage() with the following signature:

```javascript
window.opener.postMessage( <JSON payload>, <string literal "*"> );
```

This is done to ensure a single flow works for all client apps, on the web and native apps. In native apps, the page should be loaded in an environment with the javascript window.opener.postMessage function defined as a javascript-to-native hook. On the web, invoking this function will allow the frame which opened the pop-up-window to receive the payload.

The JSON payload is the stringified form of a Javascript object with a single attribute, “podPassID,” containing the Identity Payload.

Example:

```javascript
window.opener.postMessage(JSON.stringify(
    {
        "podPassID": {
            "auth": "f382b506-eaff-4f8d-af5d-3366576c6249",
            "url": "https://example.com/rss/authenticated.xml",
            "compatible": [
                {
                    "url": "https://example.com/altrss/public.xml",
                    "imageUrl": "https://example.com/atrss/image.png",
                    "title": "Example.com Altcast" 
                }
            ]
        }
    }
), "*");
```

Other actions to be defined in the future (e.g. dismiss) will be sent with the same mechanism. It’s possible that there will be environments in which window.opener.postMessage won’t be a good option - in that case, we will instead define a global object `PodPass` via the inclusion of a Javascript library, which will attempt a sequence of communication strategies.

This global object may be overwritten by the environment itself, and may offer a nicer interface than possible with postMessage.

> I’ve had several conversations with folks who prefer to render the page in the platform browser and have a redirect back to the app (mostly for flows that break out of the browser and rely on being able to easily return).

> Given the requirement that we have of avoiding a pre-registration flow between client app developers and hosting providers (akin to OAuth) this necessarily requires an open redirect which introduces some security concerns unless we define the spec carefully. We also have the issue of the flow from browser back to native app on iOS being less than ideal in some circumstances (prompting the user).

> We could mitigate the security issues by strictly defining requirements for apps that they issue a request ID at flow initiation and not honor any requests with an unknown request ID, but it’s likely that we’ll see flawed implementations.

> Privacy issues with using the system browser include access to the system cookie store during the authentication flow, though this could be seen as a benefit in some cases.

> One killer feature for using the system browser may prove to be ApplePay, which requires it.

> One other possibility suggested would be to add a “break-out-to-browser” flow which can be requested by the provider, but given that we’d be fragmenting the environment if any client apps decided to support the rest of PodPass and not that feature, the benefits seem limited at best.

### Adopt Endpoint API

The Adopt Endpoint, defined in an adopt tag, is a URL which responds to POST requests containing a JSON-formatted Adopt Payload, defined as:

```json
{
    "sourceUrl": "<canonical public url of source feed>",
    "auth": "<id token from source feed>"
}
```

Adopt endpoints respond to valid requests with an Identity Payload. The compatible portion may be ignored and can, as always, be excluded from the response. Some apps may choose to store this compatible information for use when listeners start following a show that is compatible with an existing identity on their device.

## Discovery Tags

The Discovery Tags are XML tags added within the <channel> tag of a feed. The identify tag is required, and the rest are optional. They are defined under a namespace, referred to here as “pass”, to be defined in the future.

> There’s no particular reason, as far as I can tell, to prefer a single tag with many attributes or multiple tags. Because the three discovery tags can technically exist separately (as some are only useful on public feeds, others private) it made sense to break them out as separate entities to me.

### ID Tag

The one required tag is the ID tag, which points to the Identify Flow page. It should be added to the public feed within the <channel> tags. A feed without this tag cannot initiate the identify flow.

Example:

```xml
<pass:id href="https://example.com/podpass/identify" />
```

The ID tag has no meaning in a private feed.

### Adopt Tag
The optional Adopt Tag can be included in a public feed if the feed supports acting as a destination feed during the adopt flow. Feeds which do not include this tag are not eligible for use as the destination feed in adopt flows.

The spec as written makes it possible for a feed to have an adopt tag and no id tag. I think this does a nice job of covering cases where the public feed is essentially a placeholder and not meant to be followed by anyone, but offered as a supplement for folks following other feeds, as is the case with current “grab-bag” private feeds.

Example:

```xml
<pass:adopt href="https://example.com/podpass/adopt" />
```

The Adopt Tag has no meaning in a private feed.

### Manage Tag

The optional Manage Tag can be included in private feeds when a page is offered for the ongoing management of one’s identified state.

```xml
<pass:manage href="https://example.com/podpass/manage" />
```

The Manage Tag has no meaning in a public feed.

## Display Tags

In addition to discovery tags, PodPass defines a tag which manages the display of the PodPass badge within the client app.

### Label Tag

This optional tag defines additional copy and imagery for display within the PodPass badge in the client app. The optional image-url attribute can be used to display a small icon in addition to the required text label.

> Given that there are a couple of different meanings for this tag, perhaps it makes sense to move the public case to the :id tag, since it’s meaningless without it. It can stay as :label in the private case, where no other tags are required.

```xml
<pass:label image-url="https://example.com/supporter/level-1.png">Level 1 Supporter</pass:label>
```

In a public feed, this label should be rendered near the PodPass identify flow button, and should be used to convey some aspect of the program to which the listener will be connecting (e.g. “Supporters of Example”)

In a private feed, this label should be rendered near the PodPass badge, and its content should be something which meaningfully conveys either an aspect of the identity of the listener (e.g. username) or the relationship between the listener and the podcast.
