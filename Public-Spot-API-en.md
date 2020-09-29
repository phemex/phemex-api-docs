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
      * [Query Closed orders](#spotQueryClosedOrder)
      * [Query history trades](#spotQueryHistTrade)
    * [Spot Asset Api List](#spotAssetAPIList)
      * [Query Deposit history](#depositHist)
      * [Query withdraw history](#withdrawHist)


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

## Restful API Response

<a name="httpreturncodes"/>

### HTTP Return Codes

* HTTP `401` return code is used when unauthenticated
* HTTP `403` return code is used when lack of priviledge.
* HTTP `429` return code is used when breaking a request rate limit.
* HTTP `5XX` return codes are used for Phemex internal errors. Note: This doesn't means the operation failed, the execution status is **UNKNOWN** and could be Succeed.

<a name="responseformat"/>

### Rest Response format

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

### Error codes

[Trading Error Codes](TradingErrorCode.md)

## HTTP REST Request Header 

Every HTTP Rest Request must have the following Headers:
* x-phemex-access-token : This is ***API-KEY*** (id field) from Phemex site.
* x-phemex-request-expiry : This describes the Unix ***EPoch SECONDS*** to expire the request, normally it should be (Now() + 1 minute)
* x-phemex-request-signature : This is HMAC SHA256 signature of the http request. Secret is ***API Secret***, its formula is : HMacSha256( URL Path + QueryString + Expiry + body )


<a name="apiratelimits"/>

## API Rate Limits

* Ratelimte group of contract trading api is ***SPOTORDER***.  
* RateLimit Explained [phemex ratelimite docs](/Generic-API-Info.en.md)
* Contract trading api response carries following headers.

```
X-RateLimit-Remaining-SPOTORDER, # Remaining request permits in this minute
X-RateLimit-Capacity-SPOTORDER, # Request ratelimit capacity
X-RateLimit-Retry-After-SPOTORDER, # Reset timeout in seconds for current ratelimited user
```

<a name="securitytype"/>

## Endpoint security type

* Each API call must be signed and pass to server in HTTP header `x-phemex-request-signature`.
* Endpoints use `HMAC SHA256` signatures. The `HMAC SHA256 signature` is a keyed `HMAC SHA256` operation. Use your `apiSecret` as the key and the string `URL Path + QueryString + Expiry + body )` as the value for the HMAC operation.
* `apiSecret` = `Base64::urlDecode(API Secret)`
* The `signature` is **case sensitive**.

<a name="signatureexample1"/>

### Signature Example 1: HTTP GET Request

* API REST Request URL: https://api.phemex.com/spot/wallets?currency=BTC 
   * Request Path: /spot/wallets
   * Request Query: currency=BTC
   * Request Body: <null>
   * Request Expiry: 1587552406
   * Signature: HMacSha256( /spot/wallets + currency=BTC + 1587552406 )

<a name="signatureexample2"/>

### Singature Example 2: HTTP GET Request with multiple query string

* API REST Request URL: https://api.phemex.com/spot/orders/active?symbol=sBTCUSDT&orderID=bc2b8ff1-a73b-4673-aa5b-fda632285fcc 
    * Request Path: /spot/orders/active
    * Request Query: symbol=sBTCUSDT&orderID=bc2b8ff1-a73b-4673-aa5b-fda632285fcc 
    * Request Body: <null>
    * Request Expire: 1587552407
    * Signature: HMacSha256(/spot/orders/active + symbol=sBTCUSDT&orderID=bc2b8ff1-a73b-4673-aa5b-fda632285fcc + 1587552407)
    * signed string is `/spot/orders/activesymbol=sBTCUSDT&orderID=bc2b8ff1-a73b-4673-aa5b-fda632285fcc1587552407`

<a name="signatureexample3"/>

### Signature Example 3: HTTP POST Request

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

## Request/response field explained


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

### Common constants

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

## REST API List

<a name="marketapilist"/>

### Market API List 

<a name="queryproductinfo"/>

#### Query Product Information 

* Request：
```json
GET /exchange/public/cfg/v2/products

```



<a name="mdapilist"/>

### Market Data API List

<a name="queryorderbook"/>

#### Query Order Book

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
GET /md/orderbook?symbol=sBTCUSDT

{
  "error": null,
  "id": 0,
  "result": {
    "book": {
      "asks": [
        [
          877050000000,
          1000000
        ],
        [
          877100000000,
          200000
        ]
      ],
      "bids": [
        [
          877000000000,
          2000000
        ],
        [
          876950000000,
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

#### Query Recent Trades

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
        869600000000,
        1210000
      ],
      [
        1579164055036820552,
        "Sell",
        869600000000,
        580000
      ]
    ],
    "type": "snapshot"
  }
}

```

<a name="query24hrsticker"/>

#### Query 24 Hours Ticker

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
    "askEp": 892100000000,
    "bidEp": 891835000000,
    "highEp": 898264000000,
    "lastEp": 892486000000,
    "lowEp": 870656000000,
    "openEp": 896261000000,
    "symbol": "sBTCUSDT",
    "timestamp": 1590571240030003249,
    "turnoverEv": 104718804814499,
    "volumeEv": 11841148100
  }
}
```

<a name="mdapilistv1"/>

### Market Data API List

<a name="queryorderbookv1"/>

#### Query Order Book

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
GET /v1/md/orderbook?symbol=sBTCUSDT

{
  "error": null,
  "id": 0,
  "result": {
    "book": {
      "asks": [
        [
          87705000000,
          1000000
        ],
        [
          87710000000,
          200000
        ]
      ],
      "bids": [
        [
          877000000000,
          2000000
        ],
        [
          876950000000,
          200000
        ]
      ]
    },
    "depth": 30,
    "sequence": 455476965,
    "symbol": "sBTCUSDT",
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

<a name="spotQueryClosedOrder"/>

#### Query spot closed orders
    * Query closed orders by symbol

* Http Request

```
GET /exchange/spot/order?symbol=<symbol>&ordStatus=<ordStatus1,orderStatus2>ordType=<ordType1,orderType2>&start=<start>&end=<end>&limit=<limit>&offset=<offset>

```

| Field      | Type | Required | Description |  Possible Values |
|-----------|------|----------|-------------|------------------|
| symbol    | String | Yes    | symbol to query | [Trading symbols](#symbpricesub) |
| ordStatus | enum   | No     | order status filter |[common constants for order status](#commconsts) |
| ordType   | enum   | No     | order type filter |[common constants for order type](#commconsts) |
| start     | integer | No    | Epoch millisecond, start of time range    |  within 3 month        |
| end       | integer | No    | Epoch millisecond, end of time range      | within 3 months |


* Http Response

   * return a list of spot order model.

```
{
    "code": 0,
        "msg": "OK",
        "data": {
            "total": 2,
            "rows": [
            {
                "orderID": "string",
                "stopPxEp": 0,
                "avgPriceEp": 0,
                "qtyType": "<ByQuote/ByBase>",
                "leavesBaseQtyEv": 0,
                "leavesQuoteQtyEv": 0,
                "baseQtyEv": "0",
                "feeCurrency": "string",
                "stopDirection": "UNSPECIFIED",
                "symbol": "string",
                "side": "enum",
                "quoteQtyEv": 0,
                "priceEp": 0,
                "ordType": "enum",
                "timeInForce": "enum",
                "ordStatus": "enum",
                "execStatus": "enum",
                "createTimeNs": 0,
                "cumFeeEv": 0,
                "cumBaseValueEv": 0,
                "cumQuoteValueEv": 0 
            }]
        }
}

```

* Sample Response

```
{
    "code": 0,
        "msg": "OK",
        "data": {
            "total": 2,
            "rows": [
            {
                "orderID": "b2b7018d-f02f-4c59-b4cf-051b9c2d2e83",
                "stopPxEp": 0,
                "avgPriceEp": 970056000000,
                "qtyType": "ByQuote",
                "leavesBaseQtyEv": 0,
                "leavesQuoteQtyEv": 0,
                "baseQtyEv": "0",
                "feeCurrency": "1",
                "stopDirection": "UNSPECIFIED",
                "symbol": "sBTCUSDT",
                "side": "Buy",
                "quoteQtyEv": 1000000000,
                "priceEp": 970056000000,
                "ordType": "Limit",
                "timeInForce": "GoodTillCancel",
                "ordStatus": "Filled",
                "execStatus": "MakerFill",
                "createTimeNs": 1589449348601287000,
                "cumFeeEv": 0,
                "cumBaseValueEv": 103000,
                "cumQuoteValueEv": 999157680
            }
            ]
        }
}

```


<a name="spotQueryHistTrade"/>

#### Query spot history trades
   
   * Query spot history trades by symbol

* Http Request

```
GET /exchange/spot/order/trades?symbol=<symbol>&start=<start>&end=<end>&limit=<limit>&offset=<offset>

```

| Field | Type | Required | Description | Possible values |
|-------|------|----------|-------------|-----------------|
| symbol | string | Yes   | symbol to query | [spot symbol list](#spotSymList) |
| start | integer | No    | Epoch millisecond, start of time range      |  |
| end   | integer | No    | Epoch millisecond, end of time range  | | 

* Http Response

```

{
    "code": 0,
        "msg": "OK",
        "data": {
            "total": 1,
            "rows": [
            {
                "qtyType": "ByQuote/ByBase",
                "transactTimeNs": 0,
                "clOrdID": "string"
                "orderID": "string",
                "symbol": "string",
                "side": "enum",
                "priceEP": 0,
                "baseQtyEv": 0,
                "quoteQtyEv": 0,
                "action": "enum",
                "execStatus": "enum",
                "ordStatus": "enum",
                "ordType": "enum",
                "execInst": "enum",
                "timeInForce": enum",
                "stopDirection": "enum",
                "tradeType": "enum",
                "stopPxEp": 0,
                "execId": "0"
                "execPriceEp": 0,
                "execBaseQtyEv": 0,
                "execQuoteQtyEv": 0,
                "leavesBaseQtyEv": 0,
                "leavesQuoteQtyEv": 0,
                "execFeeEv": 0,
                "feeRateEr": 0
            }
            ]
        }
}

```

* Sample Response

```

{
    "code": 0,
        "msg": "OK",
        "data": {
            "total": 1,
            "rows": [
            {
                "qtyType": "ByQuote",
                "transactTimeNs": 1589450974800550100,
                "clOrdID": "8ba59d40-df25-d4b0-14cf-0703f44e9690",
                "orderID": "b2b7018d-f02f-4c59-b4cf-051b9c2d2e83",
                "symbol": "sBTCUSDT",
                "side": "Buy",
                "priceEP": 970056000000,
                "baseQtyEv": 0,
                "quoteQtyEv": 1000000000,
                "action": "New",
                "execStatus": "MakerFill",
                "ordStatus": "Filled",
                "ordType": "Limit",
                "execInst": "None",
                "timeInForce": "GoodTillCancel",
                "stopDirection": "UNSPECIFIED",
                "tradeType": "Trade",
                "stopPxEp": 0,
                "execId": "c6bd8979-07ba-5946-b07e-f8b65135dbb1",
                "execPriceEp": 970056000000,
                "execBaseQtyEv": 103000,
                "execQuoteQtyEv": 999157680,
                "leavesBaseQtyEv": 0,
                "leavesQuoteQtyEv": 0,
                "execFeeEv": 0,
                "feeRateEr": 0
            }
            ]
        }
}

```

<a name="spotAssetApiList"/>

# Spot Asset API

<a name="depositHist"/>

#### Query recent deposit history within 3 months

* Http Request

```
GET /exchange/wallets/depositList?currency=<currency>&offset=<offset>&limit=<limit>
```

| Field    | Type   | Required  | Description| Possible Values |
|----------|--------|-----------|------------|-----------------|
| currency | String | True      | the currency to query | BTC,ETH, ... |


* Response

```
{ 
    address: "1xxxxxxxxxxxxxxxxxx"
    amountEv: 1000000
    confirmations: 1
    createdAt: 1574685871000
    currency: "BTC"
    currencyCode: 1
    status: "Success"
    txHash: "9e84xxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
    type: "Deposit"
}
```

<a name="withdrawHist"/>

#### Query recent withdraw history within 3 months

* Http Request

```
GET /exchange/wallets/withdrawList?currency=<currency>&offset=<offset>&limit=<limit>
```

* Response

```
{
    address: "1Lxxxxxxxxxxx"
    amountEv: 200000
    currency: "BTC"
    currencyCode: 1
    expiredTime: 0
    feeEv: 50000
    rejectReason: null
    status: "Succeed"
    txHash: "44exxxxxxxxxxxxxxxxxxxxxx"
    withdrawStatus: ""
}
```


<a name="wsapi"/>

# Websocket API Standards

<a name="sessionmanagement"/>

## Session Management

* Each client is required to actively send heartbeat (ping) message to Phemex data gateway ('DataGW' in short) with interval less than 30 seconds, otherwise DataGW will drop the connection. If a client sends a ping message, DataGW will reply with a pong message.
* Clients can use WS built-in ping message or the application level ping message to DataGW as heartbeat. The heartbeat interval is recommended to be set as *5 seconds*, and actively reconnect to DataGW if don't receive messages in *3 heartbeat intervals*.

<a name="wsapiratelimits"/>

## API Rate Limits

* Each Client has concurrent connection limit to *5* in maximum.
* Each connection has subscription limit to *20* in maximum.
* Each connection has throttle limit to *10* request/s.

<a name="wsapilist"/>

## WebSocket API List

<a name="heartbeat"/>

### Heartbeat

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

### API User Authentication

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
    1570091232
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

### Subscribe OrderBook 

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
< {"book":{"asks":[[892697000000,1781800],[892708000000,7543500],[892718000000,6552500],[892720000000,4714100],[892728000000,8431000],[892735000000,11644800],[892756000000,5909600],[892790000000,9053100],[892798000000,3336400],[892819000000,1689500],[892828000000,1616700],[892855000000,6484000],[892869000000,6873200],[892872000000,1919900],[892875000000,2373200],[892942000000,1875300],[892944000000,3117500],[892962000000,1353500],[892965000000,2589800],[892966000000,11169800],[892973000000,7829700],[892977000000,2697200],[892978000000,2110700],[892985000000,12563700],[892988000000,5374100],[893023000000,3816000],[893031000000,5852700],[893035000000,4990900],[893061000000,3479500],[893083000000,327900]],"bids":[[892376000000,6866500],[892354000000,14209000],[892353000000,5287200],[892348000000,6417800],[892340000000,8074400],[892334000000,3991900],[892303000000,4558000],[892295000000,10154700],[892280000000,16214500],[892277000000,11425300],[892270000000,39156500],[892268000000,13821100],[892260000000,32157500],[892257000000,5466100],[892252000000,11468700],[892241000000,13940300],[892226000000,33832300],[892220000000,3000000],[892171000000,4320400],[892165000000,4454000],[892152000000,5336400],[892144000000,4539200],[892134000000,7472200],[892127000000,5352700],[892087000000,10264400],[892082000000,4908000],[892038000000,1485500],[892031000000,4089200],[892030000000,4895500],[892014000000,3846600]]},"depth":30,"sequence":677996311,"symbol":"sBTCUSDT","timestamp":1590570810187570850,"type":"snapshot"}
< {"book":{"asks":[[892696000000,1669700],[892697000000,10455500],[892728000000,0],[892735000000,0],[892748000000,1550900],[892790000000,0],[892819000000,19087900],[892860000000,17152500],[892882000000,11546100],[892893000000,10986500],[892973000000,0],[892985000000,0],[893004000000,8306500],[893061000000,0],[893065000000,5446400],[893073000000,0],[893083000000,0],[893073000000,0],[893083000000,0]],"bids":[]},"depth":30,"sequence":677996548,"symbol":"sBTCUSDT","timestamp":1590570810819505422,"type":"incremental"}
< {"book":{"asks":[],"bids":[[892387000000,4792900],[892354000000,3170700],[892226000000,0],[892187000000,14425000],[892171000000,6366500],[892165000000,0],[892135000000,11511400],[892134000000,0],[892127000000,0],[892090000000,5446000],[892079000000,4687800],[892041000000,8590200],[892030000000,0],[892014000000,0]]},"depth":30,"sequence":677996941,"symbol":"sBTCUSDT","timestamp":1590570811244188841,"type":"incremental"}
```

<a name="bookunsub"/>

### Unsubscribe OrderBook

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

### Subscribe Trade

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
      867350000000,
      560000
    ],
    [
      1573716995033683000,
      "Buy",
      867350000000,
      520000
    ],
    [
      1573716991485286000,
      "Buy",
      867350000000,
      510000
    ],
    [
      1573716988636291300,
      "Buy",
      867350000000,
      120000
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
      86730000000,
      210000
    ]
  ],
  "type": "incremental"
}
```

<a name="tradeunsub"/>

### Unsubscribe Trade

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

### Subscribe Kline

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

### Unsubscribe Kline

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

### Subscribe Wallet-Order messages

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

### Unsubscribe Wallet-Order
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

### Subscribe 24 Hours Ticker
On each successful subscription, DataGW will publish 24-hour ticker metrics for all symbols every 1 second.

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

#### Hours Ticker Message Format：

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
    "askEp": 892100000000,
    "bidEp": 891835000000,
    "highEp": 898264000000,
    "lastEp": 892486000000,
    "lowEp": 870656000000,
    "openEp": 896261000000,
    "symbol": "sBTCUSDT",
    "timestamp": 1590571240030003249,
    "turnoverEv": 104718804814499,
    "volumeEv": 11841148100
  },
  "timestamp": 1576490244024818000
}
```

<a name="symbpricesub"/>

### All Trading Symbols

   | symbol  |
   |---------|
   |sBTCUSDT |
   |sXRPUSDT |
   |sETHUSDT |
   |sLINKUSDT|
   |sXTZUSDT |
   |sLTCUSDT |


