---
layout: post
title:  "Building Secretwords"
date:   2021-05-04 15:00 +0200
---

As [mentioned previously]({% post_url 2021-04-09-secretwords %}), I recently built a simple online game while learning the Elixir/Phoenix web stack. This post describes what I built and how. This was a fun learning experience for me and hopefully useful for you too. The finished project is here: https://github.com/maciejgryka/secretwords

## Requirements

So first, what are we building? Let's see:

- It should be a web-based game, with the same rules are [Codenames](https://codenamesgame.com/).
- It should use [Elixir](https://elixir-lang.org/), [Phoenix](https://www.phoenixframework.org/) and maybe- [LiveView](https://github.com/phoenixframework/phoenix_live_view) (importantly, it should be server-side-rendered- as much as possible with minimal Javascript). While tech stacks mostly don't belong in requirements, I'm doing- this to learn a specific stack, so there.
- It should work in real-time. In other words, the state of the game needs to be kept in sync and auto-updated for- everyone at the same time.
- It's OK to keep everything in memory, even if it's lost on restarts. No persistence, no databases.
- No logins, authentication - anyone can connect and play immediately without an account.


That's pretty much it! I think - writing these down retrospectively is not ideal, but this should covers my initial intentions.

## Setup

Great, now we have a starting point - let's take the first step to having a working app. Before installing Elixir and Phoenix, I recommend setting up [asdf](https://asdf-vm.com/) - it will make your life easy down the line. Follow [installation instructions](https://asdf-vm.com/#/core-manage-asdf?id=install) from there and then add [erlang](https://github.com/asdf-vm/asdf-erlang), [elixir](https://github.com/asdf-vm/asdf-elixir) and [nodejs](https://github.com/asdf-vm/asdf-nodejs) (don't panic! node is needed in development, but  we won't be using it directly).

```bash
asdf plugin add erlang
asdf plugin add elixir
asdf plugin add nodejs
```

Once you do that, you can create a new phoenix project with

```bash
mix archive.install hex phx_new
mix phx.new secretwords --no-ecto --live
```

The `--live` flag sets up LiveView and `--no-ecto` means no PostgreSQL support will be added; we won't be using a database.

Now you have a start of the project (if you need more details, follow the [official Phoenix tutorial](https://hexdocs.pm/phoenix/up_and_running.html)). Let's skip to the fun part: how does it feel to write Elixir code?

## Writing Elixir code

I don't know where it was, but the first time I've heard about Phoenix was something like "Oh Elixir is just a nice language, which compiles to Erlang and Phoenix is a Rails-like web framework for Elixir". Let me tell you, the second half of that sentence is a lie.

Elixir is a nice language, it does compile to Erlang and Phoenix is a web framework for Elixir. But if you know Rails, abandon any hope of knowing anything about Phoenix, other than "how the web works". Writing Elixir is a totally different experience to Ruby and nothing really carries over in terms of how to structure your app, how to think about solving problems etc. While authors of both Elixir and Phoenix came from the Rails community and you can see some inheritance if you look closely enough, it felt like an alien world to me. I was excited to visit and this world feels increasingly like home - but it is different. And I feel it's important to know this, because I struggled for a while unnecessarily trying to fit what I was learning into what I already knew. Only once I realized it was a completely different beast and made space in my head for that fact, was I able to learn and appreciate everything properly.

Probably the clearest two differences are functional programming (instead of object-oriented) and immutable data structures. I was so used to OO and mutability that it took some real effort to switch mindsets. Let's go through a quick  example: I store the state in a data structure I call, imaginatively, `GameState`. It has a bunch of attributes: id, the grid of words on the "table", who's on which team, how many points each team has etc. In the Rails world, it would probably be a class with a bunch of attributes and methods to modify them. For instance a method to join a team might be something like:

```ruby
def join(color, user_id) do
  @game.teams[color].add(user_id)
  @game.ensure_leaders()
end
```

While my Elixir implementation is:

```elixir
def join(game, color, user_id) do
  updated_teams = %{game.teams | color => MapSet.put(game.teams[color], user_id)}

  game
  |> Map.put(:teams, updated_teams)
  |> ensure_leaders()
end
```

Keep in mind that this is kind of the worst case scenario for Elixir - modifying deeply-nested, immutable data structures is challenging in the functional world. So much so, that [José Valim himself recently asked for crowd-sourced opinions about the best way of solving such problems in different languages](https://twitter.com/josevalim/status/1379771275627921409) (and [this blog post](https://nickjanetakis.com/blog/100-ways-to-solve-a-specific-programming-problem-in-50-languages) has more background info).

To spell it out, what I want to do is updating a single field in a 3-deep map. To do this, I basically need to hand-update 3 different maps:

- update the list of team members by calling `MapSet.put`,
- update the `teams` map with  the resulting list of team members,
- finally update the game itself with the new `teams`.


Other than than, there are a few things going on here:

- There is no `self` or anything similar - we're purely operating on arguments and we just accept `game` struct (not "object") as one of our arguments.
- We also return an updated `GameState` struct, though it's a different one (as in, it's stored at a different memory location) than the one we accepted. Since everything is immutable and we want to make changes, we create new structs.
- In rails we can assume e.g. `@game.teams[color]` is a `Set` and we can just call its `add` method, which is pretty convenient. In Elixir, we can use `MapSet`, however, since there are no classes or objects, we call `MapSet.put` explicitly, so it's a bit more verbose. I'm on the fence about the relative importance of being succinct vs. explicit here.
- In both cases we call `ensure_leaders()`, which just makes sure that some rule constraints are satisfied after every update.
- The pipe operator `|>` might seem strange at first, but you get used to it very quickly.


There are other differences, not obvious from this snippet. An important one is having no early return, but instead gaining great pattern matching on function arguments itself. As a result you end up writing many smaller functions with fewer branches, which I feel is is a readability win. For example, I have a log_activity function, which takes either one message or a list of messages and appends them to some field:

```elixir
def log_activity(game, messages) when is_list(messages) do
  %{game | activity: messages ++ game.activity}
end

def log_activity(game, message) do
  %{game | activity: [message | game.activity]}
end
```

This is nice! I have two functions instead of one and they're all very clear. I have the freedom to name second argument in singular or plural. Compare to the equivalent in Python:

```python
def log_activity(self, messages):
  if isinstance(messages, list):
    self.activity += messages
  else:
    self.activity.append(messages)
```

It's still a small function, but feels less satisfying. I'd probably not write it like this at all and just have a single version, which always takes lists and I'd call it like `game.log_activity(['some message'])` when there's only a single message.

In the end every language has trade-offs and I'm really enjoying the ones Elixir makes.

## Persistence

I knew up-front there will be no need to permanently store any data. Still, we do need to remember the state of the game while it’s in progress in a way that’s accessible to all players. Generally a way to store some state in Elixir is a [GenServer](https://hexdocs.pm/elixir/GenServer.html), which can store things and let other processes read and write them. However, I also wanted to see what [ETS](https://elixir-lang.org/getting-started/mix-otp/ets.html) is all about so I threw it into the mix as a learning experience.

What I ended up with is not ideal - I have a single GenServer, called `GameStore`, responsible for reading from and writing to ETS tables. It works, but it's not isolated - if anything goes wrong with a game it will affect all the others too. It would've been better, probably, to isolate them so each game would have its own GenServer. I might experiment with doing that later.

One cool thing about ETS, though, is that it should be pretty easy to swap with [DETS](https://erlang.org/doc/man/dets.html), which is disk-based. I thought trying to preserve some state across restarts might be an interesting experiment. Losing the state of all games on restart  (e.g. when I deploy) is definitely not ideal - but I never got to solving it.

The way it all works is that I pass `GameState` structs around, each one representing a single game. The `GameStore` has functions to retrieve and update existing game states as well as a convenience `get_or_create` function, which returns a game if it already exists or creates a new one if it doesn't. All games are identified by an ID, which is a random string.

There's a similar thing going on with users - while there's no logins, I wanted to both be able to identify players throughout a session and let them set a nice, human-friendly nickname. The `UserStore` just maps a random ID to a username - otherwise it's very similar to the `GameStore` (and also uses ETS).

Broadcasting changes
When someone changes their nickname, we want all the other players to know about it. Since each player basically has a process with their own copy of the game state, this doesn't happen automatically. I.e. if Bob changes his username to Ben, Alice would have to reload the page to see that update. Not ideal - and luckily it's very easy to change.

The way to deal with this is using a PubSub - there are basically two lines of code to add:

- `PubSub.broadcast!` whenever a change is made,
- `PubSub.subscribe` inside the LiveView `mount` function to force re-rendering when anything is broadcast.


With these additions (once for `GameStore` and once for `UserStore`) everything works smoothly - I'm still amazed at how easy this is to do.

## Plugs

Another neat abstraction in Phoenix are plugs - if you've ever used something like Django, you can think of plugs as middlware: they take a request, modify it and pass it along. I've added two plugs: one to set the user_id and one to make sure the user is assigned to some team in the current game.

The first plug does two things: it makes sure that the conn struct (which represents the current user's session) has a user_id, crating it if necessary and it also makes sure that the map user_id -> username exists in ETS. While user_ids are unique and static, the username mapping is used to display the names in human-readable form. It's possible that multiple users will end up with the same name - this is fine!

The other plug only takes effect once a user joins some game and makes sure that the user is assigned to one of the teams. This is strictly not necessary, I could've allowed users to join a game without joining a team, but I thought having this constraint would make things easier. If the current user does not belong to any team, we assign them at random.

## Testing

Testing is important to me (I wouldn't be working at [RainforestQA](https://www.rainforestqa.com/) if it wasn't), but I always struggle to find a satisfying balance on side projects. I definitely want to have some harness to make sure things don't break without having to manually check all the features at every release. At the same time, the real value of testing comes after some time and I'm never sure I'll be working on any side project long enough to get that pay-off.

However, the more projects I work on, the more I realize the payoff from testing (and also documentation) comes sooner than I expect.

Most of the pieces were very easy to test, scoring another point for functional style and immutable data. Both these traits guide you towards writing many small functions with no side effects and these are really nice to write tests for. For this reason testing the `GameState` module representing the core logic was a breeze. Tests for the LiveView itself were nothing to write home about - not bad, but not amazingly convenient either and I'm still not sure how much I need them given the other layers.

The "top of the testing pyramid", the functional tests, were fun to figure out. I had no idea how to write functional tests for multiplayer scenarios! The game has a bunch of constrains, among them the minimum number of players. For instance some interface elements only show up after the game is started and you can only start it with at least 4 players present.

Luckily this turned out to be pretty easy to do technically with [wallaby](https://github.com/elixir-wallaby/wallaby) - but it still requires quite a bit of management, so I'm sure a better solution is possible. Each feature test can accept multiple sessions, each representing a player. From there it's just a matter of making sure all the interactions happen in the right order. It looks something like this

```elixir
@sessions 4
feature "four players can start", %{sessions: [player1, player2, player3, player4]} do
  game_path = Routes.live_path(@endpoint, SecretwordsWeb.GameLive, 'game_id')

  player1
  |> visit(game_path)
  |> assert(...)

  player2
  |> visit(game_path)
  |> assert(...)

  player3
  |> visit(game_path)
  |> assert(...)

  player4
  |> visit(game_path)
  |> assert(...)
end
```

It does the job, but is a bit of a pain to manage - just like any Selenium-like testing framework. I've had a couple of problems, which were very obvious visually (e.g. Tailwind issues, see below), but were not caught by my integration tests. They mostly fell into the category of "important, but impossible to specify by XPath selectors" and are the main reason we're not fans of such tests at Rainforest (AJ wrote [a comprehensive blog post](https://www.rainforestqa.com/blog/the-downfall-of-dom-and-the-rise-of-ui-testing) about that). However, I couldn't easily use Rainforest, because we don't yet have a great multiplayer testing story.

Finally, I get lots of pleasure from my code being buttoned-up, so I've also added code formatting (using the built-in `mix format --check-formatted` command) and style checks (using [credo](https://github.com/rrrene/credo)) to my standard test command.

I also started write type specs for my code and really wanted to set up dialyzer to perform analysis each time I deploy. However, it takes quite a long time to run from scratch I had some trouble with caching, so I couldn't find a practical way to use it. I'm sure it's doable and not too tricky - but it's something I'll have to figure out later.

Finally, I wanted a CI/CD pipeline, because I really believe having it forces you into better habits. I got some way there! I got CI (Continuous Integration), but didn't build out the CD (Continuous Deployment) part. In other words, I use GitHub Actions to run all the tests mentioned above on every commit to every branch - but I have to run a couple commands locally to deploy the code to production. The [setup-beam GH Action](https://github.com/erlef/setup-beam) from the Erlang Ecosystem Foundation makes things pretty easy.

## Tailwind

[TailwindCSS](https://github.com/erlef/setup-beam) is all the hype these days, so I wanted to try it out. Half-way through building, [the JIT](https://blog.tailwindcss.com/just-in-time-the-next-generation-of-tailwind-css) was announced so I gave that a shot and it worked pretty well. Besides some flailing around with the initial setup, the only bump in the road I hit was using dynamically-generated classes, which I covered before. Since then I reverted back to just using more verbose conditionals, which output full class names instead of partial strings. It's not perfect, but doesn't bother me too much.

Other than that, I also played with TailwindUI, but I think I'll need some more time with it to make it feel really useful - the designs are beautiful, but I end up customizing them so much that they lose their charm.

## Deployment

The last piece of the puzzle is deployment. There are a bunch of options, but since this was mostly about learning, I wanted to set up a server from scratch. I've done this a couple of time in the past for Python projects and Elixir is easier generally to deploy: the compiled release is just a bunch of binary files you can pretty much drop onto a server and expect to work.

The specifics are kinda boring: set up a plain Ubuntu server with automatic updates, ufw, nginx. The part, which was new to me this time was setting up systemd to make sure the server process always runs - it was still pretty straightforward, though.

Finally, I wrote 3 scripts inspired by [scripts-to-rule-them-all](https://github.com/github/scripts-to-rule-them-all): server, test, and deploy.

This was a fun project and I learned a lot!
