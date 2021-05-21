---
layout: post
title:  "Using Rustler with Elixir 1.12/OTP 24"
date:   2021-05-21 15:00 +0200
---

Read this if you want to get `rustler` running on the new, shiny [Elixir 1.12/OTP 24](https://elixir-lang.org/blog/2021/05/19/elixir-v1-12-0-released/).

The problem is that, as I'm writing this, `rustler` had a bunch of fixes, which did not yet make it to an official Hex release. So if you try to just add the following to your `mix.exs` (as [hex](https://hex.pm/packages/rustler) tells you to), your project will not build on the new version of Elixir/OTP:

```elixir
# this does not work with Elixir 1.12
{:rustler, "~> 0.21.1"}
```

You might think that specifying `"~> 0.22.0-rc.0"` as the version might save you, since there's an unreleased Release Candidate with some changes (the `rustler` API got much nicer!). However, you need to get even more cutting edge than that and use the `master` branch from the git repository, anything after [commit 4786f2](https://github.com/rusterlium/rustler/commit/4786f233a971058d0f677e0686bdf465dd458f01).

There are a couple more gotchas: firstly, the [`rustler` repo](https://github.com/rusterlium/rustler) contains code for both the Rust and the Elixir packages. The Elixir package is in the [`rustler_mix` directory](https://github.com/rusterlium/rustler/tree/master/rustler_mix) so you need to do a [`sparse` git checkout](https://www.git-scm.com/docs/git-sparse-checkout) to tell Elixir where to find the package. In the end, this is the line to add to your `deps`:

```elixir
{:rustler, git: "git://github.com/rusterlium/rustler.git", branch: "master", sparse: "rustler_mix"},
```

Finally, and this took me way too long to figure out, you also need to update your `Cargo.toml` to use the `master` branch. When you create a new `rustler` project, `Cargo.toml` is synced with `mix.exs`, but when updating you need to remember both places. Add this:

```toml
rustler = {git = "git://github.com/rusterlium/rustler.git", branch="master"}
```

Note than you don't need the `sparse` checkout here, because the root of the repo is valid Rust package.
