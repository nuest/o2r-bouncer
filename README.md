# o2r bouncer

Node.js implementation for the oauth2-authentification process of the [o2r web api](https://o2r.info/o2r-web-api)

Requirements:

- nodejs `>= 6.2`
- npm
- modified global-tunnel with tunnel 0.0.4

## Global-tunnel

o2r bouncer uses the `global-tunnel` module, which is unfortunately unmaintained. You will need to `npm install`, then change the dependency version of `tunnel` in the `node_modules/global-tunnel/package.json` file to `0.0.4` and do a `npm install` again. Without this modification, o2r bouncer will fail when used with a `http_proxy` environment variable.

## Dockerfile

This project includes a `Dockerfile` which can be built with

```bash
docker build -t o2r-bouncer .
```

The image can then be run and configured via environment variables. For convenience, we include a `docker-compose` configuration, which can be run with

```bash
cd docker-compose && docker-compose up
# after you're done, shutdown and delete all volumes (data):
docker-compose down -v
```

### Available environment variables

* `BOUNCER_PORT`
  Define on which Port muncher should listen. Defaults to `8083`.
* `BOUNCER_MONGODB` __Required__
  Location for the mongo db. Defaults to `mongodb://localhost/`. You will very likely need to change this.
* `BOUNCER_MONGODB_DATABASE`
  Which database inside the mongo db should be used. Defaults to `muncher`.
* `OAUTH_URL_AUTHORIZATION`
  Authorization URL for the ORCiD OAuth2 API. Defaults to `https://orcid.org/oauth/authorize`.
* `OAUTH_URL_TOKEN`
  Token URL for the ORCiD OAuth2 API. Defaults to `https://orcid.org/oauth/token`.
* `OAUTH_SCOPE`
  Scope for the ORCiD API. Defaults to `/authenticate`.
* `OAUTH_CLIENT_ID` __Required__
  The client ID for your instance.
* `OAUTH_CLIENT_SECRET` __Required__
  The client secret for your instance.
* `http_proxy`
  HTTP Proxy used for the oauth process. This will be needed if your server is behind a Proxy/firewall. _Important:_ See the above details on updating the `global-tunnel` dependency before you use this.
* `SLACK_BOT_TOKEN`
  Authentication token for a bot app on Slack. See section [Slack bot](#slack-bot).
* `SLACK_VERIFICATION_TOKEN`
  Token provided by Slack for interative messages and events, to be used to verify that requests are actually coming from Slack.
* `SLACK_CHANNEL_STATUS`
  Channel to post status messages to, defaults to `#monitoring`.
* `SLACK_CHANNEL_USER`
  Channel to post messages on user events, defaults to `#monitoring`.
* `SLACK_USERNAMES_WHITELIST`
  A regex to check Slack usernames, which are allowed to react to interactive messages. Defaults to `.*` and the created regex is always case _in_sensitive. To allow specific users only, use e.g. `\\b(claerbout|peng|stodden)\\b` (case insensitive match of full words `claerbout`, `peng`, `stodden`).

### Generating the client ID & secret

See the [ORCiD Guide](https://members.orcid.org/api/accessing-public-api). As a redirect URI you need to set the path `/api/v1/auth/login`, relative to your base url. We highly recommend using `https`. The client ID & secret then need to be provided as environment variables or directly saved to the `config/config.js` file.

## Slack bot

Documentation of Slack API: https://api.slack.com/bot-users, especially [interactive messages](https://api.slack.com/interactive-messages).

The bot needs the permissions to join channels and post to them.
Add the following scopes to the app in the section "OAuth & Permissions" in the bot's apps page.

- `channels:write`
- `chat:write:bot`
- `incoming-webhook`
- `bot`

While adding the app to your Slack organisation, make sure to allow the bot to post the the desired channel.

### Local bot development

Start ngrok with `ngrok http 8083` and enter the public endpoint pointing to your local server at https://api.slack.com/apps/A6J6CDLQK/interactive-messages. ngrok also has a useful web interface at http://127.0.0.1:4040/inspect/http on all incoming requests.

## Devlopment

You must provide the required settings as environment variables, either at start time or via the debug configuration of your IDE.

To start bouncer with the required environment replace the respective settings with the actual values (check your ORCiD page) and execute the following command.

```bash
DEBUG=* OAUTH_CLIENT_ID=clientid OAUTH_CLIENT_SECRET=secret npm start
```

You can then start the authentication process by opening http://localhost:8083/api/v1/auth/login and see who you are afterwards at http://localhost:8083/api/v1/auth/whoami.

## License

o2r bouncer is licensed under Apache License, Version 2.0, see file LICENSE.

Copyright (C) 2016 - o2r project.
