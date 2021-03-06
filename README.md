# PubSub Multi

Multiple implementations of Publish-Subscribe.

* In-memory
* HTTP EventSource and POST as transport

Both implementations share the same contract tests, allowing them to be used interchangeably.

The API for publishing is simple:

```javascript
// publisher is an instance of MemoryPublisher
await publisher.publish('some-signal')
```

The API for publishing too:

```javascript
// publisher is an instance of MemoryPublisher or EventSourcePubSub
const subscriber = publisher.makeSubscriber()
await subscriber.subscribe('some-signal', async () => {
  console.log('received some-signal')
})
```

## HTTP

To use this over HTTP, mount the express middleware in your express app:

```javascript
const express = require('express')
const bodyParser = require('body-parser')
const { MemoryPublisher, pubSubRouter } = require('pubsub-multi')

const publisher = new MemoryPublisher() // Implements both pub and sub interface

const app = express()
app.use(bodyParser.json())
app.use(bodyParser.text())
app.use(pubSubRouter({ publisher }))
```

To publish, you would use `publisher.publish` as described above. The `pubSubRouter` then sends signals to
subscribers using Server-Sent Events as the transport.

Clients are configured as follows:

```javascript
const { EventSourcePubSub } = require('pubsub-multi')
const Fetch22 = require('fetch-22') // Tiny lib that simplifies HTTP
const fetch = window.fetch.bind(window)
const baseUrl = 'http://host.com'
const fetch22 = new Fetch22({ baseUrl, fetch })
const eventSource = new EventSource(`${baseUrl}/pubsub`)

const publisher = new EventSourcePubSub({ fetch22, eventSource })
const subscriber = publisher.makeSubscriber()
```

To subscribe, you would use `subscriber.subscribe` as described above.
The `EventSourcePubSub` registers subscriptions on the server
with HTTP POST.
