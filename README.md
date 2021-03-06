# Slack Events API adapter for Node and Express

[![Build Status](https://travis-ci.org/slackapi/node-slack-events-api.svg?branch=master)](https://travis-ci.org/slackapi/node-slack-events-api)
[![codecov](https://codecov.io/gh/slackapi/node-slack-events-api/branch/master/graph/badge.svg)](https://codecov.io/gh/slackapi/node-slack-events-api)

## Installation

```
$ npm install --save @slack/events-api
```

## Configuration

Before you can use the [Events API](https://api.slack.com/events-api) you must
[create a Slack App](https://api.slack.com/apps/new), and turn on
[Event Subscriptions](https://api.slack.com/events-api#subscriptions).

In order to complete the subscription, you will need a **Request URL** that can already respond to a
verification request. This module, combined with the use of a development proxy, can make this
easier for you.

1.  Start the verification tool:
`./node_modules/.bin/slack-verify --token <token> [--path=/event] [--port=3000]`. You will need to
substitute your own verification token for `<token>`. You may also want to choose your own `path`
and/or `port`.

2.  Start your development proxy. We recommend using [ngrok](https://ngrok.com/) for its stability,
but using a custom subdomain will require a paid plan. Otherwise,
[localtunnel](https://localtunnel.github.io/www/) is an alternative that gives you custom subdomains
for free.

  > With ngrok: `ngrok http -subdomain=<projectname> 3000`

  > With localtunnel: `lt --port 3000 --subdomain <projectname>`

3.  Input your Request URL into the Slack App configuration settings. This URL depends on how you
used the previous two commands. For example, using the default path and the subdomain name "mybot":

  > With ngrok: `https://mybot.ngrok.io/event`

  > With localtunnel: `https://mybot.localtunnel.me/event`

4.  Once the verification is complete, you can terminate the two processes (verification tool and
development server). You can proceed to selecting the event types your App needs.

**NOTE:** This method of responding to the verification request should only be used
**in development**. After you deploy your application to production, you should come back and modify
your Request URL appropriately.

## Usage

The easiest way to start using the Events API is by using the built-in HTTP server.

```javascript
// Initialize using verification token from environment variables
const createSlackEventAdapter = require('@slack/events-api').createSlackEventAdapter;
const slackEvents = createSlackEventAdapter(process.env.SLACK_VERIFICATION_TOKEN);
const port = process.env.PORT || 3000;

// Attach listeners to events by Slack Event. See: https://api.slack.com/events/api
slackEvents.on('message.im', (event)=> {
  console.log(`Received a DM event: user ${event.user} in channel ${event.channel} says ${event.text}`);
});

// Start a basic HTTP server
slackEvents.start(port).then(() => {
  console.log(`server listening on port ${port}`);
});
```

### Using with Express

For usage within an existing Express application, you can route requests to the adapter's express
middleware by calling the `expressMiddleware()` method;

```javascript
const http = require('http');

// Initialize using verification token from environment variables
const createSlackEventAdapter = require('@slack/events-api').createSlackEventAdapter;
const slackEvents = createSlackEventAdapter(process.env.SLACK_VERIFICATION_TOKEN);
const port = process.env.PORT || 3000;

// Initialize an Express application
const express = require('express');
const bodyParser = require('body-parser');
const app = express();

// You must use a body parser for JSON before mounting the adapter
app.use(bodyParser.json());

// Mount the event handler on a route
// NOTE: you must mount to a path that matches the Request URL that was configured earlier
app.use('/event', slackEvents.expressMiddleware());

// Attach listeners to events by Slack Event. See: https://api.slack.com/events/api
slackEvents.on('message.im', (event)=> {
  console.log(`Received a DM event: user ${event.user} in channel ${event.channel} says ${event.text}`);
});

// Start the express application
http.createServer(app).listen(port, () => {
  console.log(`server listening on port ${port}`);
});
```

## Documentation

To learn more, see the [reference documentation](docs/reference.md).

## Support

Need help? Join [dev4slack](http://dev4slack.xoxco.com/) and talk to us in
[#slack-api](https://dev4slack.slack.com/messages/slack-api/).

You can also [create an Issue](https://github.com/slackapi/node-slack-events-api/issues/new)
right here on GitHub.
