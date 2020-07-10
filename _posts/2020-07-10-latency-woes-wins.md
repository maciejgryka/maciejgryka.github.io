---
layout: post
title:  "Latency woes and wins"
date:   2020-07-10 15:00 +0200
---

Increasing latency in modern interfaces is annoying & many people have been writing about it (Dan Luu, recent HN post). I've been particularly annoyed by some data visualization tools - I love exploring datasets, but quickly get annoyed if I have to wait for the output too long. Interactivity is super important to maintaining flow.

Web technologies themselves are getting better overall - much more powerful and convenient; it's impressive! But at the same time, latency seems to be increasing and I'm personally not happy with that trade-off.

Some things are trying to reverse that trend - e.g. Phoenix seems like a pretty cool tech with potential to make things snappier. Some JS bundlers are the result of devs being fed up waiting for things to build.

Among such things, WebAssembly excites me the most. It's truly cross-platform and can be pretty fast. On top of that, it plays well with Rust, which everyone want to learn these days, including me. I smell an opportunity.

As a proof of concept for something I was thinking about for a while, I wanted to see if I can process non-trivial amount of data in the browser quickly. I don't want to wait for a response from the server if it's not necessary and I'm OK with trading off initial loading time to have better flow once I start work.

Measurement is always the tricky part: checking whether things take 20ms or 200ms is difficult to do precisely, even though the difference in user experience is massive. There are some ways to get better-than-nothing measurement, e.g. "Is it Snappy" iOS app. It doesn't really scale to any systematic and repeatable approach, but it's a place to start.

With all that said, here's an experiment: [https://maciej.gryka.net/lab/rustwasm1/](https://maciej.gryka.net/lab/rustwasm1/). Drop a CSV file into it to check it out. You can use any CSV file you have, which contains a timestamp column, or just some [generated sample data](https://maciej.gryka.net/lab/rustwasm1/?csv=/static/data/sample_events.csv). If you look at the JS console, you'll see that grouping half a million rows by minute takes about half a second. Not bad, I think?
