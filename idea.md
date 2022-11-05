---
layout: post
title: Improving web privacy & UX
description: Idea for better user experience while having privacy
language: en
tags:
 - gdpr
 - privacy
 - spec
last_modified_at: 2022-11-05T12:20:00+02:00
sitemap: true
published: true
---

Note: Following is work-in-progress, and just quickly drafted from my mind
related to discussion at <https://mastodontti.fi/@vrntnn/109290555339487403>.

Since dawn of GDPR (25th May 2018) most sites have been nagging user and asking
them to select how their data should be handled.  

This makes user experience (UX) pretty awful, has user has to do selections
always when they navigate to a new site, and if eg. browser has cleaned up the
cookies storing the previous preferences.

What if we instead had site-specific policy files, describing the privacy
policies and eg. cookies the specific site uses. Then user could just define
their preferences via browser settings (which can be then synced to all of
their devices).


## `/.well-known/privacy-policy.json`

This would be the site-specific privacy policy, where site describes how it
tracks it's users. Current example includes just cookies, but this could easily
be extended to support eg. browser localStorage, or eg. to describe some GDPR
data processing policies, like "here is our human-readable privacy policy".

There might already exist some formats describing eg. cookie usage in
machine-readable format, and such formats might be preferable to inveting yet
another new standard. Large part of this whole idea is to force browsers to by
default deny all tracking, and require sites to describe all data they collect
before allowing anything.

```json
{
  "meta": {
    "version": "2022-11-05T15:15:15+00:00",  // ISO-8601, usually last updated
                                             // timestamp
    "type": "fi.ypcs.spec.privacy-policy.v0" // type of this JSON file
                                             // (reference to specification,
                                             // so your device knows how this
                                             // document should be interpreted)
  },
  "cookies": [
    {
        "name": "mycookie",
        "description": "Human-readable description of the policy.",
        "type": "<analytics|functional{.preferences,...}|tracking>",
        "mandatory": "false",
    },
    {
        "name": "_(ga|gid|gat|AMP_TOKEN|gac_.*)", // also support some simple
                                                  // regular expression format
        "url": "https://wwww.google-analytics.com/.well-known/privacy-policy.json",
        "type": "analytics"
    }
  ]
}
```

This policy would be domain-specific, and each site could provide their own
policy. Eg. if you visit site `example.net`, that site would have
`example.net/.well-known/privacy-policy.json`. If site uses eg. Google
Analytics, those servers could have their own polícy as well.

By forcing site to define all names of the third-party service cookies as well,
this would also prevent that third-party  service from adding any additional
trackers without the consent of the *site owner* as well.

Browsers supporting this protocol must then deny all cookies (and other,
current or future) tracking technologies that have not been described in the
policy. Optionally browsers could have some anonymized reporting feature,
collecting information about sites violating the protocol, ie. sites that try
to use cookies outside the policy.


## Known issues

If browser would openly report your preferences to any server you access, that
could make it possible to identify users based on your privacy preferences. Eg.
malicious site could list tens or hundreds of different cookies, to try
fingerprint your decisions.

So, it might be necessary to nag user, show some information about the cookies
site is using, and ask if we should use user's global default preferences. When
encountering a new item, eg. new site with new cookie, user would be asked if
that cookie should be allowed.

Ideally user selections would consist of following data:

1. site that sets the tracker, `example.net`
2. type of the tracker (eg. cookie)
3. decision (allow|deny)

so the question, *asked by the browser*, based on the site privacy policy,
could be something like

> This site uses following tracking methods:
>
>  - Functional cookie `mycookie`, described as "the description provided by
>    site owner, trying to justify why this should be accepted"
>  - Google Analytics (your policy: allow)
>
>  Allow / Customize / Deny

Browser would then store this information. Later, when user visits same site on
eg. different device, but with synced browser settings, browser could just
allow or deny based on the previous selections.

Also, if user chooses to allow certain service globally, in our example Google
Analytics, the questions could use previously set defaults.

This would not remove the nag from the first visit on a site, but would allow
browsers to store the decisions, and only ask once ever unless site changes
it's policy. On the other hand, as question would be asked, user would control
if their previously chosen preferences should be communicated to the site. It
would also be possible to notice if site tries to ask preferences for "too
many" different services, which could be related to a fingerpriting
attempt.
