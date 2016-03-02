# Alice

![](http://i.imgur.com/UndMkm3.png)

[![Hex.pm](https://img.shields.io/hexpm/l/alice.svg)](https://hex.pm/packages/alice)
[![Hex.pm](https://img.shields.io/hexpm/v/alice.svg)](https://hex.pm/packages/alice)
[![Hex.pm](https://img.shields.io/hexpm/dt/alice.svg)](https://hex.pm/packages/alice)

A Lita-inspired Slack bot written in Elixir.

Very much a work in progress, but it works well enough.

For an example bot, see the [Awesome Friendz] bot. For an example
handler, see [Google Images Handler].

You'll need a Slack API token which can be retrieved from the [Web API page] or
by creating a new [bot integration].

[Awesome Friendz]: https://github.com/adamzaninovich/awesome_friendz_alice
[Google Images Handler]: https://github.com/adamzaninovich/alice_google_images

[Web API page]: https://api.slack.com/web
[bot integration]: https://my.slack.com/services/new/bot

## Creating Your Own Bot With Alice

### Create the Project

Create a new mix project.
```sh
mix new my_bot
cd my_bot
rm lib/my_bot.ex
```

### Configure the Application

In `mix.exs`, bring in alice and any other handlers you want. You also need to
include the `websocket_client` dependency because it's not a [hex] package.
[hex]: http://hex.pm
```elixir
defp deps do
  [
    {:websocket_client, github: "jeremyong/websocket_client"},
    {:alice,                  "~> 0.1.0"},
    {:alice_against_humanity, "~> 0.0.1"},
    {:alice_google_images,    "~> 0.0.1"}
  ]
end
```

In the same file, configure the app, registering any handlers you want. You can
add handlers through dependencies, or you can write them directly in your bot
instance. (We recommend putting them in `lib/alice/handlers/my_handler.ex`)
```elixir
def application do
  [ applications: [:alice],
    mod: {
      Alice, [
        Alice.Handlers.Random,
        Alice.Handlers.AgainstHumanity,
        Alice.Handlers.GoogleImages
      ] } ]
end
```

In `config/config.exs` add any configuration that your bot needs.
```elixir
use Mix.Config

config :alice,
  api_key: System.get_env("SLACK_API_TOKEN"),
  state_backend: :redis,
  redis: System.get_env("REDIS_URL")

config :alice_google_images,
  cse_id: System.get_env("GOOGLE_CSE_ID"),
  cse_token: System.get_env("GOOGLE_CSE_TOKEN"),
  safe_search_level: :medium
```

With that, you're done! Run your bot with `iex -S mix` or `mix run --no-halt`.

### Deploying to Heroku

Create a new heroku app running Elixir.
```sh
heroku create --buildpack "https://github.com/HashNuke/heroku-buildpack-elixir.git"
```

Create a file called `heroku_buildpack.config` at the root of your project.
```sh
erlang_version=18.2.1
elixir_version=1.2.1
always_rebuild=false

post_compile="pwd"
```

Create a `Procfile` at the root of your project. If you don't create the proc as
a `worker`, Heroku will assume it's a web process and will terminate it for not
binding to port 80.
```ruby
worker: mix run --no-halt
```

You may also need to reconfigure Heroku to run the worker.
```sh
heroku ps:scale web=0 worker=1
```

Add your slack token and any other environment variables to Heroku
```sh
heroku config:set SLACK_API_TOKEN=xoxb-23486423876-HjgF354JHG7k567K4j56Gk3o
```

Push to Heroku
```sh
git add -A
git commit -m "initial commit"
git push heroku master
```

Your bot should be good to go. :metal:

## Creating a Route Handler Plugin

### First Steps

```sh
mix new alice_google_images
cd alice_google_images
rm lib/alice_google_images.ex test/alice_google_images_test.exs
mkdir -p lib/alice/handlers
mkdir -p test/alice/handlers
touch lib/alice/handlers/google_images.ex test/alice/handlers/google_images_test.exs
```

### Configuring the App

In `mix.exs`, update `application` and `deps` to look like the following:

```elixir
def application do
  [applications: []]
end

defp deps do
  [
    {:websocket_client, github: "jeremyong/websocket_client"},
    {:alice, "~> 0.1.4"}
  ]
end
```

### Writing Route Handlers

In `lib/alice/handlers/google_images.ex`:

```elixir
defmodule Alice.Handlers.GoogleImages do
  use Alice.Router

  command ~r/(image|img)\s+me (?<term>.+)/i, :fetch
  route   ~r/(image|img)\s+me (?<term>.+)/i, :fetch

  def handle(conn, :fetch) do
    conn
    |> extract_term
    |> get_images
    |> select_image
    |> reply(conn)
  end

  #...
end
```

### Registering Handlers

In the `mix.exs` file of your bot, add your handler to the list of handlers to
register on start

```elixir
def application do
  [ applications: [:alice],
    mod: {Alice, [Alice.Handlers.GoogleImages, ...] } ]
end
```
