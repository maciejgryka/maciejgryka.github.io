---
layout: post
title:  "Bringing more party parrot into your life since 2018"
date:   2019-01-12 07:33:00 +0200
---

Every so often I get the urge to build something that serves no real purpose other than being fun to build. This was one of these times.

Years ago, [party parrot](https://cultofthepartyparrot.com/) was brought on to our Slack at [Rainforest](http://rainforestqa.com/). Since then it has vastly enriched our emoji vocabulary, allowing us to express a wide range of emotions. If someone posts something [mildly interesting](https://www.reddit.com/r/mildlyinteresting/) `:slowparrot:` is the appropriate reaction, however, when you're unreasonably excited about it `:ultrafastparrot:` is the way to go. Sometimes we have to freeze code releases for a while, which we signal with the `:snowflake:` emoji and when it's time to start releasing again it's `:shipitparrot:` time. I lead the science team, so I can't believe I've only just discovered `:scienceparrot:` today, but I foresee a bright future for it on our channel. It might sound ridiculous, but that's just the way it is. `:dealwithitparrot:`

The ever-expanding arsenal of party parrots triggered an alert in my head: there has to be a better, more scalable way! I should be able to make the world a little better[1] and create a way for people to make party-parrot-style emojis out of anything! And so [partyasterisk.com](https://www.partyasterisk.com) was born. You can use it to turn any image you have into a party-style image.

Building it was really fun: it's running on Docker on Heroku, uses the latest version of Python, has all the scientific python libs and has a CD set up on CircleCI. It's such a joy to be able to over-engineer and greenfield project to your hearts content without worrying about it making sense. Pure play! And the ability to [troll your CEO](https://twitter.com/maciejgryka/status/1076092936754946050) is just the icing on the cake.

My plans for it currently include continuing to unreasonably optimize it, add recursion, rewrite it in Rust and deploy on Kubernetes. Check back soon!

[1] Or a little different, at least, depending on your stance on emojis.
