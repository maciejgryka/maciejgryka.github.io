---
layout: post
title:  "Building regex.help"
date:   2021-05-13 15:00 +0200
---

Now that [regex.help](https://regex.help) is functional and deployed (check it out if you're writing regular expressions) I want to share how I built it. In the text post will cover deployment and CI/CD setup. In case you want to look at the code, head over to [the GitHub repo](https://github.com/maciejgryka/regex_help).

First: what is [regex.help](https://regex.help)? It's a web interface for [`grex`](https://github.com/pemistahl/grex), which basically writes regex for you. Given a couple of example strings you want to match, grex will generate a regular expression matching all of them. That's pretty much it, although quoting the [README](https://github.com/pemistahl/grex#2--do-i-still-need-to-learn-to-write-regexes-then-top-): *"often, the resulting expression is still longer or more complex than it needs to be"* so you might want to adjust it by hand. You can do that interactively on [regex.help](https://regex.help) while getting real-time feedback for each of the examples you specified.

## The stack

This is the second side-project I've built using Elixir, Phoenix and LiveView (the first one was [Secretwords]({% post_url 2021-05-03-building-secretwords %})). The stack is pretty much the same, including Tailwind for CSS. I was hoping I'll have a use case for [Alpine.js](https://github.com/alpinejs/alpine/) to close the loop and use the entire [PETAL stack](https://thinkingelixir.com/petal-stack-in-elixir/), alas, there was no need for any custom JS this time either.

The main difference this time was that `grex` is a Rust library - so I had to figure out a way to bundle it with an Elixir app. Luckily, a bunch of people already spent a bunch of time on this and the resulting library, [`rustler`](https://github.com/rusterlium/rustler) works pretty well. It's not entirely mature and there were a couple of sharp egdes ([here's how to deal with them]({% post_url 2021-05-21-using-rustler-elixir-1-12-otp-24 %})), but the API is pretty nice and overall it works like you'd expect.

This is pretty exciting! I feel like this stack is powerful: LiveView gives you so much power by itself and then it's also pretty easy to combine with Rust in case you need to do anything computationally inensive.

## The code

Overall mapping how I wanted the app to work to code was straightforward, even if it took a while to get right. There components are:

- An [event for modifying the input](https://github.com/maciejgryka/regex_help/blob/27dc73088b054bc518a61467a79f93769861e3fe/lib/regex_help_web/live/page_live.ex#L22-L28), that is the example strings we want to write a regex for. Every time we modify the input, we check whether it's within some maximum allowed length (I found that giving `grex` more than 1000 characters OOM's on the current infrastructure) and then ask `grex` to generate a regex for us and output it.
- There are various binary flags `grex` accepts and we expose a subset of them. [Every time a flag is toggled](https://github.com/maciejgryka/regex_help/blob/27dc73088b054bc518a61467a79f93769861e3fe/lib/regex_help_web/live/page_live.ex#L34-L45), we recompute the regex using the new settings.
- There's also the option to edit the generated regex and check whether it still matches all the examples given. In this case, [on every edit](https://github.com/maciejgryka/regex_help/blob/27dc73088b054bc518a61467a79f93769861e3fe/lib/regex_help_web/live/page_live.ex#L60-L67), we send both the updated regex and the examples to the rust library and [get back a boolean vector specifying which lines matched](https://github.com/maciejgryka/regex_help/blob/27dc73088b054bc518a61467a79f93769861e3fe/native/regexhelper/src/lib.rs#L39).
- There's a [thin wrapper around the Rust library](https://github.com/maciejgryka/regex_help/blob/main/lib/regex_help/regex_helper.ex), which also defines the allowed flags.
- And the [Rust library](https://github.com/maciejgryka/regex_help/blob/main/native/regexhelper/src/lib.rs) itself, which just build the arguments and passes them to the `grex` and `regex` crates.
- The only remaining part is [the HTML template](https://github.com/maciejgryka/regex_help/blob/main/lib/regex_help_web/live/page_live.html.leex), which LiveView uses to render the page. It also took me a little while to get all the events working correctly, but there's nothing inherently complicated there.

## Wrapping up

Overall this project was a joy: learning cool, new tech while creating something potentially useful. Also it was pretty quick to get working! I spent much more time setting up the infrastructure for deploys and the CI/CD pipeline for it, which I'll cover in the next post.

Please [let me know](https://twitter.com/maciejgryka) if you found this helpful, interesting, or have any feedback.
