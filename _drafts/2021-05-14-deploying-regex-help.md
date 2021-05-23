---
layout: post
title:  "Deploying regex.help"
date:   2021-05-22 15:00 +0200
---

Now that I've written up [how regex.help was built]({% post_url 2021-05-20-building-regex-help %}), it's time to focus on deployment. I used to think deployment was boring, but recently I find it more and more exciting, probably because I care more about good CI/CD. Elixir and [fly.io](https://fly.io) make it even more cool and shiny - let's jump in!

In my [last side project]({% post_url 2021-05-03-building-secretwords %}), I wanted to see how it feels deploying everything by hand. It's fine - but not very exciting. This [sad, little script](https://github.com/maciejgryka/secretwords/blob/main/script/deploy) does most of the work, but then I still need to SSH into the server and restart. If I wanted to make it do everything, I'd probably end up with something similar to [what David did here](https://github.com/zestcreative/elixir-utilities-web/blob/main/bin/deploy).

Also, starting a new project requires a bunch of setup (I wrote down the steps [here](https://github.com/maciejgryka/secretwords/blob/main/docs/provision.md)). In the past, I atomated stuff like this with [Ansible](https://www.ansible.com/), but didn't get to doing it this time.

## Fly

Which is just as well - because it made me look at other options and find [fly.io](https://fly.io/). Fly is great - it takes your `Dockerfile` and magically makes it run on servers around the world, as close to your users as possible. So when you visit your site from Berlin, you'll probably get a response from the Frankfurt data center, but if you're in San Francisco, Sunnyvale, CA will probably handle it.

The way this happens is all pretty well-described in the [Fly docs](https://fly.io/docs/introduction/) and their blog. While it initially only sounds good for stateless/DB-less apps, they actually have a great story for databases too (probably the thing I will play with next).

Also, it's the kind of place, where the [founders respond to your support requests on the forum](https://community.fly.io/t/elixir-buildpacks-error/1234/20?u=maciej) and [HN legends retweet your tweets](https://twitter.com/tqbf/status/1395838083111739395) when you mention Fly - so that's nice.

## Deployment

OK, enough with the fawning - how do you actually deploy this thing? As mentioned, you need a `Dockerfile` first. It can be pretty simple in the simplest cases - you can use the official elixir Docker image, e.g. [elixir:1.12](https://hub.docker.com/_/elixir/) and serve your app using `mix server`. In my case [the `Dockerfile` got a bit more involved](https://github.com/maciejgryka/regex_help/blob/main/Dockerfile), because I needed Rust and played with multi-stage builds a little. Fly also has [a good example in their Elixir setup guide](https://fly.io/docs/getting-started/elixir/#dockerfile).

Once I had a working `Dockerfile`, following that guide didn't take very long and my app was live. That was neat - `flyctl` feels like a Heroku-like CLI tool and the process was generally smooth, including the [custom domain setup](https://fly.io/docs/app-guides/custom-domains-with-fly/) and getting a TLS certificate.

At this point I could easily make changes to the app and deploy using `fly deploy`. Time to move on to...

## Continuous Integration

Of course, I didn't want to stop there - the code should get tested and deployed on `git push`.

- summary: describe how regex.help is deployed, contrast with Secretwords
    - fly vs. DO VPS
    - CI!
    - staging!
    - rainforest tests!
    - CD!
- writing for: web devs aspiring to do CI/CD
