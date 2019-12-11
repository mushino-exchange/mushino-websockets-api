# Mushino Websockets API

To use the API, you may connect your program to the following endpoint:

**wss://wss.mushino.com**

Or, if you just want to play around on the testnet:

**wss://wss.testnet.mushino.com**

Once connected, you can subscribe to a specific trading pair (such as the BTC perpetual future).

Then you'll receive all updates about that trading pair automatically.

Optionally, you may also supply an API key. Then you'll receive all personal updates about your account as well (such as orders, trades and withdrawals).

Here's an example in Python:

```bash
  python3.6 -m pip install "websockets"
```

```python
#!/usr/bin/env python
import asyncio
import websockets
import json

API_KEY = "YOUR_API_KEY" # Leave this out if you just want public data

async def hello():
    uri = "wss://wss.testnet.mushino.com" # Connect to testnet
    async with websockets.connect(uri) as websocket:

        # Let's receive updates about the BTC/USD perpetual future!
        await websocket.send(json.dumps({"op": "sub_pair", "content": "BTC_USD_PERP"}))

        #  OPTIONAL: Let's also authenticate, so we can receive personal updates about our trades and orders
        await websocket.send(json.dumps({"op": "auth_api", "content": API_KEY}))

        async for message in websocket:
            parsed_message = json.loads(message) # Yay, we got a message! Let's parse it!
            category = parsed_message['category'] # Can be one of the categories listed at the bottom of this document
            result = parsed_message['pair'] # Is the message related to a specific trading pair?
            msg = parsed_message['msg'] # In case there is an error, this will contain the error message
            print(parsed_message)

asyncio.get_event_loop().run_until_complete(hello())
```

Using **Javascript**? Check out the [Javascript example](#usage-examples) instead.

## Contents

[Subscribing](#subscribing)

[Receiving messages](#receiving-messages)

[Authenticating](#authenticating)

[Sending messages](#sending-messages)

[Keeping the connection alive](#keeping-the-connection-alive)

[Rate Limits](#rate-limits)

[Trading Pairs](#trading-pairs)

[Usage examples](#usage-examples)

[Message examples](#message-examples)

[Handling a message](#handling-a-message)

[Categories](#categories)

## Subscribing

To subscribe to a trading pair, you must send a JSON message in the following format:

```
   {
     op: "sub_pair",
     content: "BTC_USD_PERP"
   }
```

This message subscribes to all updates about the **BTC/USD perpetual future** (BTC_USD_PERP).

Once you have subscribed to a specific trading pair, you will automatically receive all updates about that pair (including order book changes, ticker changes, candlestick changes, trades and liquidations).

To subscribe only to ticker updates:

```
   {
     op: "sub",
     content: "BTC_USD_PERP:ticker"
   }
```

To unsubscribe from ticker updates:

```
   {
     op: "unsub",
     content: "BTC_USD_PERP:ticker"
   }
```

To unsubscribe from order book updates:

```
   {
     op: "unsub",
     content: "BTC_USD_PERP:depth"
   }
```

You may replace **BTC_USD_PERP** with a trading pair of your interest. For a list of valid trading pairs, see [Trading Pairs](#trading-pairs).

You may also replace **ticker** with a category of your interest. For a list of valid categories, see [Categories](#categories).

Getting the list of current subscriptions:

```
   {
     op: "subscriptions",
     content: "empty"
   }
```

Example result:

```
{
  code: 200,
  msg: 'subscriptions',
  result: [
    'BTC_USD_PERP:liquidation',
    'BTC_USD_PERP:index',
    'BTC_USD_PERP:ohlcv',
    'BTC_USD_PERP:match',
    'BTC_USD_PERP:trades',
    'BTC_USD_PERP:depth',
    'BTC_USD_PERP:ticker'
  ],
  pair: null,
  category: 'subscriptions'
}
```

## Receiving messages

Once you are subscribed to a trading pair, you will start receiving messages.

Messages are returned in JSON format. Example:

```
   {
     code: 200,
     msg: "Order book changed!",
     category: "depth",
     pair: "BTC_USD_PERP",
     result: {
       bids: [{qty: 20, price: 90}],
       asks: [{qty: 10, price: 100}]
     }
   }
```

The **category** and **result** properties are essential here.

The **category** can be any of the following:

* **depth** (Order book change)
* **ticker** (Ticker change)
* **index** (Mushino Index and Mark Price change)
* **ohlcv** (New candle)
* **match** (Orders were matched)
* **trades** (New trades)
* **liquidation** (New liquidation)
* **order_added** (Your order was accepted)
* **order_cancelled** (Your order was cancelled)
* **order_triggered** (Your stop order was triggered)
* **order_partially_completed** (Your order was partially filled)
* **order_completed** (Your order was completely filled)
* **trigger_added** (Your stop order was accepted)
* **trigger_cancelled** (Your stop order was cancelled)
* **trigger_failed** (Your stop order failed)
* **deposit_added** (Your deposit was detected)
* **deposit_completed** (Your deposit was credited)
* **withdrawal_added** (Your withdrawal was accepted)
* **withdrawal_completed** (Your withdrawal was completed)
* **position_updated** (Your position was updated)
* **position_closed** (Your position was closed)

The **result** property contains a JSON object with the data of the message. This may be your position (in the case of **position_updated**), or a bunch of prices (in the case of **ticker**).

## Authenticating

To authenticate, you must send a JSON message in the following format, replacing *<YOUR_API_KEY>* with your own API key:

```
   {
     op: "auth_api",
     content: "<YOUR_API_KEY>"
   }
```

Once you are authenticated, you will automatically receive updates about the following categories:

* order_added
* order_cancelled
* order_triggered
* order_partially_completed
* order_completed
* trigger_added
* trigger_cancelled
* trigger_failed
* deposit_added
* deposit_completed
* withdrawal_added
* withdrawal_completed
* position_updated
* position_closed

## Sending messages

When sending a message, you must use the following format:

```
   {
     op: "sub",
     content: "BTC_USD_PERP:ticker"
   }
```

**op** can be any of the following:

* **sub_pair** (Subscribe to all categories related to a specific trading pair)
* **sub** (Subscribe to a specific category, for a specific trading pair)
* **unsub** (Unsubscribe from a specific category)
* **unsub_all** (Unsubscribe from all categories)
* **unsub_pair** (Unsubscribe from all categories related to a specific trading pair)
* **subscriptions** (Get the list of categories that you are currently subscribed to)
* **auth_api** (Authenticate, using API key)
* **status** (Get the status of your connection)

When **op** is **sub_pair**, *content* must contain a valid trading pair, such as **BTC_USD_PERP**. For a list of valid trading pairs, see [Trading Pairs](#trading-pairs).

When **op** is **auth_api**, *content* must contain a valid API key.

When **op** is **sub**, **content** must contain a valid trading pair, concatenated by a category. Example: **BTC_USD_PERP:ticker**.

When **op** is something else, *content* can be anything, as long as it's not empty.

## Keeping the connection alive

To keep the connection alive, it is good practice to periodically ping the server.

You can do so by sending a status message:

```
   {
     op: "status",
     content: "empty"
   }
```

If you send this once every 5 seconds, you should be good.

Note: You*must always be either subscribed to something, **or** authenticated. If you are not subscribed to anything, and you have not authenticated, your connection is considered inactive, and you will be disconnected after 30 seconds.

## Rate Limits

You can be subscribed to a maximum of 10 **public** categories per client, and an unlimited amount of **private** categories.

If you must subscribe to more than 10 public categories, it is recommended that you use multiple connections.

You can maintain a maximum of 10 concurrent connections per account and per IP.

You can send a maximum of 20 messages per seconds per account and per IP.

## Trading pairs

Trading pairs must be specified with their programmatic name.

For the BTC/USD perpetual future, that is BTC_USD_PERP.

For the BTC/ALTS perpetual future, that is BTC_ALTS_PERP.

The programmatic name can be retrieved from the contract specification on Mushino.com, or through https://api.testnet.mushino.com/pairs.

Examples:

**BTC_USD_PERP**,
**ETH_USD_PERP**,
**USDT_ALTS_PERP**,
**USDT_SHIT_PERP**,
**BTC_ALTS_PERP**
**BTC_SHIT_PERP**
**BTC_USDT**
**ETH_USDT**

## Message Examples

Subscribing to trade updates for the Ethereum perpetual future:

```
   {
     op: "sub",
     content: "ETH_USD_PERP:trades"
   }
```

Unsubscribing from liquidation updates for the BTC/ALTS perpetual future:


```
   {
     op: "unsub",
     content: "BTC_ALTS_PERP:liquidation"
   }
```

Unsubscribing from everything:


```
   {
     op: "unsub_all",
     content: "empty"
   }
```

Getting the list of current subscriptions:

```
   {
     op: "subscriptions",
     content: "empty"
   }
```

Unsubscribing from everything related to the BTC/USD perpetual future:

```
   {
     op: "unsub_pair",
     content: "BTC_USD_PERP"
   }
```

## Usage Examples

**Javascript**:

```bash
  npm install --save ws
```

```javascript
const ws = require('ws');
const socket = new ws('wss://wss.testnet.mushino.com'); // Connect to testnet
const API_KEY = 'YOUR_API_KEY';

socket.on('open', () => {
    console.info('Yay, we are connected!');
    // Lets' receive updates about the BTC/USD perpetual!
    socket.send(JSON.stringify({op: 'sub_pair', content: 'BTC_USD_PERP'}));
    // OPTIONAL: Let's also receive personal updates about our account, orders and trades!
    socket.send(JSON.stringify({op: 'auth_api', content: API_KEY}));
    // Let's also send a ping every 5 seconds (helps keep the connection open)!
    setInterval(() => socket.send(JSON.stringify({op: 'status', content: 'ok'})), 5000);
    socket.on('message', function(json){
      var message = JSON.parse(json); // Yay, we got a message! Let's parse it!
      var pair = message.pair; // Is the message related to a specific trading pair?
      var category = message.category; // Can be one of the categories listed at the bottom of this document
      var msg = message.msg; // In case there is an error, this will contain the error message
      console.info("Received message: ", message);
    });
});

setInterval(() => true, 10e6); // Keep the program running for a long time...
```

**Python (3.6):**

```bash
  python3.6 -m pip install "websockets"
```

```python
#!/usr/bin/env python
import asyncio
import websockets
import json

API_KEY = "YOUR_API_KEY" # Leave this out if you just want public data

async def hello():
    uri = "wss://wss.testnet.mushino.com" # Connect to testnet
    async with websockets.connect(uri) as websocket:

        # Let's receive updates about the BTC/USD perpetual future!
        await websocket.send(json.dumps({"op": "sub_pair", "content": "BTC_USD_PERP"}))

        #  OPTIONAL: Let's also authenticate, so we can receive personal updates about our trades and orders
        await websocket.send(json.dumps({"op": "auth_api", "content": API_KEY}))

        async for message in websocket:
            parsed_message = json.loads(message) # Yay, we got a message! Let's parse it!
            category = parsed_message['category'] # Can be one of the categories listed at the bottom of this document
            result = parsed_message['pair'] # Is the message related to a specific trading pair?
            msg = parsed_message['msg'] # In case there is an error, this will contain the error message
            print(parsed_message)

asyncio.get_event_loop().run_until_complete(hello())

```

## Handling a message

When receiving a message, you would usually want to do something.

This is usually based on the category of the message.

Here is an example in Javascript of simply pretty printing the message, based on the category it contains:

```javascript

socket.on('message', function(json){
  var message = JSON.parse(json);
  return handle_message(message);
});

function handle_message(message) {
  var pair = message.pair;
  switch(message.category) {
    case 'error': return console.error("An error occured: ", message.msg);
    case 'subscribed': return console.info("Subscribed to: ", message.result);
    case 'unsubscribed': return console.info("Unsubscribed from: ", message.result);
    case 'authenticated': return console.info("Succesfully authenticated!");
    case 'ticker': return console.info("New ticker: ", message.result);
    case 'depth': return console.info("Order book changed: ", message.result);
    case 'ohlcv': return console.info("New candle: ", message.result.ohlcv);
    case 'liquidation': return console.info("New liquidation: ", message.result);
    case 'match': return console.info("Orders were matched: ", message.result);
    case 'trades': return console.info("New trades: ", message.result);
    case 'index': return console.info("New mark price: ", message.result.mark);
    case 'order_added': return console.info("Your order was accepted: ", message.result);
    case 'order_cancelled': return console.info("Your order was cancelled: ", message.result);
    case 'order_triggered': return console.info("Your stop was triggered: ", message.result);
    case 'order_partially_completed': return console.info("Your order was filled partially: ", message.result);
    case 'order_completed': return console.info("Your order was filled completely: ", message.result);
    case 'trigger_added': return console.info("Your stop was accepted: ", message.result);
    case 'trigger_cancelled': return console.info("Your stop was cancelled: ", message.result);
    case 'trigger_failed': return console.info("Your stop failed: ", message.result);
    case 'deposit_added': return console.info("Your deposit was detected: ", message.result);
    case 'withdrawal_added': return console.info("Your withdrawal was accepted: ", message.result);
    case 'deposit_completed': return console.info("Your deposit was credited: ", message.result);
    case 'withdrawal_completed': return console.info("Your withdrawal was completed: ", message.result);
    case 'ok': return true;
    default: return console.info("Unknown category: ", message);
  }
}

```

## Categories

* **depth** (Order book change) (Public)
* **ticker** (Ticker change) (Public)
* **index** (Mushino Index and Mark Price change) (Public)
* **ohlcv** (New candle) (Public)
* **match** (Orders were matched) (Public)
* **trades** (New trades) (Public)
* **liquidation** (New liquidation) (Public)
* **order_added** (Your order was accepted) (Requires authentication)
* **order_cancelled** (Your order was cancelled) (Requires authentication)
* **order_triggered** (Your stop order was triggered) (Requires authentication)
* **order_partially_completed** (Your order was partially filled) (Requires authentication)
* **order_completed** (Your order was completely filled) (Requires authentication)
* **trigger_added** (Your stop order was accepted) (Requires authentication)
* **trigger_cancelled** (Your stop order was cancelled) (Requires authentication)
* **trigger_failed** (Your stop order failed) (Requires authentication)
* **deposit_added** (Your deposit was detected) (Requires authentication)
* **deposit_completed** (Your deposit was credited) (Requires authentication)
* **withdrawal_added** (Your withdrawal was accepted) (Requires authentication)
* **withdrawal_completed** (Your withdrawal was completed) (Requires authentication)
* **position_updated** (Your position was updated) (Requires authentication)
* **position_closed** (Your position was closed) (Requires authentication)
