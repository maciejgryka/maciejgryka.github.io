---
layout: post
title:  "Setting up Plausible, Hitting the Tracker Wall"
date:   2021-05-13 15:00 +0200
---

After [buiding regex.help]({% post_url 2021-05-04-regex-help %}) I was curious about how much traffic it gets. [fly](https://fly.io) provides some basic metrics, but that wasn't enough to really know how many people find the site helpful.

For a while now I've been hearing about both [Plausible Analytics](https://plausible.io/regex.help) and [Fathom](https://usefathom.com). They share a bunch of similarities and the ones most important for me are:

- they're both provacy-first, aiming to give you useful information without tracking your visitors personally,
- there are no cookies, so no need to add any banners,
- they're fairly simple UI-wise, exposing just the high-level info.

### Features

It wasn't initially clear to me what the differences were - after a buch of reading, turns out there aren't that many! They're both very easy to set up, Plausible has a lower/cheaper pricing tier and a couple more features, which are just now in the preview version of Fathom (v3) - but otherwise they seem to be very comparable.

- Fathom
    - $14/month, 100k monthly pageviews,
    - unlimited websites,
    - uptime monitoring.

- Plausible
    - €6/month, 10k monthly pageviews, or €12/month, 100k monthly pageviews,
    - 20 websites,
    - built with Elixir <3.

- Both
    - blocked by uBlock Origin, even after DNS masking,
    - no cookies,
    - email reports,
    - serve JS from own domain,
    - privacy-focused,
    - Google Search Console integration (Fathom V3),
    - track campaigns with UTM (Fathom V3),
    - filtering (Fathom v3),

One annoyance is that they are both blocked by uBlock Origin, even after [setting up a DNS CNAME record](https://plausible.io/docs/custom-domain/) to serve the script from own domain.

![Fathom blocked by uBlock Origin](/static/2021-05-13-fathom-blocked.png)
![Plausible blocked by uBlock Origin](/static/2021-05-13-plausible-blocked.png)

This is bittersweet - on the one hand I love uBlock origin and have been using it for years, so I'm glad it's doing a good job blocking trackers. On the other - this makes these solutions much less useful for me. I want to build things for technical people and they're likely to have something like uBlock running. The alternative is probably server-side analytics? I guess I'll need to investigate that next.
