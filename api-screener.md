# Screener WebSocket API (2018-08-01)
## General
The Screener API allows you to create and use the basic primitives (watchlists, filters, signals).
You can find more information about it in [this article](http://medium.com/).

The base endpoint is: **wss://api-tokenswatch.com:9008**

In case of incorrect request you will receive the error message:

```javascript
["error", {
    "request": "qwerty",
    "reason": "invalid message format"
}]
```

The Screener API is only available for registered users so it's necessarily to provide your ID:
</br>**wss://api-tokenswatch.com:9008/YOUR_ID_NUMBER**

If your ID is correct you will immediately receive the initial message:

```javascript
["init", {
    
    // exchanges which are avaliable for screening
    "exchanges": {
        "bittrex": { // the name of exchange
            // link to the main page
            "url-main": "https://bittrex.com",
            // link to the view page with pattern (e.g. BTC-LTC, USDT-LTC)
            "url-view": "https://bittrex.com/Market/Index?MarketName=%QUOTE%-%BASE%",
            // array of available symbols
            "symbols": [
                "LTC_BTC",
                "BTC_USD",
                ...
            ]
        },
        ...
    },
    
    // currencies which you can choose to show
    "currencies": [
        "USD",
        "EUR",
        "CNY",
        ...
    ],
    
    // all available columns (of the screener table) 
    "columns": {
        // name    type    
        "symbol": "text", // simple text
        "price": "currency", // e.g. 123.43 BTC
        "1h-%": "percent",   // e.g. 43.21
        ...
    },
    
    // all avaliable routines which you can add to your watchlists
    "routines": {
        // the routine is just a common name for filer / signal
        "routine-name" : {
            // the routine contains of parameters of the following types:
            "param-name": {
                "type": "bool",
                "default": true
            },
            "param-name": {
                "type": "enum",
                "control": "combo"|"radio",
                "values": ["zero", "one", "two"],
                "default": 1
            },
            "param-name": {
                "type": "int",
                "control": "slider"|"spin",
                "min": -23,
                "max": 320,
                "default": 15
            },
            "param-name": {
                "type": "fract",
                "control": "slider"|"spin",
                "min": "0.21",
                "max": "19203.32",
                "default": "223.12"
            },
            "param-name": {
                "type": "time",
                "control": "timebox"|"spin",
                "default": 1523109778655
            },
            "param-name": {
                "type": "string",
                "control": "line"|"field",
                "max-lenght": 100,
                "lines-count": 5,
                "default": "abc"
            },
            "param-name": {
                "type": "output",
                "default": ["absolute"|"percent", -10.5, 10.5]
            },
            "param-name": {
                "type": "label",
                "text": "some text"
            },
            "param-name": {
                "type": "image",
                "url": "http://mydomain.com/image01.png",
                "width": 128,
                "height": 128
            },
            "param-name": {
                "type": "images",
                "urls": ["http://123.com/01.png", "http://123.com/02.png"],
                "width": 128,
                "height": 128
            }
        },
        // example of some filter
        "Symbol White List": {
            "symbols": {
                "type": "string",
                "control": "field",
                "max-lenght": 4096,
                "lines-count": 10,
                "default": "BTC,LTC,ETH"
            },
        },
        // example of some signal
        "Mail Signal": {
            "name": {
                "type": "string",
                "control": "line",
                "max-lenght": 32,
                "lines-count": 1,
                "default": "my-signal"
            },
            "per-trade-update": {
                "type": "bool",
                "default": true
            },
            "update-interval": {
                "type": "int",
                "control": "spin",
                "min": 1,
                "max": 600000,
                "default": 30
            },
            "recipient": {
                "type": "string",
                "control": "line",
                "max-lenght": 32,
                "lines-count": 1,
                "default": ""
            }
        },
        ...
    },
    
    // all your watchlists
    "watchlists": {
        "watchlist-name": {
            // watchlist parameters
            "params": {
                // the maximum lifetime (in seconds) which ticker could have to be shown
                "max-lifetime": 3,
                // the interval in seconds between watchlist updates
                "refresh-interval": 2
            },
            // a list of routines (filters / signals)
            "routines": [
                {"Symbol White List": { // routine name
                    // parameters
                    "symbols": "ADA,NEO"
                }},
                {"Mail Signal": { // routine name
                    // parameters
                    "name": "BuyAndHold Signals",
                    "per-trade-update": true,
                    "update-interval": 10,
                    "recipient": "me@some.com"
                }},
                ...
            ]
        },
        ...
    },
    
    // the subscription info (optional)
    "subscription": {
        // the name of watchlist to which you are subscribed
        "watchlist": "watchlist-name",
        // the interval in seconds between result sends
        "refresh-interval": 2,
        // a list of columns which will be sent to you
        "columns": ["symbol", "price", "volume"],
        // a list of exchange which will be used for search
        "exchanges": ["binance", "bittrex"],
        // the result table will be sorted by "sortColumn" using "sortOrder"
        "sortOrder": "des"|"asc",
        "sortColumn": "symbol",
        // the ticker will be used in the search only if its "searchColumn" includes "searchText" 
        "searchText": "", // if you don't want to use it, just set empty string
        "searchColumn": "symbol",
        // currency to which all the price values will be converted (empty string if conversion is disabled)
        "targetCurrency": "USD",
        // result table will contains "rowsCount" starting from "rowsOffset"
        "rowsOffset": 100,
        "rowsCount": 25
    },
    
    // a list of your favourite coins
    "favourites": [
        // symbol    exchange
        ["ADA_BTC", "bittrex"],
        ["LTC_BTC", "bittrex"],
        ["LTC_BTC", "binance"],
        ...
    ] 
}]
```

The initial message contains the information about screener's routines (filters & signals) and abilities as well as your personal data (watchlists, subscription, favourites).
 
Now you can start to work with screener by sending another messages.

## Watchlists

Lets create a new watchlist by sending `watchlist-add` message:

```javascript
["watchlist-add", {
    "name": "myWatchlist",
    "params": {
        // the maximum lifetime (in seconds) which ticker could have to be shown
        "max-lifetime": 10,
        // the interval in seconds between watchlist updates
        "refresh-interval": 2
    }
}]
```

If you want to rename the watchlist you should send `watchlist-rename` message:

```javascript
["watchlist-rename", {
    "name": "myWatchlist",
    "new-name": "theBestWatchlist",
}]
```

You can also change parameters of the existing watchlist by sending `watchlist-change` message:

```javascript
["watchlist-change", {
    "name": "myWatchlist",
    "params": {
        "max-lifetime": 110,
        "refresh-interval": 5
    }
}]
```

If you want to remove your watchlist you should just send `watchlist-remove` message:

```javascript
["watchlist-remove", {
    "name": "myWatchlist"
}]
```

## Filters and Signals

A watchlist is a stack of routines of different types (you can find all that types in initial message "routines" part).
A routine in stack is defined by order where [0] is the top element. So we can reference routine just by that order (integer number).  

Lets add some routine ("Parabolic SAR" indicator filter) by sending `routine-add` message:

```javascript
["routine-add", {
    // the name of watchlist you want to add the routine
    "watchlist": "myWatchlist",
    // the name of routine (taken from initial message "routines")
    "routine": "Parabolic SAR",
    // the order of execution (0 - first)
    "order": 0,
    // routine's parameters (see initial message "routines")
    "params": {
        "time-frame": "2_hours",
        "acceleration-factor": 0.032,
        "acceleration-max": 0.8,
        "output": ["absolute", 0.01, 0.5]
    }
}]
```

If you want to change the order of the routine you should send `routine-reorder` message:

```javascript
["routine-reorder", {
    "watchlist": "myWatchlist",
    "order": 0,
    "new-order": 1
}]
```

You can also change routine's parameters by sending `routine-change` message: 

```javascript
["routine-change", {
    "watchlist": "myWatchlist",
    "order": 0,
    "params": {
        "time-frame": "2_hours",
        "acceleration-factor": 0.045,
        "acceleration-max": 0.9,
        "output": ["absolute", 0.02, 0.4]
    }
}]
```

If you want to remove the routine just send `routine-remove` message:

```javascript
["routine-remove", {
    "watchlist": "myWatchlist",
    "order": 0
}]
```

At this point you might have all the watchlist (full of filters and signals) created and want to receive the results.
You can do it using subscription (result of filters execution, which you normally see in the table on the screen) or API-signals.

## Subscription

You can subscribe to watchlist result by sending `watchlist-subscribe` message:

```javascript
["watchlist-subscribe", {
    // the name of watchlist to which you are subscribed
    "name": "watchlist-name",
    // the interval in seconds between watchlist updates
    "refresh-interval": 2,
    // what columns you want to have in the table result
    // (taken from initial message "columns") 
    "columns": ["symbol", "price", "volume"],
    // a list of exchange which will be used for search 
    "sortOrder": "des"|"asc",
    "sortColumn": "symbol",
    // the ticker will be used in the search only if its "searchColumn" includes "searchText" 
    "searchText": "", // if you don't want to use it, just set empty string
    "searchColumn": "symbol",
    // currency to which all the price values will be converted (set empty string to disable conversion)
    "targetCurrency": "USD",
    // result table will contains "rowsCount" starting from "rowsOffset"
    "rowsOffset": 100,
    "rowsCount": 25
}]
```

If all is correct you will immediately receive result snapshot:

```javascript
["watchlist-snapshot", {
    // the name of watchlist
    "name": "watchlist0",
    // UTC timestamp (in millisecconds)
    "timestamp": 1533234905423,
    // result table rows count
    "count": 12,
    // result table rows array
    "rows": [
        // data from all columns which were specified in "watchlist-subscribe" message
        ["BTC-USD", "Bitfinex", 7800.54, -0.23],
        ["LTC-USD", "Bitfinex", 892.21, 13.3],
        ...
    ]
}]
```

Then you will be receiving result updates continuously:

```javascript
["watchlist-update", {
    // the name of watchlist
    "name": "watchlist0",
    // UTC timestamp (in millisecconds)
    "timestamp": 1533234905423,
    // result table rows array
    "rows": [
        // the first row element defines row's state:
        // 0 - unchanged
        // 1 - changed
        // 2 - added
        // 3 - removed
        [0]
        [1, "BTC-USD", "Bitfinex", 7800.54, -0.23],
        [2, "LTC-USD", "Bitfinex", 892.21, 13.3],
        [3],
        ...
    ]
}]
```

If you want to unsubscribe (stop receiving updates) from the watchlist you should send `watchlist-unsubscribe` message:

```javascript
["watchlist-unsubscribe"]
```

## API-signals

Another way to receive your screening result is API-signals (a type of signal which send your results using WebSocket).

First of all you need to add API-signal to your watchlist.
Then you should connect to the API-signal endpoint: **wss://api-tokenswatch.com:9009**.
If all is correct you will be continuously receiving `signal-emit` messages:

```javascript
["signal-emit", {
    // the name of watchlist
    "watchlist": "watchlist0",
    // the order of the signal in the watchlist
    "signal-order": 4,
    // UTC timestamp (in millisecconds)
    "timestamp": 1533234905423,
    // result table rows count
    "count": 12,
    // result table rows array
    "rows": [
        // data from all columns which were specified in API-signal parameters
        ["BTC-USD", "Bitfinex", 7800.54, -0.23],
        ["LTC-USD", "Bitfinex", 892.21, 13.3],
        ...
    ]
]
```

You don't need to have screener web-page opened, all API-signals are always working in background on our servers.

To stop receiving `signal-emit` messages you need to remove API-signal from your watchlist or just disable it (in signal parameters dialog).

## Favourites

One more feature is the favourites list. If you want to remember some coins you can just add it to the list by sending `favourites-add` message:

```javascript
["favourites-add", {
    "order": 0,            // position in the list (0 - top)
    "exchange": "binance", // taken from initial message "exchanges".name
    "symbol": "LTC_BTC"    // taken from initial message "exchanges".symbols
}]
```

To remove the element from the list you should send `favourites-remove` message:

```javascript
["favourites-remove", {
    "order": 0
}]
```

To change the order of the element in the list you should send `favourites-reorder` message:

```javascript
["favourites-reorder", {
    "order": 0,
    "new-order": 1
}]
```

To get all the elements from the favourites list you should send `favourites-get` message:

```javascript
["favourites-get"]
```

You will immediately receive the result message:

```javascript
["favourites", [
    // symbol    exchange
    ["ADA_BTC", "bittrex"],
    ["LTC_BTC", "bittrex"],
    ["LTC_BTC", "binance"],
    ...
]]
```
