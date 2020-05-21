## Table of Contents
* [Phemex Public API](#publicapi)
  * [General Public API Information](#general)
* [REST API Standards](#restapi)
  * [HTTP Restful Response](#restresponse)
    * [HTTP Return Codes](#httpreturncodes)
    * [HTTP Restful Response Format](#responseformat)
    * [Restful Response Error Codes](#errorcode)
  * [HTTP REST Request Header](#httprestheader)
  * [API Rate Limits](#apiratelimits)
  * [Endpoint security type](#securitytype)
    * [Signature Example 1: HTTP GET Request](#signatureexample1)
    * [Singature Example 2: HTTP GET Request with multiple query string](#signatureexample2)
    * [Signature Example 3: HTTP POST Request](#signatureexample3)
  * [Request/Response field explained](#fieldexplained)
    * [Spot currency and Symbol](#spotCurrencySym)
    * [Common constants](#commconsts)
  * [REST API List](#restapilist)
    * [Market API List](#marketapilist)
      * [Query Product Information](#queryproductinfo)
    * [Market Data API List ](#mdapilist)
      * [Query Order Book](#queryorderbook)
      * [Query Recent Trades](#querytrades)
      * [Query 24 Hours Ticker](#query24hrsticker)
    * [Spot Trading Api List](#spotTradingApi)
      * [Place Order](#spotPlaceOrder)
      * [Amend Order](#spotAmendOrder)
      * [Cancel Order](#spotCancelOrder)
      * [Cancel All Order by Symbol](#spotCancelAll)
      * [Query Open Order by clOrdID or orderID](#spotQueryOpenOrder)
      * [List all Open Orders by Symbol](#spotListAllOpenOrder)
      * [Query Wallets](#spotQueryWallet)

* [Websocket API Standards](#wsapi)
  * [Session Management](#sessionmanagement)
  * [API Rate Limits](#wsapiratelimits)
  * [WebSocket API List](#wsapilist)
    * [Heartbeat](#heartbeat)
    * [API User Authentication](#apiuserauth)
    * [Subscribe OrderBook](#booksub)
    * [Unsubscribe OrderBook](#bookunsub)
    * [Subscribe Trade](#tradesub)
    * [Unsubscribe Trade](#tradeunsub)
    * [Subscribe Kline](#klinesub)
    * [Unsubscribe Kline](#klinesub)
    * [Subscribe Wallet-Order)](#wosub)
    * [Unsubscribe Wallet-Order](#wounsub)
    * [Subscribe 24 Hours Ticker](#tickersub)
    * [All Trading Symbols](#symbpricesub)

<a name="publicapi"/>

# Phemex Public API

<a name="general"/>

## General Public API Information

* Phemex provides HTTP Rest API for client to operate Orders, all endpoints return a JSON object.
* The Rest API base endpoint is: **https://api.phemex.com**, or for the testnet is:  **https://testnet-api.phemex.com** 
* Phemex provides WebSocket API for client to receive market data, order and position updates.
* The WebSocket API url is: **wss://phemex.com/ws**, or for the testnet is:  **wss://testnet.phemex.com/ws** 

<a name="restapi"/>

# REST API Standards

<a name="restresponse"/>

## 1 Restful API Response

<a name="httpreturncodes"/>

### 1.1 HTTP Return Codes

* HTTP `401` return code is used when unauthenticated
* HTTP `403` return code is used when lack of priviledge.
* HTTP `429` return code is used when breaking a request rate limit.
* HTTP `5XX` return codes are used for Phemex internal errors. Note: This doesn't means the operation failed, the execution status is **UNKNOWN** and could be Succeed.

<a name="responseformat"/>

### 1.2 Rest Response format

   * All restful API except ***starting*** with `/md` shares same response format.

```
{
    "code": <code>,
    "msg": <msg>,
    "data": <data>
}

```

| Field | Description | 
|-------|------|
| code | 0 means `success`, non-zero means `error`|
| msg  | when code is non-zero, it gives short error description |
| data | operation dependant |

<a name="errorcode"/>

### 1.3 Error codes

[Trading Error Codes](TradingErrorCode.md)

## 2. HTTP REST Request Header 

Every HTTP Rest Request must have the following Headers:
* x-phemex-access-token : This is ***API-KEY*** (id field) from Phemex site.
* x-phemex-request-expiry : This describes the Unix ***EPoch SECONDS*** to expire the request, normally it should be (Now() + 1 minute)
* x-phemex-request-signature : This is HMAC SHA256 signature of the http request. Secret is ***API Secret***, its formula is : HMacSha256( URL Path + QueryString + Expiry + body )


<a name="apiratelimits"/>

## 3. API Rate Limits

* Every Client has the API call rate limit as 500 per minute.
* Every HTTP Rest response will contain a `X-RateLimit-Remaining` header which has the remain request count in this round.
* When client gets HTTP RESPONSE Code 429 means it reached limit, `X-RateLimit-Retry-After` header means the seconds that client should wait before next round.


<a name="securitytype"/>

## 4. Endpoint security type

* Each API call must be signed and pass to server in HTTP header `x-phemex-request-signature`.
* Endpoints use `HMAC SHA256` signatures. The `HMAC SHA256 signature` is a keyed `HMAC SHA256` operation. Use your `apiSecret` as the key and the string `URL Path + QueryString + Expiry + body )` as the value for the HMAC operation.
* `apiSecret` = `Base64::urlDecode(API Secret)`
* The `signature` is **case sensitive**.

<a name="signatureexample1"/>

### 4.1 Signature Example 1: HTTP GET Request

* API REST Request URL: https://api.phemex.com/spot/wallets?currency=BTC 
   * Request Path: /spot/wallets
   * Request Query: currency=BTC
   * Request Body: <null>
   * Request Expiry: 1587552406
   * Signature: HMacSha256( /spot/wallets + currency=BTC + 1587552406 )

<a name="signatureexample2"/>

### 4.2 Singature Example 2: HTTP GET Request with multiple query string

* API REST Request URL: https://api.phemex.com/spot/orders/active?symbol=sBTCUSDT&orderID=bc2b8ff1-a73b-4673-aa5b-fda632285fcc 
    * Request Path: /spot/orders/active
    * Request Query: symbol=sBTCUSDT&orderID=bc2b8ff1-a73b-4673-aa5b-fda632285fcc 
    * Request Body: <null>
    * Request Expire: 1587552407
    * Signature: HMacSha256(/spot/orders/active + symbol=sBTCUSDT&orderID=bc2b8ff1-a73b-4673-aa5b-fda632285fcc + 1587552407)
    * signed string is `/spot/orders/activesymbol=sBTCUSDT&orderID=bc2b8ff1-a73b-4673-aa5b-fda632285fcc1587552407`

<a name="signatureexample3"/>

### 4.3 Signature Example 3: HTTP POST Request

* API REST Request URL: https://api.phemex.com/spot/orders
   * Request Path: /spot/orders
   * Request Query: <null>
   * Request Body: 
   ```
   {"symbol":"sBTCUSDT","clOrdID":"ece0187f-7e02-44b5-a778-404125f124fa","side":"Buy","qtyType":"ByBase","quoteQtyEv":"0","baseQtyEv":"100000","priceEp":"700000000","stopPxEp":"0","execInst":"","ordType":"Limit","timeInForce":"","text":""}
   ```
   * Request Expiry: 1587552407 
   * signed string is 
   ```
    /spot/orders1587552407{"symbol":"sBTCUSDT","clOrdID":"ece0187f-7e02-44b5-a778-404125f124fa","side":"Buy","qtyType":"ByBase","quoteQtyEv":"0","baseQtyEv":"100000","priceEp":"700000000","stopPxEp":"0","execInst":"","ordType":"Limit","timeInForce":"","text":""}

   ```

## 5. Request/response field explained


<a name="spotCurrencySym"/>

<a name="spotCurrency"/>

### Spot Currency and Symbols

* Spot Currency and its scale factor

| currency | value scale  factor | min value  |  max value |
|--------|---------------|------------|------------|
| BTC    | 8             | 1          | 5000000000000000000 |
| USDT   | 8             | 1          | 5000000000000000000 |
| ETH    | 8             | 1          | 5000000000000000000 |
| XRP    | 8             | 1          | 5000000000000000000 |
| LINK   | 8             | 1          | 5000000000000000000 |
| XTZ    | 8             | 1          | 5000000000000000000 |
| LTC    | 8             | 1          | 5000000000000000000 |

<a name="spotSymList"/>

* Spot symbol and its scale factor

|  symbol|  type|  baseTickSize|  baseTickSizeEv|  quoteTickSize|  quoteTickSizeEv|  baseQtyPrecision|  quoteQtyPrecision|  pricePrecision|  minOrderValue|  minOrderValueEv|  maxBaseOrderSize|  maxBaseOrderSizeEv|  maxOrderValue|  maxOrderValueEv|  defaultTakerFee|  defaultTakerFeeEr|  defaultMakerFee|  defaultMakerFeeEr|
|---------|---------|-------------|-----------|-----------------|------------|------------|-------------|------|--------|-------|------|--------|-------|------|------|-------|-------|--------|
|sBTCUSDT|Spot|0.000001BTC|100|0.01USDT|1000000|6|2|2|10USDT|1000000000|1000BTC|100000000000|5|000|000USDT|500000000000000|0.001|100000|0.001|100000|
|sETHUSDT|Spot|0.00001ETH|1000|0.01USDT|1000000|5|2|2|10USDT|1000000000|10000ETH|1000000000000|5|000|000USDT|500000000000000|0.001|100000|0.001|100000|
|sXRPUSDT|Spot|0.1XRP|10000000|0.00001USDT|1000|1|2|5|10USDT|1000000000|5|000|000XRP|500000000000000|5|000|000USDT|500000000000000|0.001|100000|0.001|100000|
|sLINKUSDT|Spot|0.01LINK|1000000|0.0001USDT|10000|2|2|4|10USDT|1000000000|5|000|000LINK|500000000000000|5|000|000USDT|500000000000000|0.001|100000|0.001|100000|
|sXTZUSDT|Spot|0.01XTZ|1000000|0.0001USDT|10000|2|2|4|10USDT|1000000000|2|000|000XTZ|200000000000000|5|000|000USDT|500000000000000|0.001|100000|0.001|100000|
|sLTCUSDT|Spot|0.00001LTC|1000|0.001USDT|100000|5|2|3|10USDT|1000000000|100|000LTC|10000000000000|5|000|000USDT|500000000000000|0.001|100000|0.001|100000|


<a name="commconsts"/>

### 5.2 Common constants

* order type

| order type | description |
|-----------|-------------|
| Limit | -- |
| Market | -- |
| Stop | -- |
| StopLimit | -- |
| MarketIfTouched | -- |
| LimitIfTouched | -- |
| MarketAsLimit | -- |
| StopAsLimit | -- |
| MarketIfTouchedAsLimit | -- |


* order Status

| order status | description | 
|------------|-------------|
| Untriggered | Conditional order waiting to be triggered |
| Triggered | Conditional order being triggered|
| Rejected | Order rejected |
| New | Order placed in cross engine |
| PartiallyFilled | Order partially filled |
| Filled | Order fully filled |
| Canceled | Order canceled |

* TimeInForce

| timeInForce | description |
|------------|-------------|
| GoodTillCancel | -- |
| PostOnly | -- |
| ImmediateOrCancel | -- |
| FillOrKill | -- |


* Trigger source

| trigger | description |
|------------|-------------|
| ByLastPrice | trigger by last price |


<a name="restapilist"/>

## 6. REST API List

<a name="marketapilist"/>

### 6.1 Market API List 

<a name="queryproductinfo"/>

#### 6.1.1 Query Product Information 

* Request：
```json
GET /exchange/public/cfg/v2/products

```



<a name="mdapilist"/>

### 6.3 Market Data API List

<a name="queryorderbook"/>

#### 6.3.1 Query Order Book

* Request：
```json
GET /md/orderbook?symbol=<symbol>&id=<id>
```

| Field       | Type   | Description                                | Possible values |
|-------------|--------|--------------------------------------------|--------------|
| symbol      | String | symbol name                                | [Trading symbols](#symbpricesub) |
| id          | Integer| Optional. Request id                       |              |

* Response:
```
{
  "error": null,
  "id": 0,
  "result": {
    "book": {
      "asks": [
        [
          <priceEp>,
          <size>
        ],
        ...
        ...
        ...
      ],
      "bids": [
        [
          <priceEp>,
          <size>
        ],
        ...
        ...
        ...
      ],
    ]
    },
    "depth": 30,
    "sequence": <sequence>,
    "timestamp": <timestamp>,
    "symbol": "<symbol>",
    "type": "snapshot"
  }
}
```

| Field       | Type   | Description                                | Possible values |
|-------------|--------|--------------------------------------------|--------------|
| timestamp   | Integer| Timestamp in nanoseconds                   |              |
| priceEp     | Integer| Scaled book level price                    |              |
| size        | Integer| Scaled book level size                     |              |
| sequence    | Integer| current message sequence                   |              |
| symbol      | String | Spot symbol name                       | [Trading symbols](#symbpricesub) |

* Sample：
```json
GET /md/orderbook?symbol=BTCUSD

{
  "error": null,
  "id": 0,
  "result": {
    "book": {
      "asks": [
        [
          87705000,
          1000000
        ],
        [
          87710000,
          200000
        ]
      ],
      "bids": [
        [
          87700000,
          2000000
        ],
        [
          87695000,
          200000
        ]
      ]
    },
    "depth": 30,
    "sequence": 455476965,
    "timestamp": 1583555482434235628,
    "symbol": "sBTCUSDT",
    "type": "snapshot"
  }
}
```


<a name="querytrades"/>

#### 6.3.2 Query Recent Trades

* Request：
```json
GET /md/trade?symbol=<symbol>&id=<id>
```

| Field       | Type   | Description                                | Possible values |
|-------------|--------|--------------------------------------------|--------------|
| symbol      | String | symbol name                                | [Trading symbols](#symbpricesub) |
| id          | Integer| Optional. Request id                       |              |

* Response:
```
{
  "error": null,
  "id": 0,
  "result": {
    "type": "snapshot",
    "sequence": <sequence>,
    "symbol": "<symbol>",
    "trades": [
      [
        <timestamp>,
        "<side>",
        <priceEp>,
        <size>
      ],
      ...
      ...
      ...
    ]
  }
}

```

| Field       | Type   | Description                                | Possible values |
|-------------|--------|--------------------------------------------|--------------|
| timestamp   | Integer| Timestamp in nanoseconds                   |              |
| side        | String | Trade side string                          | Buy, Sell    |
| priceEp     | Integer| Scaled trade price                         |              |
| size        | Integer| Scaled trade size                          |              |
| sequence    | Integer| Current message sequence                   |              |
| symbol      | String | Spot symbol name                       | [Trading symbols](#symbpricesub) |

* Sample：
```json
GET /md/trade?symbol=sBTCUSDT

{
  "error": null,
  "id": 0,
  "result": {
    "sequence": 15934323,
    "symbol": "sBTCUSDT",
    "trades": [
      [
        1579164056368538508,
        "Sell",
        86960000,
        121
      ],
      [
        1579164055036820552,
        "Sell",
        86960000,
        58
      ]
    ],
    "type": "snapshot"
  }
}

```

<a name="query24hrsticker"/>

#### 6.3.3 Query 24 Hours Ticker

* Request：
```json
GET /md/spot/ticker/24hr?symbol=<symbol>&id=<id>
```

| Field       | Type   | Description                                | Possible values |
|-------------|--------|--------------------------------------------|--------------|
| symbol      | String | symbol name                                | [Trading symbols](#symbpricesub) |
| id          | Integer| Optional. Request id                       |              |

* Response:
```
{
  "error": null,
  "id": 0,
  "result": {
    "openEp": <open priceEp>,
    "highEp": <high priceEp>,
    "lowEp": <low priceEp>,
    "lastEp": <last priceEp>,
    "bidEp": <bid priceEp>,
    "askEp": <ask priceEp>,
    "symbol": "<symbol>",
    "turnoverEv": <turnoverEv>,
    "volumeEv": <volumeEv>,
    "timestamp": <timestamp>
  }
}
```

| Field         | Type   | Description                                | Possible values |
|---------------|--------|--------------------------------------------|--------------|
| open priceEp  | Integer| The scaled open price in last 24 hours     |              |
| high priceEp  | Integer| The scaled highest price in last 24 hours  |              |
| low priceEp   | Integer| The scaled lowest price in last 24 hours   |              |
| last priceEp  | Integer| The scaled last price                      |              |
| bid priceEp   | Integer| Scaled bid price                           |              |
| ask priceEp   | Integer| Scaled ask price                           |              |
| timestamp     | Integer| Timestamp in nanoseconds                   |              |
| symbol        | String | symbol name                                | [Trading symbols](#symbpricesub) |
| turnoverEv    | Integer| The scaled turnover value in last 24 hours |              |
| volumeEv      | Integer| The scaled trade volume in last 24 hours   |              |

* Sample：
```json
GET /md/spot/ticker/24hr?symbol=sBTCUSDT

{
  "error": null,
  "id": 0,
  "result": {
    "bidEp": 87420000,
    "askEp": 87425000,
    "lastEp": 87425000,
    "highEp": 92080000,
    "lowEp": 87130000,
    "openEp": 90710000,
    "symbol": "sBTCUSDT",
    "timestamp": 1583646442444219017,
    "turnoverEv": 1399362834123,
    "volumeEv": 125287131
  }
}
```

<a name="mdapilistv1"/>

### 7.1 Market Data API List

<a name="queryorderbookv1"/>

#### 7.1.1 Query Order Book

* Request：
```json
GET /v1/md/orderbook?symbol=<symbol>&id=<id>
```

| Field       | Type   | Description                                | Possible values |
|-------------|--------|--------------------------------------------|--------------|
| symbol      | String | Spot symbol name                       | [Trading symbols](#symbpricesub) |
| id          | Integer| Optional. Request id                       |              |

* Response:
```
{
  "error": null,
  "id": 0,
  "result": {
    "book": {
      "asks": [
        [
          <priceEp>,
          <size>
        ],
        ...
        ...
        ...
      ],
      "bids": [
        [
          <priceEp>,
          <size>
        ],
        ...
        ...
        ...
      ],
    ]
    },
    "depth": 30,
    "sequence": <sequence>,
    "symbol": "<symbol>",
    "timestamp": "<timestamp>",
    "type": "snapshot"
  }
}
```

| Field       | Type   | Description                                | Possible values |
|-------------|--------|--------------------------------------------|--------------|
| timestamp   | Integer| Timestamp in nanoseconds                   |              |
| priceEp     | Integer| Scaled book level price                    |              |
| size        | Integer| Scaled book level size                     |              |
| sequence    | Integer| current message sequence                   |              |
| symbol      | String | Spot symbol name                       | [Trading symbols](#symbpricesub) |

* Sample：
```json
GET /v1/md/orderbook?symbol=BTCUSD

{
  "error": null,
  "id": 0,
  "result": {
    "book": {
      "asks": [
        [
          87705000,
          1000000
        ],
        [
          87710000,
          200000
        ]
      ],
      "bids": [
        [
          87700000,
          2000000
        ],
        [
          87695000,
          200000
        ]
      ]
    },
    "depth": 30,
    "sequence": 455476965,
    "symbol": "BTCUSD",
    "timestamp": 1583552267253988998,
    "type": "snapshot"
  }
}
```

<a name="spotTradingApi"/>

### Spot Trading Api List

<a name="spotPlaceOrder"/>

#### Place order

* Http Request:

```json
POST /spot/orders

{  
    "symbol":"sBTCUSDT",
    "clOrdID":"",
    "side":"Buy/Sell",
    "qtyType":"ByBase/ByQuote",
    "quoteQtyEv":0,
    "baseQtyEv":0,
    "priceEp":0,
    "stopPxEp":0,
    "trigger":"UNSPECIFIED",
    "ordType":"Limit",
    "timeInForce":"GoodTillCancel",
}

```

| Field       | Type   | Required | Description               | Possible values |
|----------   |--------|----------|---------------------------|-----------------|
| symbol      | String | Yes      |                           | [Spot Trading symbols](#spotSymList) |
| side        | Enum   | Yes      |                           |  Sell, Buy     | 
| qtyType     | Enum   | Yes      | default ByBase            | ByBase, ByQuote|
| quoteQtyEv  | Number | --       | Required when qtyType = ByQuote|  |
| baseQtyEv   | Number | --       |                           | Required when qtyType = ByBase   |
| priceEp     | Number |          |                           | Scaled price            |
| stopPxEp    | Number | --       | used in conditional order |   |
| trigger     | Enum   | --       | Required in conditional order | ByLastPrice |
| timeInForce    | Enum   | No       | Default GoodTillCancel | GoodTillCancel, PostOnly,ImmediateOrCancel,FillOrKill |
| ordType    | Enum   | No        | Default to Limit         |Market, Limit, Stop, StopLimit, MarketIfTouched, LimitIfTouched|

* Http Response

```json

{
    "code": 0,
        "msg": "",
        "data": {
            "orderID": "d1d09454-cabc-4a23-89a7-59d43363f16d",
            "clOrdID": "309bcd5c-9f6e-4a68-b775-4494542eb5cb",
            "priceEp": 0,
            "action": "New",
            "trigger": "UNSPECIFIED",
            "pegPriceType": "UNSPECIFIED",
            "stopDirection": "UNSPECIFIED",
            "bizError": 0,
            "symbol": "sBTCUSDT",
            "side": "Buy",
            "baseQtyEv": 0,
            "ordType": "Limit",
            "timeInForce": "GoodTillCancel",
            "ordStatus": "Created",
            "cumFeeEv": 0,
            "cumBaseQtyEv": 0,
            "cumQuoteQtyEv": 0,
            "leavesBaseQtyEv": 0,
            "leavesQuoteQtyEv": 0,
            "avgPriceEp": 0,
            "cumBaseAmountEv": 0,
            "cumQuoteAmountEv": 0,
            "quoteQtyEv": 0,
            "qtyType": "ByBase",
            "stopPxEp": 0,
            "pegOffsetValueEp": 0
        }
}

```
<a name="spotAmendOrder"/>

#### Amend Order

* Http Request

```
PUT /spot/orders?symbol=<symbol>&orderID=<orderID>&origClOrdID=<origClOrdID>&clOrdID=<clOrdID>&priceEp=<priceEp>&baseQtyEV=<baseQtyEV>&quoteQtyEv=<quoteQtyEv>&stopPxEp=<stopPxEp> 

```

* Http Response

   * amended order

<a name="spotCancelOrder"/>

#### Cancel Order

```
DELETE /spot/orders?symbol=<symbol>&orderID=<orderID>
DELETE /spot/orders?symbol=<symbol>&clOrdID=<clOrdID>

```

* Http Response
   * Canceled order

<a name="spotCancelAll"/>

#### Cancel all order by symbol

```
DELETE /spot/orders/all?symbol=<symbol>&untriggered=<untriggered>
```

| Field | Type | Required | Description |
|---------|--------|-------|-------------|
| symbol  | Enum   | Yes   | The symbol to cancel |
| untriggered | Boolean | No | set false to cancel non-conditiaonal order, true to conditional order |

* Http Response
  * Total orders canceled

<a name="spotQueryOpenOrder"/>

#### Query Open order by clOrdID or orderID

* Http Request

```
GET /spot/orders/active?symbol=<symbol>&orderID=<orderID>
GET /spot/orders/active?symbol=<symbol>&clOrDID=<clOrdID>

```

* Http Response

   * Order object


<a name="spotListAllOpenOrder"/>

#### Query all open orders by symbol


* Http Request

```
GET /spot/orders?symbol=<symbol>

```

* Http Response
   * List of orders

<a name="spotQueryWallet"/>

#### Query wallets
   * Query spot wallet by currency

* Http Request

```
GET /spot/wallets?currency=<currency>
```

* Http Response

```
{
    "code": 0,
        "msg": "",
        "data": [{
            "currency": "BTC",
            "balanceEv": 0,
            "lockedTradingBalanceEv": 0,
            "lockedWithdrawEv": 0,
            "lastUpdateTimeNs": 0,
        }]
}
```




<a name="wsapi"/>

# Websocket API Standards

<a name="sessionmanagement"/>

## 1. Session Management

* Each client is required to actively send heartbeat (ping) message to Phemex data gateway ('DataGW' in short) with interval less than 30 seconds, otherwise DataGW will drop the connection. If a client sends a ping message, DataGW will reply with a pong message.
* Clients can use WS built-in ping message or the application level ping message to DataGW as heartbeat. The heartbeat interval is recommended to be set as *5 seconds*, and actively reconnect to DataGW if don't receive messages in *3 heartbeat intervals*.

<a name="wsapiratelimits"/>

## 2 API Rate Limits

* Each Client has concurrent connection limit to *5* in maximum.
* Each connection has subscription limit to *20* in maximum.
* Each connection has throttle limit to *10* request/s.

<a name="wsapilist"/>

## 3. WebSocket API List

<a name="heartbeat"/>

### 3.1 Heartbeat

* Request：
```
{
  "id": <id>,
  "method": "server.ping",
  "params": []
}
```

* Response:
```
{
  "error": null,
  "id": <id>,
  "result": "pong"
}
```

* Sample：
```json
> {
  "id": 1234,
  "method": "server.ping",
  "params": []
}

< {
  "error": null,
  "id": 1234,
  "result": "pong"
}
```

<a name="apiuserauth"/>

### 3.2 API User Authentication

Market trade/orderbook are published publicly without user authentication.
While for client private account/position/order data, the client should send user.auth message to Data Gateway to authenticate the session.

* Request

```
{
  "method": "user.auth",
  "params": [
    "API",
    "<token>",
    "<signature>",
    <expiry>
  ],
  "id": 1234
}
```

| Field       | Type   | Description      | Possible values |
|-------------|--------|------------------|-----------------|
| type        | String | Token type       | API             |
| token       | String | API Key     |                 |
| signature   | String | Signature generated by a funtion as HMacSha256(API Key + expiry) with ***API Secret*** ||
| expiry      | Integer| A future time after which request will be rejected, in epoch ***second***. Maximum expiry is request time plus 5 minutes ||

* Sample:

```json
> {
  "method": "user.auth",
  "params": [
    "API",
    "806066b0-f02b-4d3e-b444-76ec718e1023",
    "8c939f7a6e6716ab7c4240384e07c81840dacd371cdcf5051bb6b7084897470e",
    157009123213
  ],
  "id": 1234
}

< {
  "error": null,
  "id": 1234,
  "result": {
    "status": "success"
  }
}
```


<a name="booksub"/>

### 3.3 Subscribe OrderBook 

On each successful subscription, DataGW will immediately send the current Order Book snapshot to client and all later order book updates will be published. 

* Request 
```
{
  "id": <id>,
  "method": "orderbook.subscribe",
  "params": [
    "<symbol>"
  ]
}
```

* Response
```
{
  "error": null,
  "id": <id>,
  "result": {
    "status": "success"
  }
}
```

* Sample：
```json
> {
  "id": 1234,
  "method": "orderbook.subscribe",
  "params": [
    "sBTCUSDT"
  ]
}

< {
  "error": null,
  "id": 1234,
  "result": {
    "status": "success"
  }
}
```


#### OrderBook Message:

* Message Format：
 
```
{
  "book": {
    "asks": [
      [
        <priceEp>,
        <qty>
      ],
      .
      .
      .
    ],
    "bids": [
      [
        <priceEp>,
        <qty>
      ],
      .
      .
      .
    ]
  },
  "depth": <depth>,
  "sequence": <sequence>,
  "timestamp": <timestamp>,
  "symbol": "<symbol>",

```

| Field       | Type   | Description      | Possible values |
|-------------|--------|------------------|-----------------|
| side        | String | Price level side | bid, ask        |
| priceEp     | Integer| Scaled price     |                 |
| qty         | Integer| Price level size. Non-zero qty indicates price level insertion or updation, and qty 0 indicates price level deletion. |                 |
| sequence    | Integer| Latest message sequence |          |
| depth       | Integer| Market depth     |                 |
| type        | String | Message type     | snapshot, incremental |
  

* Sample：
 
```json
< {"book":{"asks":[[86765000,19609],[86770000,7402],[86775000,3807],[86780000,7395],[86785000,3599],[86790000,7253],[86795000,4019],[86800000,4366],[86805000,3216],[86810000,3107],[86815000,7453],[86820000,1771],[86825000,895],[86830000,3420],[86835000,1818],[86840000,1272],[86845000,1064],[86850000,195],[86855000,1630],[86860000,1017],[86865000,3509],[86870000,1105],[86875000,1262],[86880000,893],[86885000,862],[86890000,1030],[86895000,2315],[86900000,2994],[86905000,2026],[86910000,3387],[86915000,1382],[86920000,1202],[86925000,3150],[86930000,1773],[86935000,1778],[86940000,1384],[86945000,1842],[86950000,1019],[86955000,2660],[86960000,1599],[86965000,920],[86970000,1834],[86975000,752],[86980000,1384],[86985000,2471],[86990000,2133],[86995000,2981],[87000000,1091],[87005000,994],[87010000,1217],[87015000,1098],[87020000,526],[87025000,1779],[87030000,1098],[87035000,892],[87040000,2168],[87045000,822],[87050000,2410],[87055000,630],[87060000,1684],[87065000,2556],[87070000,19],[87080000,1445],[87085000,29],[87105000,2002],[87115000,658],[87120000,660],[87905000,991]],"bids":[[86760000,18995],[86755000,6451],[86750000,5311],[86745000,6867],[86740000,6180],[86735000,3127],[86730000,4852],[86725000,6213],[86720000,3902],[86715000,4510],[86710000,10063],[86705000,1118],[86700000,1891],[86695000,767],[86690000,20920],[86685000,2535],[86680000,1105],[86675000,645],[86670000,1424],[86665000,1773],[86660000,1464],[86655000,1160],[86650000,1462],[86645000,2446],[86640000,538],[86635000,506],[86630000,2291],[86625000,2981],[86620000,1712],[86615000,984],[86610000,1058],[86605000,1261],[86600000,1074],[86595000,1408],[86590000,717],[86585000,1582],[86580000,1950],[86575000,1540],[86570000,2960],[86565000,598],[86560000,759],[86555000,1266],[86550000,1943],[86545000,259],[86540000,2106],[86535000,2365],[86530000,857],[86525000,1200],[86520000,2371],[86515000,2103],[86510000,1468],[86505000,747],[86500000,1369],[86495000,2121],[86490000,3674],[86485000,1345],[86480000,1290],[86475000,1716],[86470000,1851],[86465000,1861],[86460000,1092],[86435000,21],[86430000,986],[86420000,1202],[86415000,22],[86405000,1199],[86390000,470],[86365000,920],[86360000,192],[86355000,474],[86350000,1838],[86335000,1104],[86285000,2205],[86280000,2390],[86275000,95],[86255000,2836],[86250000,589],[86240000,424],[86235000,937],[86225000,374],[86220000,1591],[86215000,517],[86210000,559],[86205000,702],[86190000,54]]},"depth":100,"sequence":1191904,"symbol":"sBTCUSDT","type":"snapshot"}
< {"book":{"asks":[[86775000,4621]],"bids":[]},"depth":100,"sequence":1191905,"symbol":"sBTCUSDT","type":"incremental"}
< {"book":{"asks":[],"bids":[[86755000,8097]]},"depth":100,"sequence":1191906,"symbol":"sBTCUSDT","type":"incremental"}
```

<a name="bookunsub"/>

###  3.4 Unsubscribe OrderBook

It unsubscribes all orderbook related subscriptions.

* Request

```
{
  "id": <id>,
  "method": "orderbook.unsubscribe",
  "params": []
}
```

* Response:

```
{
  "error": null,
  "id": <id>,
  "result": {
    "status": "success"
  }
}
```


<a name="tradesub"/>

### 3.5 Subscribe Trade

On each successful subscription, DataGW will send the 1000 history trades immediately for the subscribed symbol and all the later trades will be published.

* Request

```
{
  "id": <id>,
  "method": "trade.subscribe",
  "params": [
    "<symbol>"
  ]
}
```

* Response:

```
{
  "error": null,
  "id": <id>,
  "result": {
    "status": "success"
  }
}
```

* Sample:

```json
> {
  "id": 1234,
  "method": "trade.subscribe",
  "params": [
    "sBTCUSDT"
  ]
}

< {
  "error": null,
  "id": 1234,
  "result": {
    "status": "success"
  }
}
```

#### Trade Message Format：

```
{
  "trades": [
    [
      <timestamp>,
      "<side>",
      <priceEp>,
      <qty>
    ],
    .
    .
    .
  ],
  "sequence": <sequence>,
  "symbol": "<symbol>",
  "type": "<type>"
}
```

| Field       | Type   | Description      | Possible values |
|-------------|--------|------------------|-----------------|
| timestamp   | Integer| Timestamp in nanoseconds for each trade ||
| side        | String | Execution taker side| bid, ask        |
| priceEp     | Integer| Scaled execution price  |                 |
| qty         | Integer| Execution size   |                 |
| sequence    | Integer| Latest message sequence ||
| symbol      | String | Spot symbol name     ||
| type        | String | Message type     |snapshot, incremental |
  

* Sample
```json
< {
  "sequence": 1167852,
  "symbol": "sBTCUSDT",
  "trades": [
    [
      1573716998128563500,
      "Buy",
      86735000,
      56
    ],
    [
      1573716995033683000,
      "Buy",
      86735000,
      52
    ],
    [
      1573716991485286000,
      "Buy",
      86735000,
      51
    ],
    [
      1573716988636291300,
      "Buy",
      86735000,
      12
    ]
  ],
  "type": "snapshot"
}

< {
  "sequence": 1188273,
  "symbol": "sBTCUSDT",
  "trades": [
    [
      1573717116484024300,
      "Buy",
      86730000,
      21
    ]
  ],
  "type": "incremental"
}
```

<a name="tradeunsub"/>

### 3.6 Unsubscribe Trade

It unsubscribes all trade subscriptions.

* Request

```
{
  "id": <id>,
  "method": "trade.subscribe",
  "params": [
    "<symbol>"
  ]
}
```

* Response:

```
{
  "error": null,
  "id": <id>,
  "result": {
    "status": "success"
  }
}
```

<a name="klinesub"/>

### 3.7 Subscribe Kline

On each successful subscription, DataGW will send the 1000 history klines immediately for the subscribed symbol and all the later kline update will be published in real-time.

* Request

```
{
  "id": <id>,
  "method": "kline.subscribe",
  "params": [
    "<symbol>",
    "<interval>"
  ]
}
```

* Response:

```
{
  "error": null,
  "id": <id>,
  "result": {
    "status": "success"
  }
}
```

* Sample:

```json
# subscribe 1-day kline
> {
  "id": 1234,
  "method": "kline.subscribe",
  "params": [
    "sBTCUSDT",
    86400
  ]
}

< {
  "error": null,
  "id": 1234,
  "result": {
    "status": "success"
  }
}
```

#### Kline Message Format：

```
{
  "klines": [
    [
      <timestamp>,
      "<interval>",
      <lastCloseEp>,
      <openEp>,
      <highEp>,
      <lowEp>,
      <closeEp>,
      <volumeEv>,
      <turnoverEv>,
    ],
    .
    .
    .
  ],
  "sequence": <sequence>,
  "symbol": "<symbol>",
  "type": "<type>"
}
```

| Field       | Type   | Description      | Possible values |
|-------------|--------|------------------|-----------------|
| timestamp   | Integer| Timestamp in nanoseconds for each trade ||
| interval    | Integer| Kline interval type      | 60, 300, 900, 1800, 3600, 14400, 86400, 604800, 2592000, 7776000, 31104000 |
| lastCloseEp | Integer| Scaled last close price  |                 |
| openEp      | Integer| Scaled open price        |                 |
| highEp      | Integer| Scaled high price        |                 |
| lowEp       | Integer| Scaled low price         |                 |
| closeEp     | Integer| Scaled close price       |                 |
| volumeEv    | Integer| Scaled trade voulme during the current kline interval ||
| turnoverEv  | Integer| Scaled turnover value    |                 |
| sequence    | Integer| Latest message sequence  ||
| symbol      | String | Contract symbol name     ||
| type        | String | Message type     |snapshot, incremental |
  

* Sample
```json
< {
  "kline": [
    [
      1590019200,
      86400,
      952057000000,
      952000000000,
      955587000000,
      947835000000,
      954446000000,
      1162621600,
      11095452729869
    ],
    [
      1589932800,
      86400,
      977566000000,
      978261000000,
      984257000000,
      935452000000,
      952057000000,
      11785486656,
      113659374080189
    ],
    [
      1589846400,
      86400,
      972343000000,
      972351000000,
      989607000000,
      949106000000,
      977566000000,
      11337554900,
      109928494593609
    ]
  ],
  "sequence": 380876982,
  "symbol": "sBTCUSDT",
  "type": "snapshot"
}

< {
  "kline": [
    [
      1590019200,
      86400,
      952057000000,
      952000000000,
      955587000000,
      928440000000,
      941597000000,
      4231329700,
      40057408967508
    ]
  ],
  "sequence": 396865028,
  "symbol": "sBTCUSDT",
  "type": "incremental"
}
```

<a name="klineunsub"/>

### 3.8 Unsubscribe Kline

It unsubscribes all kline subscriptions or for a symbol.

* Request

```
# unsubscribe all Kline subscriptions
{
  "id": <id>,
  "method": "kline.unsubscribe",
  "params": []
}

# unsubscribe all Kline subscriptions of a symbol
{
  "id": <id>,
  "method": "kline.unsubscribe",
  "params": [
    "<symbol>"
  ]
}
```

* Response:

```
{
  "error": null,
  "id": <id>,
  "result": {
    "status": "success"
  }
}
```


<a name="wosub"/>

### 3.7 Subscribe Wallet-Order messages

WO subscription requires the session been authorized successfully. DataGW extracts the user information from the given token and sends WO messages back to client accordingly. 0 or more latest WO snapshot messages will be sent to client immediately on subscription, and incremental messages will be sent for later updates. Each account snapshot contains a users' wallets and open / max 100 closed / max 100 filled order event message history.

* Request

```
{
  "id": <id>,
  "method": "wo.subscribe",
  "params": []
}
```

* Response:

```
{
  "error": null,
  "id": <id>,
  "result": {
    "status": "success"
  }
}
```

* Sample
```json
> {
  "id": 1234,
  "method": "wo.subscribe",
  "params": []
}

< {
  "error": null,
  "id": 1234,
  "result": {
    "status": "success"
  }
}
```




#### Wallet-Order Message Sample:

```
{"wallets":[],"orders":[{"userID":60463,...}],"sequence":11450, "timestamp":<timestamp>, "type":"<type>"}

```

| Field       | Type   | Description      | Possible values |
|-------------|--------|------------------|-----------------|
| timestamp   | Integer| Transaction timestamp in nanoseconds | |
| sequence    | Integer| Latest message sequence |          |
| symbol      | String | Symbol name    |          |
| type        | String | Message type     | snapshot, incremental |



* Sample:

```json
< {"orders":{"closed":[{"action":"New","avgPriceEp":0,"baseCurrency":"BTC","baseQtyEv":10000,"bizError":0,"clOrdID":"123456","createTimeNs":1587463924959744646,"cumBaseQtyEv":10000,"cumFeeEv":0,"cumQuoteQtyEv":66900000,"curBaseWalletQtyEv":899990000,"curQuoteWalletQtyEv":66900000,"cxlRejReason":0,"feeCurrency":"BTC","leavesBaseQtyEv":0,"leavesQuoteQtyEv":0,"ordStatus":"Filled","ordType":"Limit","orderID":"35217ade-3c6b-48c7-a280-8a1edb88013e","pegOffsetValueEp":0,"priceEp":68000000,"qtyType":"ByBase","quoteCurrency":"USDT","quoteQtyEv":66900000,"side":"Sell","stopPxEp":0,"symbol":"sBTCUSDT","timeInForce":"GoodTillCancel","transactTimeNs":1587463924964876798,"triggerTimeNs":0,"userID":200076}],"fills":[{"avgPriceEp":0,"baseCurrency":"BTC","baseQtyEv":10000,"clOrdID":"123456","execBaseQtyEv":10000,"execFeeEv":0,"execID":"8135ebe3-f767-577b-b70d-1a839d5178e0","execPriceEp":669000000000,"execQuoteQtyEv":66900000,"feeCurrency":"BTC","lastLiquidityInd":"RemovedLiquidity","ordType":"Limit","orderID":"35217ade-3c6b-48c7-a280-8a1edb88013e","priceEp":68000000,"qtyType":"ByBase","quoteCurrency":"USDT","quoteQtyEv":66900000,"side":"Sell","symbol":"sBTCUSDT","transactTimeNs":1587463924964876798,"userID":200076}],"open":[{"action":"New","avgPriceEp":0,"baseCurrency":"BTC","baseQtyEv":100000000,"bizError":0,"clOrdID":"31f793f4-163d-aa3f-5994-0e1164719ba2","createTimeNs":1587547657438535949,"cumBaseQtyEv":0,"cumFeeEv":0,"cumQuoteQtyEv":0,"curBaseWalletQtyEv":630000005401500000,"curQuoteWalletQtyEv":351802500000,"cxlRejReason":0,"feeCurrency":"BTC","leavesBaseQtyEv":100000000,"leavesQuoteQtyEv":0,"ordStatus":"New","ordType":"Limit","orderID":"b98b25c5-6aa4-4158-b9e5-477e37bd46d8","pegOffsetValueEp":0,"priceEp":666500000000,"qtyType":"ByBase","quoteCurrency":"USDT","quoteQtyEv":0,"side":"Sell","stopPxEp":0,"symbol":"sBTCUSDT","timeInForce":"GoodTillCancel","transactTimeNs":1587547657442752950,"triggerTimeNs":0,"userID":200076}]},"sequence":349,"timestamp":1587549121318737606,"type":"snapshot","wallets":[{"balanceEv":0,"currency":"LTC","lastUpdateTimeNs":1587481897840503662,"lockedTradingBalanceEv":0,"lockedWithdrawEv":0,"userID":200076},{"balanceEv":351802500000,"currency":"USDT","lastUpdateTimeNs":1587543489127498121,"lockedTradingBalanceEv":0,"lockedWithdrawEv":0,"userID":200076},{"balanceEv":630000005401500000,"currency":"BTC","lastUpdateTimeNs":1587547210089640382,"lockedTradingBalanceEv":100000000,"lockedWithdrawEv":0,"userID":200076},{"balanceEv":0,"currency":"ETH","lastUpdateTimeNs":1587481897840503662,"lockedTradingBalanceEv":0,"lockedWithdrawEv":0,"userID":200076},{"balanceEv":0,"currency":"XRP","lastUpdateTimeNs":1587481897840503662,"lockedTradingBalanceEv":0,"lockedWithdrawEv":0,"userID":200076},{"balanceEv":0,"currency":"LINK","lastUpdateTimeNs":1587481897840503662,"lockedTradingBalanceEv":0,"lockedWithdrawEv":0,"userID":200076},{"balanceEv":0,"currency":"XTZ","lastUpdateTimeNs":1587481897840503662,"lockedTradingBalanceEv":0,"lockedWithdrawEv":0,"userID":200076}]}
< {"orders":{"closed":[],"fills":[],"open":[{"action":"New","avgPriceEp":0,"baseCurrency":"BTC","baseQtyEv":100000000,"bizError":0,"clOrdID":"0c1099e5-b900-5351-cf60-edb15ea2539c","createTimeNs":1587549529513521745,"cumBaseQtyEv":0,"cumFeeEv":0,"cumQuoteQtyEv":0,"curBaseWalletQtyEv":630000005401500000,"curQuoteWalletQtyEv":351802500000,"cxlRejReason":0,"feeCurrency":"BTC","leavesBaseQtyEv":100000000,"leavesQuoteQtyEv":0,"ordStatus":"New","ordType":"Limit","orderID":"494a6cbb-32b3-4d6a-b9b7-196ea2506fb5","pegOffsetValueEp":0,"priceEp":666500000000,"qtyType":"ByBase","quoteCurrency":"USDT","quoteQtyEv":0,"side":"Sell","stopPxEp":0,"symbol":"sBTCUSDT","timeInForce":"GoodTillCancel","transactTimeNs":1587549529518394235,"triggerTimeNs":0,"userID":200076}]},"sequence":350,"timestamp":1587549529519959388,"type":"incremental","wallets":[{"balanceEv":630000005401500000,"currency":"BTC","lastUpdateTimeNs":1587547210089640382,"lockedTradingBalanceEv":200000000,"lockedWithdrawEv":0,"userID":200076},{"balanceEv":351802500000,"currency":"USDT","lastUpdateTimeNs":1587543489127498121,"lockedTradingBalanceEv":0,"lockedWithdrawEv":0,"userID":200076}]}

```


<a name="wounsub"/>

### 3.8 Unsubscribe Wallet-Order
* Request：

```
{
  "id": <id>,
  "method": "wo.unsubscribe",
  "params": []
}
```

* Response:

```
{
  "error": null,
  "id": <id>,
  "result": {
    "status": "success"
  }
}
```


<a name="tickersub"/>

### 3.9 Subscribe 24 Hours Ticker
On each successful subscription, DataGW will publish 24-hour ticker metrics for all symbols every 5 seconds.

* Request

```
{
  "id": <id>,
  "method": "spot_market24h.subscribe",
  "params": []
}
```

* Response:

```
{
  "error": null,
  "id": <id>,
  "result": {
    "status": "success"
  }
}
```

* Sample:

```json
> {
  "method": "spot_market24h.subscribe",
  "params": [],
  "id": 1234
}

< {
  "error": null,
  "id": 1234,
  "result": {
    "status": "success"
  }
}
```

#### 24 Hours Ticker Message Format：

```
{
  "spot_market24h": {
    "openEp": <open priceEp>,
    "highEp": <high priceEp>,
    "lowEp": <low priceEp>,
    "lastEp": <last priceEp>,
    "bidEp": <bid priceEp>,
    "askEp": <ask priceEp>,
    "symbol": "<symbol>",
    "turnoverEv": <turnoverEv>,
    "volumeEv": <volumeEv>
  },
  "timestamp": <timestamp>
}
```

| Field         | Type   | Description                                | Possible values |
|---------------|--------|--------------------------------------------|--------------|
| open priceEp  | Integer| The scaled open price in last 24 hours     |              |
| high priceEp  | Integer| The scaled highest price in last 24 hours  |              |
| low priceEp   | Integer| The scaled lowest price in last 24 hours   |              |
| last priceEp  | Integer| The scaled last price                      |              |
| bid priceEp   | Integer| Scaled bid price                           |              |
| ask priceEp   | Integer| Scaled ask price                           |              |
| timestamp     | Integer| Timestamp in nanoseconds                   |              |
| symbol        | String | Spot Symbol name                       | [Trading symbols](#symbpricesub) |
| turnoverEv    | Integer| The scaled turnover value in last 24 hours |              |
| volumeEv      | Integer| The scaled trade volume in last 24 hours   |              |
  
* Sample:

```json
< {
  "spot_market24h": {
    "bidEp": 87450676,
    "askEp": 87450676,
    "closeEp": 87425000,
    "highEp": 92080000,
    "lowEp": 87130000,
    "openEp": 90710000,
    "symbol": "BTCUSD",
    "timestamp": 1583646442444219017,
    "turnoverEv": 1399362834123,
    "volumeEv": 125287131
  },
  "timestamp": 1576490244024818000
}
```

<a name="symbpricesub"/>

### 3.10 All Trading Symbols

   | symbol  |
   |---------|
   |sBTCUSDT |
   |sXRPUSDT |
   |sETHUSDT |
   |sLINKUSDT|
   |sXTZUSDT |
   |sLTCUSDT |


