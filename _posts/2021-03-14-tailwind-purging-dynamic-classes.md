---
layout: post
title:  "Tailwind, puring & dynamic classes"
date:   2021-03-14 15:00 +0200
---


When working with TailiwindCSS, be careful to not generate CSS classes dynamically - if you do, some of your CSS classes might be missing in production as explained in the docs. The reason for this is Tailwind using PurgeCSS to delete unused classes and PurgeCSS being intentionally naive about how it detects which classes are used.

For example, if you do something like 

```html
<div class="text-<%= team %>-600">...</div>
```

you might expect that the text will be red if team = "red". But it probably won't be, because PurgeCSS doesn't know what `<%= team %>` evaluates to and doesn't see `text-red-600` anywhere in your code, so it removes it. I say "probably", because you might be using `text-red-600` somewhere else, in which case this class will be included and that particular style will work, making it all a little tricky to debug.

There are a couple of workarounds. You can do what the docs suggest and just conditionally output entire class names, like this

```html
<div class="<%= if team == 'red' do %>team-red-600<% else %>team-blue-600<% end %>">...</div>
```

This is fine, I guess, but gets unwieldy if you have a bunch of distinct possibilities, not just two. Maybe you have a component, say "product" and it can have one of 4 states: coming-soon, available, sold-out, deprecated. Having a 4-way `if` in your templates is a pain; it'd be nicer if you could just generate the class name dynamically.

The best way I found for this is to define all these variants as Tailwind components and then preventing Tailwind from purging any components. It's not ideal - you might end up with unnecessary CSS classes in your production bundle.

Is there a better way?
