# Market Data Web Socket API (2018-08-01)
## General
The base endpoint is: **wss://api-tokenswatch.com:9007**

After connection you will immediately receive the initial message which contains symbols for all available exchanges:

```javascript
["symbols", {
    "binance": ["LTC_BTC", "ETH_BTC", ...],
    "bittrex": ["LTC_BTC", "ETH_BTC", ...],
    ...
}]
```
There are two data access methods - loading and subscribing.

You should send `load-` request when you need to retrieve some portion of market data once.

If you want to receive every data update (continuously) you should send `subs-` request.
To stop receiving data updates just send `unsubs-` request.

In case of incorrect request you will receive the error message:

```javascript
["error", {
    "request": "qwerty",
    "reason": "invalid message format"
}]
```
## Quotes

We provide real-time Depth-of-Market (Level2) for many exchanges as well as quotes history.

You should send `load-dom` request first time to get current state of Depth-of-Market:

```javascript
["load-dom", {
    "exchange": "binance", // name of exchanges (from "symbols" message)
    "symbol": "LTC_BTC"    // name of ticker (from "symbols" message)
}]
```

You will receive the following response:

```javascript
["dom", {
    "exchange": "binance",  // name of exchanges (from "symbols" message)
    "symbol": "LTC_BTC",    // name of ticker (from "symbols" message)
    "data": [
        [
            1532975165509,  // time (in milliseconds)
            6543.43,        // price
            0.13,           // quantity
            0,              // orders count (NOT supported by many exchanges)
            0               // 0 - Bid, 1 - Ask, 2 - Undefined
        ],
        ...
    ]
}]
```

After that you can send `subs-quotes` request to receive update continuously:

```javascript
["subs-quotes", {
    "exchange": "binance", // name of exchanges (from "symbols" message)
    "symbol": "LTC_BTC"    // name of ticker (from "symbols" message)
}]
```

You will receive the following response (continuously until you call `unsubs-quotes`):

```javascript
["quote", {
    "exchange": "binance",  // name of exchanges (from "symbols" message)
    "symbol": "LTC_BTC",    // name of ticker (from "symbols" message)
    "data": [
        1532975165509,      // time (in milliseconds)
        6543.43,            // price
        0.13,               // quantity
        0,                  // orders count (NOT supported by many exchanges)
        0                   // 0 - Bid, 1 - Ask, 2 - Undefined
    ]
}]
```

To stop receiving quote updates you should send `unsubs-quotes` message:

```javascript
["unsubs-quotes", {
    "exchange": "binance", // name of exchanges (from "symbols" message)
    "symbol": "LTC_BTC"    // name of ticker (from "symbols" message)
}]
```

To get quotes history (sequence of quotes for some time period in the past) you should send `load-quotes` request:

```javascript
["load-quotes", {
    "exchange": "binance", // name of exchanges (from "symbols" message)
    "symbol": "LTC_BTC",   // name of ticker (from "symbols" message)
    "from": 1532975165509, // start time (in milliseconds)
    "to": 1532977000000    // end time (in milliseconds)
}]
```

You will receive `quotes` response once:

```javascript
["quotes", {
    "exchange": "binance", // name of exchanges (from "symbols" message)
    "symbol": "LTC_BTC",   // name of ticker (from "symbols" message)
    "data": [
        [
            1532975165509, // time (in milliseconds)
            6543.43,       // price
            0.13,          // quantity
            0,             // orders count (NOT supported by many exchanges)
            0              // 0 - Bid, 1 - Ask, 2 - Undefined
        ],
        ...
    ]
}]
```
## Trades

If you want to get history of trades for some time period in the past you should send `load-trades` request:

```javascript
["load-trades", {
    "exchange": "binance", // name of exchanges (from "symbols" message)
    "symbol": "LTC_BTC",   // name of ticker (from "symbols" message)
    "from": 1532975165509, // start time (in milliseconds)
    "to": 1532977000000    // end time (in milliseconds)
}]
```

You will receive `trades` response once:

```javascript
["trades", {
    "exchange": "binance", // name of exchanges (from "symbols" message)
    "symbol": "LTC_BTC",   // name of ticker (from "symbols" message)
    "data": [
        [
            101123819,     // unique ID
            1532975165398, // time (in milliseconds)
            6552.98,       // price
            1.4,           // quantity
            1              // 0 - Bought, 1 - Sold, 2 - Undefined
        ],
        ...
    ]
}]
```

If you want to receive all new trades continuously you should send `subs-quotes` request:

```javascript
["subs-trades", {
    "exchange": "binance", // name of exchanges (from "symbols" message)
    "symbol": "LTC_BTC"    // name of ticker (from "symbols" message)
}]
```

You will receive the following response (continuously until you call `unsubs-trades`):

```javascript
["trade", {
    "exchange": "binance", // name of exchanges (from "symbols" message)
    "symbol": "LTC_BTC",   // name of ticker (from "symbols" message)
    "data": [
        101123819,         // unique ID
        1532975165398,     // time (in milliseconds)
        6552.98,           // price
        1.4,               // quantity
        1                  // 0 - Bought, 1 - Sold, 2 - Undefined
    ]
}]
```

To stop receiving trade updates you should send `unsubs-trades` message:

```javascript
["unsubs-trades", {
    "exchange": "binance", // name of exchanges (from "symbols" message)
    "symbol": "LTC_BTC"    // name of ticker (from "symbols" message)
}]
```

## Candles

We provide candle real-time updates as well as history for the following time frames:

```javascript
"1min"
"5min"
"15min"
"30min"
"1hr"
"2hr"
"4hr"
"6hr"
"12hr"
"1day"
"1week"
```

If you want to get candle history for some period in the past you should send `load-candles` request:

```javascript
["load-candles", {
    "exchange": "binance", // name of exchanges (from "symbols" message)
    "symbol": "LTC_BTC",   // name of ticker (from "symbols" message)
    "timeframe": "15min",  // time frame
    "from": 1532975165509, // start time (in milliseconds)
    "to": 1532977000000    // end time (in milliseconds)
}]
```

You will receive `candles` response once:

```javascript
["candles", {
    "exchange": "binance", // name of exchanges (from "symbols" message)
    "symbol": "LTC_BTC",   // name of ticker (from "symbols" message)
    "timeframe": "15min",  // time frame
    "data": [
        [
            1532975165398, // time (in milliseconds)
            6552.98,       // open price
            6554.12,       // close price
            6551.42,       // low price
            6557.08,       // high price
            13328.6        // volume
        ],
        ...
    ]
}]
```

If you want to receive all new candles continuously you should send `subs-candles` request:

```javascript
["subs-candles", {
    "exchange": "binance", // name of exchanges (from "symbols" message)
    "symbol": "LTC_BTC",   // name of ticker (from "symbols" message)
    "timeframe": "15min"   // time frame
}]
```

You will receive the following response (continuously until you call `unsubs-candles`):

```javascript
["candle", {
    "exchange": "binance", // name of exchanges (from "symbols" message)
    "symbol": "LTC_BTC",   // name of ticker (from "symbols" message)
    "timeframe": "15min",  // time frame
    "data": [
        1532975165398,     // time (in milliseconds)
        6552.98,           // open price
        6554.12,           // close price
        6551.42,           // low price
        6557.08,           // high price
        13328.6            // volume
    ]
}]
```

To stop receiving trade updates you should send `unsubs-candles` message:

```javascript
["unsubs-candles", {
    "exchange": "binance", // name of exchanges (from "symbols" message)
    "symbol": "LTC_BTC",   // name of ticker (from "symbols" message)
    "timeframe": "15min"   // time frame
]
```
