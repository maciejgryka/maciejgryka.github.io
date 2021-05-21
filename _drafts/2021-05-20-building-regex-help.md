---
layout: post
title:  "Building regex.help"
date:   2021-05-13 15:00 +0200
---

- ~intro: what is regex.help, how to use it~
- summary: describe the tech stack used to build regex.help
    - elixir, phoenix, liveview, tailwind
    - grex, rustler
    - live dashboard
- audience: web devs unfamiliar with the Elixir ecosystem
- lessons:
    - PETL stack is cool, still happy after the 2nd toy project
    - some things are still immature (rustler needs an unreleased version)
    - however, Elixir itself is stable

Now that [regex.help](https://regex.help) is functional and deployed (check it out if you're writing regular expressions) I want to share how I built it together with a couple of lessons. In this post I will focus on the buildng part, while the text one will cover deployment and CI/CD setup.

First: what is [regex.help](https://regex.help)? It's a web interface for [`grex`](https://github.com/pemistahl/grex), which basically writes regex for you. Given a couple of example strings you want to match, grex will generate a regular expression matching all of them. That's pretty much it, although quoting the [README](https://github.com/pemistahl/grex#2--do-i-still-need-to-learn-to-write-regexes-then-top-): *"often, the resulting expression is still longer or more complex than it needs to be"* so you mihght want to adjust it by hand. You can do it interactively on [regex.help](https://regex.help) while getting real-time feedback for each of the examples you specified.

## The stack

This is the second side-project I've built using Elixir, Phoenix and LiveView (the first one was [Secretwords]({% post_url 2021-05-03-building-secretwords %})). The stack is pretty much the same, including Tailwind for CSS. I was hoping I'll have a use case for [Alpine.js](https://github.com/alpinejs/alpine/) to close the loop and use the entire [PETAL stack](https://thinkingelixir.com/petal-stack-in-elixir/), alas, there was no need for any custom JS this time either.

The main difference this time was that `grex` is a Rust library - so I had to figure out a way to bundle it with an Elixir app. Luckily, a bunch of people already spent a bunch of time on this and the resulting library, [`rustler`]()
