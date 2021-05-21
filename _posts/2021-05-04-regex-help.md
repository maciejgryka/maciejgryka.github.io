---
layout: post
title:  "regex.help"
date:   2021-05-04 15:00 +0200
---

Next time you're trying to write some regex, check out [regex.help](https://regex.help) - it should make your task much easier. A couple of weeks ago, after [Secretwords was finished]({% post_url 2021-05-03-building-secretwords %}), a colleague pointed me towards [`grex`](https://github.com/pemistahl/grex). It's a really neat tool to help you write regex.

The main idea is that you provide a couple of examples of the text you want to match and `grex` spits out a pattern matching all of them.

Usually, the way you write regex is probably something like this:
1. have one or more examples of the thing you want to match in your head,
1. write a regex that you think might match them,
1. try it out,
1. find out it's wrong, fix it and try again.

This is fine and there are many tools already, which help you get there. However, with grex you can skip a step:

1. enter one or more examples of the thing you want to match,
1. you now have a regex and it's correct (!!!), 
1. it's likely not exactly what you want though, so you can still iterate.

I think this is pretty neat. Since I'm currently in the Elixir/Phoenix/LiveView honemoon land, this also gave me a chance to play with [`rustler`](https://github.com/rusterlium/rustler), which is an Elixir library allowing you to embed some Rust code. I was very pleasantly surprised how easy it was to plug it all together.

Of course, another post, about how I built [regex.help](https://regex.help) is on the way. One fun difference from Secretwords: I deployed [regex.help](https://regex.help) on [fly.io](https://fly.io/) and I'm very excited about this platform.
