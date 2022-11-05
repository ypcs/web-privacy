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
fingerprint your decisions
