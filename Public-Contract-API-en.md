## Table of Contents * [Phemex Public API](#publicapi) * [General Public API Information](#general)
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
    * [Price/Ratio/Value Scales](#scalingfactors)
    * [Common constants](#commconsts)
  * [REST API List](#restapilist)
    * [Market API List](#marketapilist)
      * [Query Product Information](#queryproductinfo)
    * [Trade API List](#orderapilist)
      * [Place Order With Put Method, *Prefered*](#placeorderwithput)
      * [Order examples](#orderexample)
      * [Place Order](#placeorder)
      * [Amend Order by OrderID](#amendorder)
      * [Cancel Single Order by OrderID](#cancelsingleorder)
      * [Bulk Cancel Orders](#cancelorder)
      * [Cancel All Orders](#cancelall)
      * [Query Trading Account and Positions](#querytradeaccount)
      * [Query Trading Account and Positions with unrealized PNL](#queryPosWithPnl)
      * [Change Position Leverage](#changeleverage)
      * [Change Position Risklimt](#changerisklimit)
      * [Assign Position Balance in Isolated Margin Mode](#assignposbalance)
      * [Query Open Orders by Symbol](#queryopenorder)
      * [Query Closed Orders by Symbol](#queryorder)
      * [Query Order by orderID](#queryorderbyid)
      * [Query User Trades by Symbol](#querytrade)
    * [Market Data API List ](#mdapilist)
      * [Query Order Book](#queryorderbook)
      * [Query kline](#querykline)
      * [Query Recent Trades](#querytrades)
      * [Query 24 Hours Ticker](#query24hrsticker)
      * [Query History Trades](#queryhisttrades)
    * [Asset API List](#assetapilist)
      * [Query client and wallets](#clientwalletquery)
      * [Transfer self balance to parent or subclients](#walletransferout)
      * [Transfer from sub-client wallet](#walletransferin)
    * [Future Data Api List](#futureDataAPIList)
      * [Query Funding Fees History](#futureDataFundingFeesHist)
      * [Query Orders History](#futureDataOrdersHist)
      * [Query Orders By Ids](#futureDataOrdersByIds)
      * [Query Trades History](#futureDataTradesHist)
      * [Query Trading Fees History](#futureDataTradingFeesHist)
    * [Withdraw](#withdraw)
      * [Request withdraw](#requestwithdraw)
      * [Confirm withdraw](#confirmwithdraw)
      * [Cancel withdraw](#cancelwithdraw)
      * [List withdraw requests](#listwithdraw)
      * [Withdraw address management](#withdrawaddrmgmt)
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
    * [Subscribe Account-Order-Position (AOP)](#aopsub)
    * [Unsubscribe Account-Order-Position (AOP)](#aopunsub)
    * [Subscribe 24 Hours Ticker](#tickersub)
    * [Subscribe symbol price](#symbpricesub)

<a name="publicapi"/>

# Phemex Public API

<a name="general"/>

## General Public API Information

* Phemex provides HTTP Rest API for client to operate Orders, all endpoints return a JSON object.
* The default Rest API base endpoint is: **https://api.phemex.com**. The High rate limit Rest API base endpoint is: **https://vapi.phemex.com**. Or for the testnet is:  **https://testnet-api.phemex.com** 
* Phemex provides WebSocket API for client to receive market data, order and position updates.
* The WebSocket API url is: **wss://phemex.com/ws**. The High rate limit WebSocket API url is: **wss://vapi.phemex.com/ws**. Or for the testnet is:  **wss://testnet.phemex.com/ws** 

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

<a name="httprestheader"/>

## HTTP REST Request Header 

Every HTTP Rest Request must have the following Headers:
* x-phemex-access-token : This is ***API-KEY*** (id field) from Phemex site.
* x-phemex-request-expiry : This describes the Unix ***EPoch SECONDS*** to expire the request, normally it should be (Now() + 1 minute)
* x-phemex-request-signature : This is HMAC SHA256 signature of the http request. Secret is ***API Secret***, its formula is : HMacSha256( URL Path + QueryString + Expiry + body )

Optional Headers:
* x-phemex-request-tracing: a unique string to trace http-request, less than 40 bytes. This header is a must in resolving latency issues.

<a name="apiratelimits"/>

## API Rate Limits

* [Order spamming limitations](https://phemex.com/user-guides/order-spamming-limitations)
* RateLimit group of contract trading api is ***CONTRACT***.  
* RateLimit Explained [phemex ratelimit docs](/Generic-API-Info.en.md)
* Contract trading api response carries following headers.

```
X-RateLimit-Remaining-CONTRACT, # Remaining request permits in this minute
X-RateLimit-Capacity-CONTRACT, # Request ratelimit capacity
X-RateLimit-Retry-After-CONTRACT, # Reset timeout in seconds for current ratelimited user
```


<a name="securitytype"/>

## Endpoint security type

* Each API call must be signed and pass to server in HTTP header `x-phemex-request-signature`.
* Endpoints use `HMAC SHA256` signatures. The `HMAC SHA256 signature` is a keyed `HMAC SHA256` operation. Use your `apiSecret` as the key and the string `URL Path + QueryString + Expiry + body )` as the value for the HMAC operation.
* `apiSecret` = `Base64::urlDecode(API Secret)`
* The `signature` is **case sensitive**.

<a name="signatureexample1"/>

### Signature Example 1: HTTP GET Request

* API REST Request URL: https://api.phemex.com/accounts/accountPositions?currency=BTC
   * Request Path: /accounts/accountPositions
   * Request Query: currency=BTC
   * Request Body: <null>
   * Request Expiry: 1575735514
   * Signature: HMacSha256( /accounts/accountPositions + currency=BTC + 1575735514 )

<a name="signatureexample2"/>

### Singature Example 2: HTTP GET Request with multiple query string

* API REST Request URL: https://api.phemex.com/orders/activeList?ordStatus=New&ordStatus=PartiallyFilled&ordStatus=Untriggered&symbol=BTCUSD 
    * Request Path: /orders/activeList
    * Request Query: ordStatus=New&ordStatus=PartiallyFilled&ordStatus=Untriggered&symbol=BTCUSD
    * Request Body: <null>
    * Request Expire: 1575735951
    * Signature: HMacSha256(/orders/activeList + ordStatus=New&ordStatus=PartiallyFilled&ordStatus=Untriggered&symbol=BTCUSD + 1575735951)
    * signed string is `/orders/activeListordStatus=New&ordStatus=PartiallyFilled&ordStatus=Untriggered&symbol=BTCUSD1575735951`

<a name="signatureexample3"/>

### Signature Example 3: HTTP POST Request

* API REST Request URL: https://api.phemex.com/orders
   * Request Path: /orders
   * Request Query: <null>
   * Request Body: {"symbol":"BTCUSD","clOrdID":"uuid-1573058952273","side":"Sell","priceEp":93185000,"orderQty":7,"ordType":"Limit","reduceOnly":false,"timeInForce":"GoodTillCancel","takeProfitEp":0,"stopLossEp":0}
   * Request Expiry: 1575735514
   * Signature: HMacSha256( /orders + 1575735514 + {"symbol":"BTCUSD","clOrdID":"uuid-1573058952273","side":"Sell","priceEp":93185000,"orderQty":7,"ordType":"Limit","reduceOnly":false,"timeInForce":"GoodTillCancel","takeProfitEp":0,"stopLossEp":0})
   * signed string is `/orders1575735514{"symbol":"BTCUSD","clOrdID":"uuid-1573058952273","side":"Sell","priceEp":93185000,"orderQty":7,"ordType":"Limit","reduceOnly":false,"timeInForce":"GoodTillCancel","takeProfitEp":0,"stopLossEp":0}`


## Request/response field explained
<a name="fieldexplained"/>

### Price/Ratio/Value Scales
<a name="scalingfactors"/>

Fields with post-fix "Ep", "Er" or "Ev" have been scaled based on symbol setting.
* Fields with post-fix "Ep" are scaled prices, `priceScale` in [products](#queryproductinfo)
* Fields with post-fix "Er" are scaled ratios, `ratioScale` in [products](#queryproductinfo)
* Fields with post-fix "Ev" are scaled values, `valueScale` of `settleCurrency` in [products](#queryproductinfo) 

| Symbol | Price scale | Ratio scale | Value scale | settlement currency |
|--------|-------------|-------------|-------------|---------------------|
| BTCUSD | 10,000      | 100,000,000 | 100,000,000 | BTC                |
| cETHUSD| 10,000      | 100,000,000 | 100,000,000 | ETH                |
| uBTCUSD| 10,000      | 100,000,000 |      10,000 | USD                |
| ETHUSD | 10,000      | 100,000,000 |      10,000 | USD                |
| XRPUSD | 10,000      | 100,000,000 |      10,000 | USD                |
| LINKUSD| 10,000      | 100,000,000 |      10,000 | USD                |
| XTZUSD | 10,000      | 100,000,000 |      10,000 | USD                |
| LTCUSD | 10,000      | 100,000,000 |      10,000 | USD                |
| GOLDUSD| 10,000      | 100,000,000 |      10,000 | USD                |
| ADAUSD | 10,000      | 100,000,000 |      10,000 | USD                |
| BCHUSD | 10,000      | 100,000,000 |      10,000 | USD                |
| COMPUSD| 10,000      | 100,000,000 |      10,000 | USD                |
| ALGOUSD| 10,000      | 100,000,000 |      10,000 | USD                |
| YFIUSD | 10,000      | 100,000,000 |      10,000 | USD                |
| DOTUSD | 10,000      | 100,000,000 |      10,000 | USD                |
| UNIUSD | 10,000      | 100,000,000 |      10,000 | USD                |
| BATUSD | 10,000      | 100,000,000 |      10,000 | USD                |
| CHZUSD | 10,000      | 100,000,000 |      10,000 | USD                |
| MANAUSD| 10,000      | 100,000,000 |      10,000 | USD                |
| ENJUSD | 10,000      | 100,000,000 |      10,000 | USD                |
|SUSHIUSD| 10,000      | 100,000,000 |      10,000 | USD                |
| SNXUSD | 10,000      | 100,000,000 |      10,000 | USD                |
| GRTUSD | 10,000      | 100,000,000 |      10,000 | USD                |
| MKRUSD | 10,000      | 100,000,000 |      10,000 | USD                |
| TRXUSD | 10,000      | 100,000,000 |      10,000 | USD                |
| EOSUSD | 10,000      | 100,000,000 |      10,000 | USD                |
| ONTUSD | 10,000      | 100,000,000 |      10,000 | USD                |
| NEOUSD | 10,000      | 100,000,000 |      10,000 | USD                |
| ZECUSD | 10,000      | 100,000,000 |      10,000 | USD                |
| FILUSD | 10,000      | 100,000,000 |      10,000 | USD                |
| KSMUSD | 10,000      | 100,000,000 |      10,000 | USD                |
| XMRUSD | 10,000      | 100,000,000 |      10,000 | USD                |
| QTUMUSD| 10,000      | 100,000,000 |      10,000 | USD                |
| XLMUSD | 10,000      | 100,000,000 |      10,000 | USD                |
| ATOMUSD| 10,000      | 100,000,000 |      10,000 | USD                |
| LUNAUSD| 10,000      | 100,000,000 |      10,000 | USD                |
| SOLUSD | 10,000      | 100,000,000 |      10,000 | USD                |

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


* Execution instruction

| Execution instruction | description |
|------------|-------------|
| ReduceOnly | reduce position size, never increase position size |
| CloseOnTrigger | close the position  |


* Trigger source

| trigger | description |
|------------|-------------|
| ByMarkPrice | trigger by mark price|
| ByLastPrice | trigger by last price |


<a name="restapilist"/>

## REST API List

<a name="marketapilist"/>

### Market API List

<a name="queryproductinfo"/>

#### Query Product Information

* Request：

```
GET /public/products 
```

<a name="orderapilist"/>

### Trade API List

<a name="placeorderwithput"/>

#### Place order with argument in url query string

* Request

```
PUT /orders/create?clOrdID=<clOrdID>&symbol=<symbol>&reduceOnly=<reduceOnly>&closeOnTrigger=<closeOnTrigger>&orderQty=<orderQty>&displayQty=<displayQty>&ordType=<ordType>&priceEp=<priceEp>&side=<side>&text=<text>&timeInForce=<timeInForce>&stopPxEp=<stopPxEp>&takeProfitEp=<takeProfitEp>&stopLossEp=<stopLossEp>&pegOffsetValueEp=<pegOffsetValueEp>&pegPriceType=<pegPriceType>&trailingStopEp=<trailingStopEp>&triggerType=<triggerType>&tpTrigger=<tpTrigger>&tpSlTs=<tpSlTs>&slTrigger=<slTrigger>
```

| Field | Type | Required | Description | Possible values |
|-------|-------|--------|--------------|-----------------|
| symbol | String | Yes | Which symbol to place order | [Trading symbols](#symbpricesub) | 
| clOrdID | String | Yes | client order id, max length is 40| |
| side |  Enum | Yes | Order direction, Buy or Sell | Buy, Sell | 
| orderQty | Integer | Yes | Order quntity | |
| priceEp | Integer | - | Scaled price, required for limit order | | 
| ordType | Enum | - | default to Limit | Market, Limit, Stop, StopLimit, MarketIfTouched, LimitIfTouched| 
| stopPxEp | Integer | - | Trigger price for stop orders | |
| timeInForce | Enum | - | Time in force. default to GoodTillCancel | GoodTillCancel, ImmediateOrCancel, FillOrKill, PostOnly| 
| reduceOnly | Boolean | - | whether reduce position side only. Enable this flag, i.e. reduceOnly=true, position side won't change | true, false |
| closeOnTrigger | Boolean | - | implicitly reduceOnly, plus cancel other orders in the same direction(side) when necessary | true, false|
| triggerType | Enum | - | Trigger source, whether trigger by mark price, index price or last price | ByMarkPrice, ByLastPrice |
| takeProfitEp | Integer | - | Scaled take profit price | |
| stopLossEp | Integer | - | Scaled stop loss price | | 
| slTrigger | Enum | - |Trigger source, by mark-price or last-price | ByMarkPrice, ByLastPrice |
| tpTrigger | Enum | - | Trigger source, by mark-price or last-price | ByMarkPrice, ByLastPrice |
| pegOffsetValueEp | Integer | - | Trailing offset from current price. Negative value when position is long, positive when position is short | |
| pegPriceType | Enum | - | Trailing order price type |TrailingStopPeg, TrailingTakeProfitPeg |


* HTTP Response:

```
{
    "code": 0,
        "msg": "",
        "data": {
            "bizError": 0,
            "orderID": "ab90a08c-b728-4b6b-97c4-36fa497335bf",
            "clOrdID": "137e1928-5d25-fecd-dbd1-705ded659a4f",
            "symbol": "BTCUSD",
            "side": "Sell",
            "actionTimeNs": 1580547265848034600,
            "transactTimeNs": 0,
            "orderType": null,
            "priceEp": 98970000,
            "price": 9897,
            "orderQty": 1,
            "displayQty": 1,
            "timeInForce": null,
            "reduceOnly": false,
            "stopPxEp": 0,
            "closedPnlEv": 0,
            "closedPnl": 0,
            "closedSize": 0,
            "cumQty": 0,
            "cumValueEv": 0,
            "cumValue": 0,
            "leavesQty": 1,
            "leavesValueEv": 10104,
            "leavesValue": 0.00010104,
            "stopPx": 0,
            "stopDirection": "UNSPECIFIED",
            "ordStatus": "Created"
        }
}
```

* Important fields description
   * Order average filled price,  inverse contract:`avgPrice = (cumQty/cumValueEv)/contractSize`; linear contract: `avgPrice = (cumValueEv/cumQty)/contractSize`. `contractSize` is fixed in [product](#marketapilist) api.

| Field | Description |
|------|----------|
| bizError | bizError = 0 means processing normally, non-zero values mean wrong state. Separate section to explain these errors; `code` in response is equal to bizError if response contains only one order |
| cumQty | cumulative filled order quantity |
| cumValueEv | cumulative filled order value (scaled) |
| leavesQty | unfilled order quantity | 
| leavesValueEv | unfilled order value |

<a name="orderexample"/>

* More order type examples

  * Stop-loss orders (ordType = Stop/StopLimit) and Take-profit order (ordType = MarketIfTouched/LimitIfTouched)
  * Stop-loss order is triggered when price moves against order-side(buy/sell), while Take-profit order is triggered when price moves in profitable direction to order-side(buy/sell).

    | ordType                         | side | parameter requirements            | trigger condition |
    |---------------------------------|------|-----------------------------------|-------------------|
    | Stop/StopLimit                  | Sell |  stopPxEp < last-price/mark-price | last/mark-price <= stopPxEp  |
    | Stop/StopLimit                  | Buy  |  stopPxEp > last-price/mark-price | last/mark-price >= stopPxEp  |
    | MarketIfTouched/LimitIfTouched  | Sell |  stopPxEp > last-price/mark-price | last/mark-price >= stopPxEp  |
    | MarketIfTouched/LimitIfTouched  | Buy  |  stopPxEp < last-price/mark-price | last/mark-price <= stopPxEp  |

  * StopLoss Sell order, triggered order is placed as limit order (Assume current last-price is 30k)

  ```
  {
      "clOrdID": "stop-loss-order-then-limit",
      "symbol": "BTCUSD",
      "side": "Sell",
      "ordType": "StopLimit",
      "triggerType": "ByMarkPrice",
      "stopPxEp": "299550000", // "trigger price, when ordType= Stop/StopLimit and side = Sell, stopPxEp must less than last-price"
      "priceEp": "299650000", // "when ordType = StopLimit, priceEp is required, when ordType = Stop, priceEp is not required "
      "orderQty": 10000
  }

  ```

  * StopLoss Buy order, triggered order is placed as market order (Assume current last-price is 30k)

  ```
  {
      "clOrdID": "stop-loss-order-then-market",
      "symbol": "BTCUSD",
      "side": "Buy",
      "ordType": "Stop",
      "triggerType": "ByMarkPrice",
      "stopPxEp": "333550000", // "trigger price, when ordType = Stop/StopLimit and side = Buy, stopPxEp must be larger than last-price"
      "priceEp": "0",// not required 
      "orderQty": 10000
  }

  ```

  * Take-profit Sell order, triggered order is placed as limit order (Assume current last-price is 30k)

  ```
  {
      "clOrdID": "take-profit-order-then-limit",
      "symbol": "BTCUSD",
      "side": "Sell",
      "ordType": "LimitIfTouched",
      "triggerType": "ByMarkPrice",
      "stopPxEp": "333550000", // "trigger price, when ordType = LimitIfTouched/MarketIfTouched and side = Sell, stopPxEp is larger than last-price"
      "priceEp": "334550000", // "when ordType = LimitIfTouched, priceEp is required, when ordType = MarketIfTouched, priceEp is not required "
      "orderQty": 10000
  }
  ```

  * Take-profit Buy order, triggered order is placed as market order (Assume current last-price is 30k)

  ```
  {
      "clOrdID": "take-profit-order-then-market",
      symbol": "BTCUSD",
      "side": "Buy",
      "ordType": "MarketIfTouched",
      "triggerType": "ByLastPrice",
      "stopPxEp": "299550000", // "when ordType = LimitIfTouched/MarketIfTouched and side = Buy, stopPxEp is less than last-price"
      "priceEp": "0", // "not required"
      "orderQty": 10000
  }
  ```

  * Place a order with stop-loss and take-profit

  ```
  {
      "clOrdID": "order-with-take-profit-stop-loss",
      "symbol": "BTCUSD",
      "side": "Buy",
      "priceEp": 300000000,
      "orderQty": 1000,
      "ordType": "Limit",
      "takeProfitEp": 3111100000,
      "tpTrigger": "ByLastPrice",
      "stopLossEp": "299990000",
      "slTrigger": "ByMarkPrice"
  }
  ```

  * Trailing stop order(Assume current position is long, current last-price is 32k)

  ```
  {
  "symbol": "BTCUSD",
  "side": "Sell", // assume current position is long
  "ordType": "Stop",
  "orderQty": 0,
  "priceEp": 0,
  "triggerType": "ByLastPrice",
  "stopPxEp": 315000, // "if position is long, this value should be less than last-price; if position is short, this value is larger than last-price",
  "timeInForce": "ImmediateOrCancel",
  "closeOnTrigger": true, 
  "pegPriceType": "TrailingStopPeg",
  "pegOffsetValueEp": -10000000, // retraces by $1000.0 from the optimal price, sign is opposite to position side, i.e. Long Position => negative sign; Shot Position => positive sign
  "clOrdID": "cl-order-id"
  }
  ```

  * Trailing stop order with activiation price

  ```
  {
  "symbol": "BTCUSD",
  "side": "Sell",
  "ordType": "Stop",
  "orderQty": 0,
  "priceEp": 0,
  "triggerType": "ByLastPrice",
  "stopPxEp": 340000000, // activation price of this trailing order, this value should be larger than last-price
  "timeInForce": "ImmediateOrCancel",
  "closeOnTrigger": true,
  "pegPriceType": "TrailingTakeProfitPeg",
  "pegOffsetValueEp": -10000000,//retraces by $1000.0 from the optimal price, sign is opposite to position side, i.e. Long Position => negative sign; Shot Position => positive sign  
  "clOrdID": "cl-order-id"
  }
  ```

<a name="placeorder"/>

#### Place Order 

* HTTP Request:

```
POST /orders

{
  "actionBy": "FromOrderPlacement",
  "symbol": "BTCUSD",
  "clOrdID": "uuid-1573058952273",
  "side": "Sell",
  "priceEp": 93185000,
  "orderQty": 7,
  "ordType": "Limit",
  "reduceOnly": false,
  "triggerType": "UNSPECIFIED",
  "pegPriceType": "UNSPECIFIED",
  "timeInForce": "GoodTillCancel",
  "takeProfitEp": 0,
  "stopLossEp": 0,
  "pegOffsetValueEp": 0,
  "pegPriceType": "UNSPECIFIED"
}
```

  * Fields are the same as [above place-order](#placeorderwithput)
<a name="amendorder"/>


#### Amend order by orderID

* Request

```
PUT
/orders/replace?symbol=<symbol>&orderID=<orderID>&origClOrdID=<origClOrdID>&clOrdID=<clOrdID>&price=<price>&priceEp=<priceEp>&orderQty=<orderQty>&stopPx=<stopPx>&stopPxEp=<stopPxEp>&takeProfit=<takeProfit>&takeProfitEp=<takeProfitEp>&stopLoss=<stopLoss>&stopLossEp=<stopLossEp>&pegOffsetValueEp=<pegOffsetValueEp>&pegPriceType=<pegPriceType>
```

| Field  | Required | Description |
|--------|----------|-------------|
| symbol | Yes  | order symbol, cannot be changed|
| orderID| Yes  |order id, cannot be changed |
| origClOrdID | No | original clOrderID |
| clOrdID| No | new clOrdID |
| price  | No | new order price |
| priceEp| No | new order price with scale |
| orderQty | No | new orderQty |
| stopPx | No | new stop price |
| stopPxEp | No | new stop price with scale |
| takeProfit | No | new stop profit price |
| takeProfitEp | No | new stop profit price with scale |
| stopLoss | No | new stop loss price |
| stopLossEp | No | new stop loss price with scale |
| pegOffsetValueEp | No | New trailing offset |
| pegPriceType | No | New peg price type |


* Response
   * amended order 


<a name="cancelsingleorder"/>

#### Cancel Single Order

* Request
```
DELETE /orders/cancel?symbol=<symbol>&orderID=<orderID>
```

* Response
   * Full Order
   * This response means cancel operations succeeded not the order is canceled. One needs to query to order to determine whether this order has been cancelled or not.

```
{
    "code": 0,
        "msg": "",
        "data": {
            "bizError": 0,
            "orderID": "2585817b-85df-4dea-8507-5db1920b9954",
            "clOrdID": "4b19fd1e-a1a7-2986-d02a-0288ad5137d4",
            "symbol": "BTCUSD",
            "side": "Buy",
            "actionTimeNs": 1580533179846642700,
            "transactTimeNs": 1580532966633276200,
            "orderType": null,
            "priceEp": 80040000,
            "price": 8004,
            "orderQty": 1,
            "displayQty": 1,
            "timeInForce": null,
            "reduceOnly": false,
            "stopPxEp": 0,
            "closedPnlEv": 0,
            "closedPnl": 0,
            "closedSize": 0,
            "cumQty": 0,
            "cumValueEv": 0,
            "cumValue": 0,
            "leavesQty": 1,
            "leavesValueEv": 12493,
            "leavesValue": 0.00012493,
            "stopPx": 0,
            "stopDirection": "UNSPECIFIED",
            "ordStatus": "New"
        }
}

```


<a name="cancelorder"/>

#### Bulk Cancel Orders

* Request
```
DELETE /orders?symbol=<symbol>&orderID=<orderID1>,<orderID2>,<orderID3>
```

* Response
   * Canceled orders

<a name="cancelall"/>

#### Cancel All Orders
   * In order to cancel all orders, include conditional order and active order, one must invoke this API twice with different arguments.
   * `untriggered=false` to cancel active order including triggerred conditional order.
   * `untriggered=true` to cancel conditional order, the order is not triggerred.

* Request

```
DELETE /orders/all?symbol=<symbol>&untriggered=<untriggered>&text=<text>
```

| Field       | Type   | Required  | Description                    | Possible values         |
|-------------|--------|-----------|--------------------------------|-------------------------|
| symbol      | String | Yes       | which Symbol to cancel         | [Trading symbols](#symbpricesub) |
| untriggered | Boolean| No        | default to false, default cancel non-conditional order; if intending to cancel conditional order, set this to true| true,false|
| text        | comments| No       | comments of this operation, limited to 40 characters  |  |

* Response
   * `data` part of response is subject to change, ***DONT*** rely on it

<a name="querytradeaccount"/>

#### Query trading account and positions

* Request

```
GET /accounts/accountPositions?currency=<currency>
```
| Field       | Type   | Description                                | Possible values |
|-------------|--------|--------------------------------------------|--------------|
| currency | string | in url query parameter. which trading account | BTC,USD |

* Response

```
{
    "code": 0,
        "msg": "",
        "data": {
            "account": {
                "accountId": 0,
                "currency": "BTC",
                "accountBalanceEv": 0,
                "totalUsedBalanceEv": 0
            },
            "positions": [
            {
                "accountID": 0,
                "symbol": "BTCUSD",
                "currency": "BTC",
                "side": "None",
                "positionStatus": "Normal",
                "crossMargin": false,
                "leverageEr": 0,
                "leverage": 0,
                "initMarginReqEr": 0,
                "initMarginReq": 0.01,
                "maintMarginReqEr": 500000,
                "maintMarginReq": 0.005,
                "riskLimitEv": 10000000000,
                "riskLimit": 100,
                "size": 0,
                "value": 0,
                "valueEv": 0,
                "avgEntryPriceEp": 0,
                "avgEntryPrice": 0,
                "posCostEv": 0,
                "posCost": 0,
                "assignedPosBalanceEv": 0,
                "assignedPosBalance": 0,
                "bankruptCommEv": 0,
                "bankruptComm": 0,
                "bankruptPriceEp": 0,
                "bankruptPrice": 0,
                "positionMarginEv": 0,
                "positionMargin": 0,
                "liquidationPriceEp": 0,
                "liquidationPrice": 0,
                "deleveragePercentileEr": 0,
                "deleveragePercentile": 0,
                "buyValueToCostEr": 1150750,
                "buyValueToCost": 0.0115075,
                "sellValueToCostEr": 1149250,
                "sellValueToCost": 0.0114925,
                "markPriceEp": 93169002,
                "markPrice": 9316.9002,
                "markValueEv": 0,
                "markValue": null,
                "estimatedOrdLossEv": 0,
                "estimatedOrdLoss": 0,
                "usedBalanceEv": 0,
                "usedBalance": 0,
                "takeProfitEp": 0,
                "takeProfit": null,
                "stopLossEp": 0,
                "stopLoss": null,
                "realisedPnlEv": 0,
                "realisedPnl": null,
                "cumRealisedPnlEv": 0,
                "cumRealisedPnl": null
            }
            ]
        }
}
```

<b>Note</b> `unRealizedPnlEv` needs to be calculated in client side with latest `markPrice`, formula is as below.

```
Inverse contract: unRealizedPnl = (posSize/contractSize) / avgEntryPrice - (posSize/contractSize) / markPrice)

Linear contract:  unRealizedPnl = (posSize*contractSize) * markPrice - (posSize*contractSize) * avgEntryPrice

posSize is a signed vaule. contractSize is a fixed value.

```

#### Query trading account and positions with unrealized-pnl

<a name="queryPosWithPnl"/>

Below API presents unrealized pnl at `markprice` of positions with **considerable** cost, thus its [ratelimit](/Generic-API-Info.en.md#contractAPIGroup) weight is very high.

* Request

```
GET /accounts/positions?currency=<currency>

```

* Response

```
{
  "code": 0,
  "msg": "",
  "data": {
    "account": {
      "accountId": 111100001,
      "currency": "BTC",
      "accountBalanceEv": 879599942377,
      "totalUsedBalanceEv": 285,
      "bonusBalanceEv": 0
    },
    "positions": [
      {
        "accountID": 111100001,
        "symbol": "BTCUSD",
        "currency": "BTC",
        "side": "Buy",
        "positionStatus": "Normal",
        "crossMargin": false,
        "leverageEr": 0,
        "initMarginReqEr": 1000000,
        "maintMarginReqEr": 500000,
        "riskLimitEv": 10000000000,
        "size": 5,
        "valueEv": 26435,
        "avgEntryPriceEp": 189143181,
        "posCostEv": 285,
        "assignedPosBalanceEv": 285,
        "bankruptCommEv": 750000,
        "bankruptPriceEp": 5000,
        "positionMarginEv": 879599192377,
        "liquidationPriceEp": 5000,
        "deleveragePercentileEr": 0,
        "buyValueToCostEr": 1150750,
        "sellValueToCostEr": 1149250,
        "markPriceEp": 238287555,
        "markValueEv": 0,
        "unRealisedPosLossEv": 0,
        "estimatedOrdLossEv": 0,
        "usedBalanceEv": 285,
        "takeProfitEp": 0,
        "stopLossEp": 0,
        "cumClosedPnlEv": -8913353,
        "cumFundingFeeEv": 123996,
        "cumTransactFeeEv": 940245,
        "realisedPnlEv": 0,
        "unRealisedPnlEv": 5452,
        "cumRealisedPnlEv": 0
      }
    ]
  }
}

```

<b>Note</b> Highly recommend calculating `unRealizedPnlEv` in client side with latest `markPrice` to avoid ratelimit penalty.

<a name="changeleverage"/>

#### Change leverage

* Request

```
PUT /positions/leverage?symbol=<symbol>&leverage=<leverage>&leverageEr=<leverageEr>
```

| Field                | Type   | Description                                | Possible values |
|----------------------|--------|--------------------------------------------|--------------|
| symbol               | string | which postion needs to change              | [Trading symbols](#symbpricesub) |
| leverage             | integer   | unscaled leverage                       |  |
| leverageEr           | integer   | ratio scaled leverage, leverage wins when both leverage and leverageEr provided|  |


* Response

```
{
    "code": 0,
    "msg": "OK"
}
```

<a name = "changerisklimit"/>

#### Change position risklimit

* Request

```
PUT /positions/riskLimit?symbol=<symbol>&riskLimit=<riskLimit>&riskLimitEv=<riskLimitEv>
```
| Field                | Type   | Description                                | Possible values |
|----------------------|--------|--------------------------------------------|--------------|
| symbol               | string | which postion needs to change              | [Trading symbols](#symbpricesub) |
| riskLimit             | integer   | unscaled value, reference BTC/USD value scale   |  |
| riskLimitEv           | integer   | value scaled risklimit, riskLimitEv wins when both riskLimit and riskLimitEv provided|  |

<a name="assignposbalance"/>

#### Assign position balance in isolated marign mode

* Request

***This API is POST***

```
POST /positions/assign?symbol=<symbol>&posBalance=<posBalance>&posBalanceEv=<posBalanceEv>

```
| Field                | Type   | Description                                | Possible values |
|----------------------|--------|--------------------------------------------|--------------|
| symbol               | string | which postion needs to change              | [Trading symbols](#symbpricesub) |
| posBalance             | integer   | unscaled value                       |  |
| posBalanceEv           | integer   | value scaled for position balance, posBalanceEv wins when both posBalance and posBalanceEv provided|  |

<a name="queryopenorder"/>

#### Query open orders by symbol

   * Order status includes `New`, `PartiallyFilled`, `Filled`, `Canceled`, `Rejected`, `Triggered`, `Untriggered`;
   * Open order status includes `New`, `PartiallyFilled`, `Untriggered`;

* Request

```
GET /orders/activeList?symbol=<symbol>
```

| Field                | Type   | Description                                | Possible values |
|----------------------|--------|--------------------------------------------|--------------|
| symbol | String | which symbol needs to query | [Trading symbols](#symbpricesub)  |


* Response
   * Full order

```
{
    "code": 0,
        "msg": "",
        "data": {
            "rows": [
            {
                "bizError": 0,
                "orderID": "9cb95282-7840-42d6-9768-ab8901385a67",
                "clOrdID": "7eaa9987-928c-652e-cc6a-82fc35641706",
                "symbol": "BTCUSD",
                "side": "Buy",
                "actionTimeNs": 1580533011677666800,
                "transactTimeNs": 1580533011677666800,
                "orderType": null,
                "priceEp": 84000000,
                "price": 8400,
                "orderQty": 1,
                "displayQty": 1,
                "timeInForce": null,
                "reduceOnly": false,
                "stopPxEp": 0,
                "closedPnlEv": 0,
                "closedPnl": 0,
                "closedSize": 0,
                "cumQty": 0,
                "cumValueEv": 0,
                "cumValue": 0,
                "leavesQty": 0,
                "leavesValueEv": 0,
                "leavesValue": 0,
                "stopPx": 0,
                "stopDirection": "Falling",
                "ordStatus": "Untriggered"
            },
            {
                "bizError": 0,
                "orderID": "93397a06-e76d-4e3b-babc-dff2696786aa",
                "clOrdID": "71c2ab5d-eb6f-0d5c-a7c4-50fd5d40cc50",
                "symbol": "BTCUSD",
                "side": "Sell",
                "actionTimeNs": 1580532983785506600,
                "transactTimeNs": 1580532983786370300,
                "orderType": null,
                "priceEp": 99040000,
                "price": 9904,
                "orderQty": 1,
                "displayQty": 1,
                "timeInForce": null,
                "reduceOnly": false,
                "stopPxEp": 0,
                "closedPnlEv": 0,
                "closedPnl": 0,
                "closedSize": 0,
                "cumQty": 0,
                "cumValueEv": 0,
                "cumValue": 0,
                "leavesQty": 1,
                "leavesValueEv": 10096,
                "leavesValue": 0.00010096,
                "stopPx": 0,
                "stopDirection": "UNSPECIFIED",
                "ordStatus": "New"
            },
            {
                "bizError": 0,
                "orderID": "2585817b-85df-4dea-8507-5db1920b9954",
                "clOrdID": "4b19fd1e-a1a7-2986-d02a-0288ad5137d4",
                "symbol": "BTCUSD",
                "side": "Buy",
                "actionTimeNs": 1580532966629408500,
                "transactTimeNs": 1580532966633276200,
                "orderType": null,
                "priceEp": 80040000,
                "price": 8004,
                "orderQty": 1,
                "displayQty": 1,
                "timeInForce": null,
                "reduceOnly": false,
                "stopPxEp": 0,
                "closedPnlEv": 0,
                "closedPnl": 0,
                "closedSize": 0,
                "cumQty": 0,
                "cumValueEv": 0,
                "cumValue": 0,
                "leavesQty": 1,
                "leavesValueEv": 12493,
                "leavesValue": 0.00012493,
                "stopPx": 0,
                "stopDirection": "UNSPECIFIED",
                "ordStatus": "New"
            }
            ]
        }
}

```

<a name="queryorder"/>

#### Query closed orders by symbol

* This API is for ***closed*** orders. For open orders, please use [open order query](#queryopenorder)

* Request

```
GET /exchange/order/list?symbol=<symbol>&start=<start>&end=<end>&offset=<offset>&limit=<limit>&ordStatus=<ordStatus>&withCount=<withCount>
```

| Field                | Type   | Description                                | Possible values |
|----------------------|--------|--------------------------------------------|--------------|
| symbol | String | which symbol needs to query | [Trading symbols](#symbpricesub) |
| start  | Integer | start time range, Epoch millis | |
| end  | Integer | end time range, Epoch millis | |
| offset | Integer | offset to resultset | | 
| limit | Integer | limit of resultset  | | 
| ordStatus | String | order status list filter | New, PartiallyFilled, Untriggered, Filled, Canceled | 

* Response
   * sample response

```
{
    "code": 0,
        "msg": "OK",
        "data": {
            "total": 39,
            "rows": [
            {
                "orderID": "7d5a39d6-ff14-4428-b9e1-1fcf1800d6ac",
                "clOrdID": "e422be37-074c-403d-aac8-ad94827f60c1",
                "symbol": "BTCUSD",
                "side": "Sell",
                "orderType": "Limit",
                "actionTimeNs": 1577523473419470300,
                "priceEp": 75720000,
                "price": null,
                "orderQty": 12,
                "displayQty": 0,
                "timeInForce": "GoodTillCancel",
                "reduceOnly": false,
                "takeProfitEp": 0,
                "takeProfit": null,
                "stopLossEp": 0,
                "closedPnlEv": 0,
                "closedPnl": null,
                "closedSize": 0,
                "cumQty": 0,
                "cumValueEv": 0,
                "cumValue": null,
                "leavesQty": 0,
                "leavesValueEv": 0,
                "leavesValue": null,
                "stopLoss": null,
                "stopDirection": "UNSPECIFIED",
                "ordStatus": "Canceled",
                "transactTimeNs": 1577523473425416400
            },
            {
                "orderID": "b63bc982-be3a-45e0-8974-43d6375fb626",
                "clOrdID": "uuid-1577463487504",
                "symbol": "BTCUSD",
                "side": "Sell",
                "orderType": "Limit",
                "actionTimeNs": 1577963507348468200,
                "priceEp": 71500000,
                "price": null,
                "orderQty": 700,
                "displayQty": 700,
                "timeInForce": "GoodTillCancel",
                "reduceOnly": false,
                "takeProfitEp": 0,
                "takeProfit": null,
                "stopLossEp": 0,
                "closedPnlEv": 0,
                "closedPnl": null,
                "closedSize": 0,
                "cumQty": 700,
                "cumValueEv": 9790209,
                "cumValue": null,
                "leavesQty": 0,
                "leavesValueEv": 0,
                "leavesValue": null,
                "stopLoss": null,
                "stopDirection": "UNSPECIFIED",
                "ordStatus": "Filled",
                "transactTimeNs": 1578026629824704800
            }
            ]
        }
}
```

<a name="queryorderbyid"/>

#### Query user order by orderID or Query user order by client order ID
* Request

```
GET /exchange/order?symbol=<symbol>&orderID=<orderID1,orderID2>
GET /exchange/order?symbol=<symbol>&clOrdID=<clOrdID1,clOrdID2>
```

* Response

```
{
    "code": 0,
        "msg": "OK",
        "data": [
        {
            "orderID": "7d5a39d6-ff14-4428-b9e1-1fcf1800d6ac",
            "clOrdID": "e422be37-074c-403d-aac8-ad94827f60c1",
            "symbol": "BTCUSD",
            "side": "Sell",
            "orderType": "Limit",
            "actionTimeNs": 1577523473419470300,
            "priceEp": 75720000,
            "price": null,
            "orderQty": 12,
            "displayQty": 0,
            "timeInForce": "GoodTillCancel",
            "reduceOnly": false,
            "takeProfitEp": 0,
            "takeProfit": null,
            "stopLossEp": 0,
            "closedPnlEv": 0,
            "closedPnl": null,
            "closedSize": 0,
            "cumQty": 0,
            "cumValueEv": 0,
            "cumValue": null,
            "leavesQty": 0,
            "leavesValueEv": 0,
            "leavesValue": null,
            "stopLoss": null,
            "stopDirection": "UNSPECIFIED",
            "ordStatus": "Canceled",
            "transactTimeNs": 1577523473425416400
        },
        {
            "orderID": "b63bc982-be3a-45e0-8974-43d6375fb626",
            "clOrdID": "uuid-1577463487504",
            "symbol": "BTCUSD",
            "side": "Sell",
            "orderType": "Limit",
            "actionTimeNs": 1577963507348468200,
            "priceEp": 71500000,
            "price": null,
            "orderQty": 700,
            "displayQty": 700,
            "timeInForce": "GoodTillCancel",
            "reduceOnly": false,
            "takeProfitEp": 0,
            "takeProfit": null,
            "stopLossEp": 0,
            "closedPnlEv": 0,
            "closedPnl": null,
            "closedSize": 0,
            "cumQty": 700,
            "cumValueEv": 9790209,
            "cumValue": null,
            "leavesQty": 0,
            "leavesValueEv": 0,
            "leavesValue": null,
            "stopLoss": null,
            "stopDirection": "UNSPECIFIED",
            "ordStatus": "Filled",
            "transactTimeNs": 1578026629824704800
        }
    ]
}
```

<a name="querytrade"/>

#### Query user trade

* Request

```
GET /exchange/order/trade?symbol=<symbol>&start=<start>&end=<end>&limit=<limit>&offset=<offset>&withCount=<withCount>
```

| Field     | Type     | Required | Description                                                     | Possible Values                 | 
|-----------|----------|----------|-----------------------------------------------------------------|---------------------------------|
| symbol    | String   | Yes      | Trading symbol                                                  | BTCUSD, ETHUSD ...              |
| tradeType | String   | No       | Trade type of execution order                                   | Trade,Funding,AdlTrade,LiqTrade |
| start     | Long     | No       | Epoch time in milli-seconds of range start                      | --                              |
| end       | Long     | No       | Epoch time in milli-seconds of range end                        | --                              |
| limit     | Integer  | No       | The expected count of returned data-set. Default to 50          | --                              |
| offset    | Integer  | No       | Offset of total dataset in a range                              | --                              |
| withCount | Boolean  | No       | A flag to tell if the count of total result set is required     | --                              |



* Response
  * Response of this API includes normal trade, funding records,  liquidation, ADL trades,etc. `tradeType` can distiguish these types.
  * Sample trade response

```
{
    "code": 0,
        "msg": "OK",
        "data": {
            "total": 79,
            "rows": [
            {
                "transactTimeNs": 1578026629824704800,
                "symbol": "BTCUSD",
                "currency": "BTC",
                "action": "Replace",
                "side": "Sell",
                "tradeType": "Trade",
                "execQty": 700,
                "execPriceEp": 71500000,
                "orderQty": 700,
                "priceEp": 71500000,
                "execValueEv": 9790209,
                "feeRateEr": -25000,
                "execFeeEv": -2447,
                "ordType": "Limit",
                "execID": "b01671a1-5ddc-5def-b80a-5311522fd4bf",
                "orderID": "b63bc982-be3a-45e0-8974-43d6375fb626",
                "clOrdID": "uuid-1577463487504",
                "execStatus": "MakerFill"
            },
            {
                "transactTimeNs": 1578009600000000000,
                "symbol": "BTCUSD",
                "currency": "BTC",
                "action": "SettleFundingFee",
                "side": "Buy",
                "tradeType": "Funding",
                "execQty": 700,
                "execPriceEp": 69473435,
                "orderQty": 0,
                "priceEp": 0,
                "execValueEv": 10075793,
                "feeRateEr": 4747,
                "execFeeEv": 479,
                "ordType": "UNSPECIFIED",
                "execID": "381fbe21-a116-472d-a547-9e2368dcc194",
                "orderID": "00000000-0000-0000-0000-000000000000",
                "clOrdID": "SettlingFunding",
                "execStatus": "Init"

            }
            ]
        }
}

```

* Possible trade types

|TradeTypes| Description |
|---------|--------------|
| Trade | Normal trades |
| Funding | Funding on positions |
| AdlTrade |  Auto-delevearage trades |
| LiqTrade | Liquidation trades |


<a name="mdapilist"/>

### Market Data API List

<a name="queryorderbook"/>

#### Query Order Book

* Request：
```
GET /md/orderbook?symbol=<symbol>
```

| Field       | Type   | Description                                | Possible values |
|-------------|--------|--------------------------------------------|--------------|
| symbol      | String | Contract symbol name                       | [Trading symbols](#symbpricesub) |

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
| symbol      | String | Contract symbol name                       | [Trading symbols](#symbpricesub) |

* Sample：
```
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
    "symbol": "BTCUSD",
    "type": "snapshot"
  }
}
```

<a name="querykline"/>

### Query kline

```
GET /exchange/public/md/kline?symbol=<symbol>&to=<to>&from=<from>&resolution=<resolution>

```

* Response

```
{
"code": 0,
"msg": "OK",
"data": {
"total": -1,
"rows": [[<timestamp>, <interval>, <last_close>, <open>, <high>, <low>, <close>, <volume>, <turnover>], [...]]
}
}
```

| Field       | Type    | Required    | Description            | Possible Values                        |
|-------------|---------|-------------|------------------------|----------------------------------------|
|symbol       | String  | Yes         | symbol name            | BTCUSD,ETHUSD,uBTCUSD,cETHUSD,XRPUSD...| 
| from        | Integer | Yes         | start time in seconds  |                                        |
| to          | Integer | Yes         | end time in seconds    |                                        | 
| resolution  | Integer | Yes         | kline interval         | describled as below                    |

|resolution|Description |
|----------|------------|
|60|MINUTE_1|
|300|MINUTE_5|
|900|MINUTE_15|
|1800|MINUTE_30|
|3600|HOUR_1|
|14400|HOUR_4|
|86400|DAY_1|
|604800|WEEK_1|
|2592000|MONTH_1|
|7776000|SEASON_1|
|31104000|YEAR_1|

<a name="querytrades"/>

#### Query Recent Trades

* Request：
```
GET /md/trade?symbol=<symbol>
```

| Field       | Type   | Description                                | Possible values |
|-------------|--------|--------------------------------------------|--------------|
| symbol      | String | Contract symbol name                       | [Trading symbols](#symbpricesub) |

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
| symbol      | String | Contract symbol name                       | [Trading symbols](#symbpricesub) |

* Sample：
```
GET /md/trade?symbol=BTCUSD

{
  "error": null,
  "id": 0,
  "result": {
    "sequence": 15934323,
    "symbol": "BTCUSD",
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

#### Query 24 Hours Ticker

* Request：
```
GET /md/ticker/24hr?symbol=<symbol>
```

| Field       | Type   | Description                                | Possible values |
|-------------|--------|--------------------------------------------|--------------|
| symbol      | String | Contract symbol name                       | [Trading symbols](#symbpricesub) |

* Response:
```
{
  "error": null,
  "id": 0,
  "result": {
    "open": <open priceEp>,
    "high": <high priceEp>,
    "low": <low priceEp>,
    "close": <close priceEp>,
    "indexPrice": <index priceEp>,
    "markPrice": <mark priceEp>,
    "openInterest": <open interest>,
    "fundingRate": <funding rateEr>,
    "predFundingRate": <predicated funding rateEr>,
    "symbol": "<symbol>",
    "turnover": <turnoverEv>,
    "volume": <volume>,
    "timestamp": <timestamp>
  }
}
```

| Field         | Type   | Description                                | Possible values |
|---------------|--------|--------------------------------------------|--------------|
| open priceEp  | Integer| The scaled open price in last 24 hours     |              |
| high priceEp  | Integer| The scaled highest price in last 24 hours  |              |
| low priceEp   | Integer| The scaled lowest price in last 24 hours   |              |
| close priceEp | Integer| The scaled close price in last 24 hours    |              |
| index priceEp | Integer| Scaled index price                         |              |
| mark priceEp  | Integer| Scaled mark price                          |              |
| open interest | Integer| current open interest                      |              |
| funding rateEr| Integer| Scaled funding rate                        |              |
| predicated funding rateEr| Integer| Scaled predicated funding rate  |              |
| timestamp     | Integer| Timestamp in nanoseconds                   |              |
| symbol        | String | Contract symbol name                       | [Trading symbols](#symbpricesub) |
| turnoverEv    | Integer| The scaled turnover value in last 24 hours |              |
| volume        | Integer| Symbol trade volume in last 24 hours       |              |

* Sample：
```
GET /md/ticker/24hr?symbol=BTCUSD

{
  "error": null,
  "id": 0,
  "result": {
    "close": 87425000,
    "fundingRate": 10000,
    "high": 92080000,
    "indexPrice": 87450676,
    "low": 87130000,
    "markPrice": 87453092,
    "open": 90710000,
    "openInterest": 7821141,
    "predFundingRate": 7609,
    "symbol": "BTCUSD",
    "timestamp": 1583646442444219017,
    "turnover": 1399362834123,
    "volume": 125287131
  }
}
```

<a name="queryhisttrades"/>

#### Query History Trades By symbol
   * Query History trades by symbol
   * RateLimit of this api is 5 per second

```
GET /exchange/public/nomics/trades?market=<symbol>&since=<since>
```

| Field  | Type     | Description                                                       | Possible values                   |
|--------|----------|-------------------------------------------------------------------|-----------------------------------|
| market | String   | the market of symbol                                              |[Trading symbols](#scalingfactors) |
| since  | String   | Last id of response field, 0-0-0 is from the very initial trade   |                                   |

* Response

```
{
    "code": 0,
        "data": [
        {
            "id": "string",
            "amount_quote": "string",
            "price": "string",
            "side": "string",
            "timestamp": "string",
            "type": "string"
        }
        ],
        "msg": "string"
}
```

* Sample

```
{
    "code": 0,
        "msg": "OK",
        "data": [
        {
            "id": "1183-3-2",
            "timestamp": "2019-11-24T08:32:17.046Z",
            "price": "7211.00000000",
            "amount_quote": "1",
            "side": "sell",
            "type": "limit"
        },
        {
            "id": "1184-2-1",
            "timestamp": "2019-11-24T08:32:17.047Z",
            "price": "7211.00000000",
            "amount_quote": "1",
            "side": "buy",
            "type": "limit"
        }]
}

```

<a name="assetapilist"/>

### Asset API list
   * Asset includes BTC in wallets, BTC in btc-trading account, USD in usd-trading account.
   * In wallet level, Main/parent client can transfer BTC between Sub-client and main/parent client.
   * In wallet level, Sub client can *only* transfer self BTC to main/parent client wallet.
   * client can *only* transfer its own asset between wallet and trading accounts. 

<a name="clientwalletquery"/>

#### Query client and wallets

* Request

```
/phemex-user/users/children?offset=<offset>&limit=<limit>&withCount=<withCount>
```

* Response

```
{
    "code": 0,
        "msg": "OK",
        "data": {
            "total": 87,
            "rows": [
            {
                "userId": 6XXX12,
                "email": "x**@**.com",
                "nickName": "nickName",
                "passwordState": 1,
                "clientCnt": 0,
                "totp": 1,
                "logon": 0,
                "parentId": 0,
                "parentEmail": null,
                "status": 1,
                "wallet": {
                    "totalBalance": "989.25471319",
                    "totalBalanceEv": 98925471319,
                    "availBalance": "989.05471319",
                    "availBalanceEv": 98905471319,
                    "freezeBalance": "0.20000000",
                    "freezeBalanceEv": 20000000,
                    "currency": "BTC",
                    "currencyCode": 1
                },
                "userMarginVo": [
                {
                    "currency": "BTC",
                    "accountBalance": "3.90032508",
                    "totalUsedBalance": "0.00015666",
                    "accountBalanceEv": 390032508,
                    "totalUsedBalanceEv": 15666,
                    "bonusBalanceEv": 0,
                    "bonusBalance": "0"
                },
                {
                    "currency": "USD",
                    "accountBalance": "38050.35000000",
                    "totalUsedBalance": "0.00000000",
                    "accountBalanceEv": 380503500,
                    "totalUsedBalanceEv": 0,
                    "bonusBalanceEv": 0,
                    "bonusBalance": "0"
                }
                ]
            },
            ...
            ]
        }
}
```

Wallet fields

| Field | Type | Description | 
|-------|------|-------------|
| currency | String | currency name |
| totalBalanceEv | Integer | scaled balance amount value |
| availBalanceEv | Integer | scaled available balance value |
| freezeBalanceEv | Integer | scaled used balance value |

Margin fields

| Field | Type | Description |
|-------|------|-------------|
| currency | String | currency name |
| accountBalanceEv | Integer | scaled trading account balance value |
| totalUsedBalanceEv | Integer | Scaled used trading account balance value |
| bonusBalanceEv | Integer | Scaled bonus value |

<a name="walletransferout"/>

#### Main/parent-client transfer self wallet balance to sub-client wallet. (Or Subclient transfer self wallet balance to main/parent client wallet )

* Request
   * Main/parent can transfer its wallet balance to its own subclients.
   * Sub-client can only transfer its wallet balance to its parent/main client.
   * When sub-client transfer its wallet balance, `clientCnt = 0`

```
POST: /exchange/wallets/transferOut

Body:
{
    "amount": 0, // unscaled amount
    "amountEv": 0, // scaled amount, when both amount and amountEv are provided, amountEv wins.
    "clientCnt": 0, // client number, this is from API in children list; when sub-client issues this API, client must be 0.
    "currency": "string"
}

```

| Field | Type |  Required | Description |
|-------|-----|---------|---------------|
| amount | Integer | - | unscaled amount value to transfer |
| amountEv | Integer | - | scaled amount value to transfer |
| clientCnt | Integer | Yes | which client to transfer |
| currency | String | Yes | currency name, currently only support BTC |

* Response
   * This API is synchrous, `code == 0` means succeeded. If timed-out, history can be queried.

```json
{"code":0,"msg":"OK","data":"OK"}
```

<a name="walletransferin"/>

#### Transfer from sub-client wallet. Only main/parent client has priviledge.

* Request
```
POST: /exchange/wallets/transferIn

Body:
{
    "amountEv":10000000,
    "currency":"BTC",
    "clientCnt":1
}

```

* Response

```json
{"code":0,"msg":"OK","data":"OK"}
```

<a name="futureDataApiList"/>

### Future Data API List

<a name="futureDataFundingFeesHist"/>

#### Query Funding Fees History

* Http Request

```
GET /api-data/futures/funding-fees?symbol=<symbol>
```

| Field    | Type           | Required | Description               | Possible Values                 |
|----------|----------------|----------|---------------------------|---------------------------------|
| symbol   | String         | True     | the currency to query     | uBTCUSD ...                     |
| offset   | Integer        | False    | page start from 0         | start from 0, default 0         |
| limit    | Integer        | False    | page size                 | default 20, max 200             |

* Response

```
[
  {
    "createTime": 0,
    "currency": "string",
    "execFeeEv": 0,
    "execPriceEp": 0,
    "execQty": 0,
    "execValueEv": 0,
    "feeRateEr": 0,
    "fundingRateEr": 0,
    "side": "string",
    "symbol": "string"
  }
]
```

<a name="futureDataOrdersHist"/>

#### Query Orders History

* Http Request

```
GET /api-data/futures/orders?symbol=<symbol>
```

| Field    | Type           | Required | Description               | Possible Values                 |
|----------|----------------|----------|---------------------------|---------------------------------|
| symbol   | String         | True     | the currency to query     | uBTCUSD ...                     |
| start    | Long           | False    | start time in millisecond | default 2 days ago from the end |
| end      | Long           | False    | end time in millisecond   | default now                     |
| offset   | Integer        | False    | page start from 0         | start from 0, default 0         |
| limit    | Integer        | False    | page size                 | default 20, max 200             |

* Response

```
[
  {
    "actionTimeNs": 0,
    "bizError": 0,
    "clOrdID": "string",
    "closedPnlEv": 0,
    "closedSize": 0,
    "cumQty": 0,
    "cumValueEv": 0,
    "displayQty": 0,
    "leavesQty": 0,
    "leavesValueEv": 0,
    "ordStatus": "string",
    "orderID": "string",
    "orderQty": 0,
    "orderType": "string",
    "priceEp": 0,
    "reduceOnly": true,
    "side": "string",
    "stopDirection": "string",
    "stopLossEp": 0,
    "symbol": "string",
    "takeProfitEp": 0,
    "timeInForce": "string",
    "transactTimeNs": 0
  }
]
```

<a name="futureDataOrdersByIds"/>

#### Query Orders By Ids 

* Http Request

```
GET /api-data/futures/orders/by-order-id?symbol=<symbol>
```

| Field    | Type     | Required | Description            | Possible Values                                                                                                                     |
|----------|----------|----------|------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| symbol   | String   | True     | the currency to query  | uBTCUSD ...                                                                                                                         |
| orderID  | String   | False    | order id               | orderID and clOrdID can not be both empty. If both IDs are given, it will return list of orders which match both orderID or clOrdID |
| clOrdID  | String   | False    | client order id        | refer to orderID                                                                                                                    |

* Response

```
[
  {
    "actionTimeNs": 0,
    "bizError": 0,
    "clOrdID": "string",
    "closedPnlEv": 0,
    "closedSize": 0,
    "cumQty": 0,
    "cumValueEv": 0,
    "displayQty": 0,
    "leavesQty": 0,
    "leavesValueEv": 0,
    "ordStatus": "string",
    "orderID": "string",
    "orderQty": 0,
    "orderType": "string",
    "priceEp": 0,
    "reduceOnly": true,
    "side": "string",
    "stopDirection": "string",
    "stopLossEp": 0,
    "symbol": "string",
    "takeProfitEp": 0,
    "timeInForce": "string",
    "transactTimeNs": 0
  }
]
```

<a name="futureDataTradesHist"/>

#### Query Trades History

* Http Request

```
GET /api-data/futures/trades?symbol=<symbol>
```

| Field    | Type           | Required | Description               | Possible Values                 |
|----------|----------------|----------|---------------------------|---------------------------------|
| symbol   | String         | True     | the currency to query     | uBTCUSD ...                     |
| start    | Long           | False    | start time in millisecond | default 2 days ago from the end |
| end      | Long           | False    | end time in millisecond   | default now                     |
| offset   | Integer        | False    | page start from 0         | start from 0, default 0         |
| limit    | Integer        | False    | page size                 | default 20, max 200             |

* Response

```
[
  {
    "action": "string",
    "clOrdID": "string",
    "closedPnlEv": 0,
    "closedSize": 0,
    "currency": "string",
    "execFeeEv": 0,
    "execID": "string",
    "execPriceEp": 0,
    "execQty": 0,
    "execStatus": "string",
    "execValueEv": 0,
    "feeRateEr": 0,
    "ordType": "string",
    "orderID": "string",
    "orderQty": 0,
    "priceEp": 0,
    "side": "string",
    "symbol": "string",
    "tradeType": "string",
    "transactTimeNs": 0
  }
]
```

<a name="futureDataTradingFeesHist"/>

#### Query Trading Fees History

* Http Request

```
GET /api-data/futures/trading-fees?symbol=<symbol>
```

| Field    | Type           | Required | Description               | Possible Values                 |
|----------|----------------|----------|---------------------------|---------------------------------|
| symbol   | String         | True     | the currency to query     | uBTCUSD ...                     |
| offset   | Integer        | False    | page start from 0         | start from 0, default 0         |
| limit    | Integer        | False    | page size                 | default 20, max 200             |

* Response

```
[
  {
    "createTime": 0,
    "currency": "string",
    "exchangeFeeValueEv": 0,
    "id": 0,
    "makerRateEr": 0,
    "makerValueEv": 0,
    "symbol": "string",
    "takerRateEr": 0,
    "takerValueEv": 0,
    "userId": 0
  }
]
```

<a name="withdraw"/>

### Withdraw
   * Several restrictions are required for withdraw: 1. bind Google 2FA, 2. Password change out of 24hour, 3. meet minimum BTC amount requirement.

<a name="requestwithdraw"/>

#### Request Withdraw

* Request

```
POST /exchange/wallets/createWithdraw?otpCode=<otpCode>
Body: 
{
      "address": <address>,// address must set before withdraw
      "amountEv": <amountEv>, // scaled btc value
      "currency": <currency> // fixed to BTC 
}
```
| Filed | Type | Required |  Description | Possible values |
|------|------|----------|--------------|-----------------|
| otpCode | String | Yes | In URL query, From Google 2FA| |
| address | String | Yes | In body, address must be saved before hand | |
| amountEv | Integer | Yes | In body, scaled amount value | |
| currency | String | Yes | In body, currently only support BTC | |


* Sample code to get Google 2FA code via API

```
api 'com.warrenstrange:googleauth:1.1.2'

import com.warrenstrange.googleauth.GoogleAuthenticator;

@Test
public void testAuth() {
    String secret = "XXXXXXXXXXXXXXXX"; // save from binding Google 2FA
    GoogleAuthenticator gAuth = new GoogleAuthenticator();
    int code = gAuth.getTotpPassword(secret);
    boolean ans = gAuth.authorize(secret, code);
    Assert.assertTrue(ans);
}

```

* Response


```
{
    "code": 0,
    "msg" : "OK",
    "data": <withdrawRequest> 
}

```

Response Fileds

| Filed | Type | Description |
|-------|------|-------------|
| id    | Integer | withdraw id |
| currency | String | currency name |
| status | String | Withdraw request processing state |
| amountEv | Integer | Scaled withdraw amount |
| feeEv | Integer | Scaled withdraw fee amount |
| address | String | Withdraw target address |
| txhash | String | transaction hash on blockchain |
| submitedAt | Integer | submitted time in epoch |
| expiredTime | Integer | expire time in epoch |

<a name="confirmwithdraw"/>

#### Confirm withdraw

   * After withdraw request submitted, a confirmation link is sent to registration email. The confirm code should be extracted out from the link and then passed in as url query parameter.

* Request
```
GET /exchange/wallets/confirm/withdraw?code=<withdrawConfirmCode>
```

* Response

```
{
    "code": 0,
    "msg" : "OK"
}
```

<a name="cancelwithdraw"/>

#### Cancel withdraw
   * Withdraw request can be canceled before mannual `review`;

* Request

```
POST /exchange/wallets/cancelWithdraw
Body:
{
    id: <withdrawRequestId>
}
```

<a name="listwithdraw"/>

#### List withdraws

* Request

```
GET /exchange/wallets/withdrawList?currency=<currency>&limit=<limit>&offset=<offset>&withCount=<withCount>
```

* Response
   * List of withdraw requests

<a name="withdrawaddrmgmt"/>

#### Withdraw address management
   * Withdraw address management support create, remove and list. Recommend manage it from website.

* Request

```
POST /exchange/wallets/createWithdrawAddress?otpCode={optCode}
Body: 
{
    "address": <address>,
    "currency": <currency>
    "remark": <name>
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| address | String | Yes | valid BTC address |
| currency | String | Yes | Currrently only support `BTC` |
| remark | String | Yes | Name of this address |

* Response
```
{
    "code": 0,
    "msg": "OK",
    "data": 1 //subject to change
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
* Each connection has throttle limit to *20* request/s.

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
```
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
| expiry      | Integer| A future time after which request will be rejected, in epoch ***second***. Maximum expiry is request time plus 2 minutes ||

* Sample:

```
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
```
> {
  "id": 1234,
  "method": "orderbook.subscribe",
  "params": [
    "BTCUSD"
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

DataGW publishes order book message with types: incremental, snapshot. Incremental messages are published with 20ms interval. And snapshot messages are published with 60-second interval for client self-verification.

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
| depth       | Integer| Market depth     | 30              |
| type        | String | Message type     | snapshot, incremental |
  

* Sample：
 
```
< {"book":{"asks":[[86765000,19609],[86770000,7402],[86775000,3807],[86780000,7395],[86785000,3599],[86790000,7253],[86795000,4019],[86800000,4366],[86805000,3216],[86810000,3107],[86815000,7453],[86820000,1771],[86825000,895],[86830000,3420],[86835000,1818],[86840000,1272],[86845000,1064],[86850000,195],[86855000,1630],[86860000,1017],[86865000,3509],[86870000,1105],[86875000,1262],[86880000,893],[86885000,862],[86890000,1030],[86895000,2315],[86900000,2994],[86905000,2026],[86910000,3387],[86915000,1382],[86920000,1202],[86925000,3150],[86930000,1773],[86935000,1778],[86940000,1384],[86945000,1842],[86950000,1019],[86955000,2660],[86960000,1599],[86965000,920],[86970000,1834],[86975000,752],[86980000,1384],[86985000,2471],[86990000,2133],[86995000,2981],[87000000,1091],[87005000,994],[87010000,1217],[87015000,1098],[87020000,526],[87025000,1779],[87030000,1098],[87035000,892],[87040000,2168],[87045000,822],[87050000,2410],[87055000,630],[87060000,1684],[87065000,2556],[87070000,19],[87080000,1445],[87085000,29],[87105000,2002],[87115000,658],[87120000,660],[87905000,991]],"bids":[[86760000,18995],[86755000,6451],[86750000,5311],[86745000,6867],[86740000,6180],[86735000,3127],[86730000,4852],[86725000,6213],[86720000,3902],[86715000,4510],[86710000,10063],[86705000,1118],[86700000,1891],[86695000,767],[86690000,20920],[86685000,2535],[86680000,1105],[86675000,645],[86670000,1424],[86665000,1773],[86660000,1464],[86655000,1160],[86650000,1462],[86645000,2446],[86640000,538],[86635000,506],[86630000,2291],[86625000,2981],[86620000,1712],[86615000,984],[86610000,1058],[86605000,1261],[86600000,1074],[86595000,1408],[86590000,717],[86585000,1582],[86580000,1950],[86575000,1540],[86570000,2960],[86565000,598],[86560000,759],[86555000,1266],[86550000,1943],[86545000,259],[86540000,2106],[86535000,2365],[86530000,857],[86525000,1200],[86520000,2371],[86515000,2103],[86510000,1468],[86505000,747],[86500000,1369],[86495000,2121],[86490000,3674],[86485000,1345],[86480000,1290],[86475000,1716],[86470000,1851],[86465000,1861],[86460000,1092],[86435000,21],[86430000,986],[86420000,1202],[86415000,22],[86405000,1199],[86390000,470],[86365000,920],[86360000,192],[86355000,474],[86350000,1838],[86335000,1104],[86285000,2205],[86280000,2390],[86275000,95],[86255000,2836],[86250000,589],[86240000,424],[86235000,937],[86225000,374],[86220000,1591],[86215000,517],[86210000,559],[86205000,702],[86190000,54]]},"depth":30,"sequence":1191904,"symbol":"BTCUSD","type":"snapshot"}
< {"book":{"asks":[[86775000,4621]],"bids":[]},"depth":30,"sequence":1191905,"symbol":"BTCUSD","type":"incremental"}
< {"book":{"asks":[],"bids":[[86755000,8097]]},"depth":30,"sequence":1191906,"symbol":"BTCUSD","type":"incremental"}
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

On each successful subscription, DataGW will send the 200 history trades immediately for the subscribed symbol and all the later trades will be published.

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

```
> {
  "id": 1234,
  "method": "trade.subscribe",
  "params": [
    "BTCUSD"
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

DataGW publishes trade message with types: incremental, snapshot. Incremental messages are published with 20ms interval. And snapshot messages are published on connection initial setup for client recovery.


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
| symbol      | String | Contract symbol name     ||
| type        | String | Message type     |snapshot, incremental |
  

* Sample
```
< {
  "sequence": 1167852,
  "symbol": "BTCUSD",
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
  "symbol": "BTCUSD",
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

### Unsubscribe Trade

It unsubscribes all trade subscriptions or for a symbol.

* Request

```
# unsubscribe all trade subsciptions
{
  "id": <id>,
  "method": "trade.unsubscribe",
  "params": [
  ]
}

# unsubscribe all trade subsciptions for a symbol
{
  "id": <id>,
  "method": "trade.unsubscribe",
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

```
# subscribe 1-day kline
> {
  "id": 1234,
  "method": "kline.subscribe",
  "params": [
    "BTCUSD",
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

DataGW publishes kline message with types: incremental, snapshot. Incremental messages are published with 20ms interval. And snapshot messages are published on connection initial setup for client recovery.

```
{
  "kline": [
    [
      <timestamp>,
      "<interval>",
      <lastCloseEp>,
      <openEp>,
      <highEp>,
      <lowEp>,
      <closeEp>,
      <volume>,
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
| volume      | Integer| Trade voulme during the current kline interval ||
| turnoverEv  | Integer| Scaled turnover value    |                 |
| sequence    | Integer| Latest message sequence  ||
| symbol      | String | Contract symbol name     ||
| type        | String | Message type     |snapshot, incremental |
  

* Sample
```
< {
  "kline": [
    [
      1590019200,
      86400,
      95165000,
      95160000,
      95160000,
      95160000,
      95160000,
      164,
      1723413
    ],
    [
      1589932800,
      86400,
      97840000,
      97840000,
      98480000,
      92990000,
      95165000,
      246294692,
      2562249857942
    ],
    [
      1589846400,
      86400,
      97335000,
      97335000,
      99090000,
      94490000,
      97840000,
      212484260,
      2194232158593
    ]
  ],
  "sequence": 1118993873,
  "symbol": "BTCUSD",
  "type": "snapshot"
}

< {
  "kline": [
    [
      1590019200,
      86400,
      95165000,
      95160000,
      95750000,
      92585000,
      93655000,
      84414679,
      892414738605
    ]
  ],
  "sequence": 1122006398,
  "symbol": "BTCUSD",
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


<a name="aopsub"/>

### Subscribe Account-Order-Position (AOP)

AOP subscription requires the session been authorized successfully. DataGW extracts the user information from the given token and sends AOP messages back to client accordingly. 0 or more latest account snapshot messages will be sent to client immediately on subscription, and incremental messages will be sent for later updates. Each account snapshot contains a trading account information, holding positions, and open / max 100 closed / max 100 filled order event message history.

* Request

```
{
  "id": <id>,
  "method": "aop.subscribe",
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
```
> {
  "id": 1234,
  "method": "aop.subscribe",
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




#### Account-Order-Position (AOP) Message Sample:

```json
{"accounts":[{"accountBalanceEv":9992165009,"accountID":604630001,"currency":"BTC","totalUsedBalanceEv":10841771568,"userID":60463}],"orders":[{"accountID":604630001,...}],"positions":[{"accountID":604630001,...}],"sequence":11450, "timestamp":<timestamp>, "type":"<type>"}

```

| Field       | Type   | Description      | Possible values |
|-------------|--------|------------------|-----------------|
| timestamp   | Integer| Transaction timestamp in nanoseconds | |
| sequence    | Integer| Latest message sequence |          |
| symbol      | String | Contract symbol name    |          |
| type        | String | Message type     | snapshot, incremental |



* Sample:

```
< {"accounts":[{"accountBalanceEv":100000024,"accountID":675340001,"bonusBalanceEv":0,"currency":"BTC","totalUsedBalanceEv":1222,"userID":67534}],"orders":[{"accountID":675340001,"action":"New","actionBy":"ByUser","actionTimeNs":1573711481897337000,"addedSeq":1110523,"bonusChangedAmountEv":0,"clOrdID":"uuid-1573711480091","closedPnlEv":0,"closedSize":0,"code":0,"cumQty":2,"cumValueEv":23018,"curAccBalanceEv":100000005,"curAssignedPosBalanceEv":0,"curBonusBalanceEv":0,"curLeverageEr":0,"curPosSide":"Buy","curPosSize":2,"curPosTerm":1,"curPosValueEv":23018,"curRiskLimitEv":10000000000,"currency":"BTC","cxlRejReason":0,"displayQty":2,"execFeeEv":-5,"execID":"92301512-7a79-5138-b582-ac185223727d","execPriceEp":86885000,"execQty":2,"execSeq":1131034,"execStatus":"MakerFill","execValueEv":23018,"feeRateEr":-25000,"lastLiquidityInd":"AddedLiquidity","leavesQty":0,"leavesValueEv":0,"message":"No error","ordStatus":"Filled","ordType":"Limit","orderID":"e9a45803-0af8-41b7-9c63-9b7c417715d9","orderQty":2,"pegOffsetValueEp":0,"priceEp":86885000,"relatedPosTerm":1,"relatedReqNum":2,"side":"Buy","stopLossEp":0,"stopPxEp":0,"symbol":"BTCUSD","takeProfitEp":0,"timeInForce":"GoodTillCancel","tradeType":"Trade","transactTimeNs":1573712555309040417,"userID":67534},{"accountID":675340001,"action":"New","actionBy":"ByUser","actionTimeNs":1573711490507067000,"addedSeq":1110980,"bonusChangedAmountEv":0,"clOrdID":"uuid-1573711488668","closedPnlEv":0,"closedSize":0,"code":0,"cumQty":3,"cumValueEv":34530,"curAccBalanceEv":100000013,"curAssignedPosBalanceEv":0,"curBonusBalanceEv":0,"curLeverageEr":0,"curPosSide":"Buy","curPosSize":5,"curPosTerm":1,"curPosValueEv":57548,"curRiskLimitEv":10000000000,"currency":"BTC","cxlRejReason":0,"displayQty":3,"execFeeEv":-8,"execID":"80899855-5b95-55aa-b84e-8d1052f19886","execPriceEp":86880000,"execQty":3,"execSeq":1131408,"execStatus":"MakerFill","execValueEv":34530,"feeRateEr":-25000,"lastLiquidityInd":"AddedLiquidity","leavesQty":0,"leavesValueEv":0,"message":"No error","ordStatus":"Filled","ordType":"Limit","orderID":"7e03cd6b-e45e-48d9-8937-8c6628e7a79d","orderQty":3,"pegOffsetValueEp":0,"priceEp":86880000,"relatedPosTerm":1,"relatedReqNum":3,"side":"Buy","stopLossEp":0,"stopPxEp":0,"symbol":"BTCUSD","takeProfitEp":0,"timeInForce":"GoodTillCancel","tradeType":"Trade","transactTimeNs":1573712559100655668,"userID":67534},{"accountID":675340001,"action":"New","actionBy":"ByUser","actionTimeNs":1573711499282604000,"addedSeq":1111025,"bonusChangedAmountEv":0,"clOrdID":"uuid-1573711497265","closedPnlEv":0,"closedSize":0,"code":0,"cumQty":4,"cumValueEv":46048,"curAccBalanceEv":100000024,"curAssignedPosBalanceEv":0,"curBonusBalanceEv":0,"curLeverageEr":0,"curPosSide":"Buy","curPosSize":9,"curPosTerm":1,"curPosValueEv":103596,"curRiskLimitEv":10000000000,"currency":"BTC","cxlRejReason":0,"displayQty":4,"execFeeEv":-11,"execID":"0be06645-90b8-5abe-8eb0-dca8e852f82f","execPriceEp":86865000,"execQty":4,"execSeq":1132422,"execStatus":"MakerFill","execValueEv":46048,"feeRateEr":-25000,"lastLiquidityInd":"AddedLiquidity","leavesQty":0,"leavesValueEv":0,"message":"No error","ordStatus":"Filled","ordType":"Limit","orderID":"66753807-9204-443d-acf9-946d15d5bedb","orderQty":4,"pegOffsetValueEp":0,"priceEp":86865000,"relatedPosTerm":1,"relatedReqNum":4,"side":"Buy","stopLossEp":0,"stopPxEp":0,"symbol":"BTCUSD","takeProfitEp":0,"timeInForce":"GoodTillCancel","tradeType":"Trade","transactTimeNs":1573712618104628671,"userID":67534}],"positions":[{"accountID":675340001,"assignedPosBalanceEv":0,"avgEntryPriceEp":86875941,"bankruptCommEv":75022,"bankruptPriceEp":90000,"buyLeavesQty":0,"buyLeavesValueEv":0,"buyValueToCostEr":1150750,"createdAtNs":0,"crossSharedBalanceEv":99998802,"cumClosedPnlEv":0,"cumFundingFeeEv":0,"cumTransactFeeEv":-24,"currency":"BTC","dataVer":4,"deleveragePercentileEr":0,"displayLeverageEr":1000000,"estimatedOrdLossEv":0,"execSeq":1132422,"freeCostEv":0,"freeQty":-9,"initMarginReqEr":1000000,"lastFundingTime":1573703858883133252,"lastTermEndTime":0,"leverageEr":0,"liquidationPriceEp":90000,"maintMarginReqEr":500000,"makerFeeRateEr":0,"markPriceEp":86786292,"orderCostEv":0,"posCostEv":1115,"positionMarginEv":99925002,"positionStatus":"Normal","riskLimitEv":10000000000,"sellLeavesQty":0,"sellLeavesValueEv":0,"sellValueToCostEr":1149250,"side":"Buy","size":9,"symbol":"BTCUSD","takerFeeRateEr":0,"term":1,"transactTimeNs":1573712618104628671,"unrealisedPnlEv":-107,"updatedAtNs":0,"usedBalanceEv":1222,"userID":67534,"valueEv":103596}],"sequence":1310812,"timestamp":1573716998131003833,"type":"snapshot"}
< {"accounts":[{"accountBalanceEv":99999989,"accountID":675340001,"bonusBalanceEv":0,"currency":"BTC","totalUsedBalanceEv":1803,"userID":67534}],"orders":[{"accountID":675340001,"action":"New","actionBy":"ByUser","actionTimeNs":1573717286765750000,"addedSeq":1192303,"bonusChangedAmountEv":0,"clOrdID":"uuid-1573717284329","closedPnlEv":0,"closedSize":0,"code":0,"cumQty":0,"cumValueEv":0,"curAccBalanceEv":100000024,"curAssignedPosBalanceEv":0,"curBonusBalanceEv":0,"curLeverageEr":0,"curPosSide":"Buy","curPosSize":9,"curPosTerm":1,"curPosValueEv":103596,"curRiskLimitEv":10000000000,"currency":"BTC","cxlRejReason":0,"displayQty":4,"execFeeEv":0,"execID":"00000000-0000-0000-0000-000000000000","execPriceEp":0,"execQty":0,"execSeq":1192303,"execStatus":"New","execValueEv":0,"feeRateEr":0,"leavesQty":4,"leavesValueEv":46098,"message":"No error","ordStatus":"New","ordType":"Limit","orderID":"e329ae87-ce80-439d-b0cf-ad65272ed44c","orderQty":4,"pegOffsetValueEp":0,"priceEp":86770000,"relatedPosTerm":1,"relatedReqNum":5,"side":"Buy","stopLossEp":0,"stopPxEp":0,"symbol":"BTCUSD","takeProfitEp":0,"timeInForce":"GoodTillCancel","transactTimeNs":1573717286765896560,"userID":67534},{"accountID":675340001,"action":"New","actionBy":"ByUser","actionTimeNs":1573717286765750000,"addedSeq":1192303,"bonusChangedAmountEv":0,"clOrdID":"uuid-1573717284329","closedPnlEv":0,"closedSize":0,"code":0,"cumQty":4,"cumValueEv":46098,"curAccBalanceEv":99999989,"curAssignedPosBalanceEv":0,"curBonusBalanceEv":0,"curLeverageEr":0,"curPosSide":"Buy","curPosSize":13,"curPosTerm":1,"curPosValueEv":149694,"curRiskLimitEv":10000000000,"currency":"BTC","cxlRejReason":0,"displayQty":4,"execFeeEv":35,"execID":"8d1848a2-5faf-52dd-be71-9fecbc8926be","execPriceEp":86770000,"execQty":4,"execSeq":1192303,"execStatus":"TakerFill","execValueEv":46098,"feeRateEr":75000,"lastLiquidityInd":"RemovedLiquidity","leavesQty":0,"leavesValueEv":0,"message":"No error","ordStatus":"Filled","ordType":"Limit","orderID":"e329ae87-ce80-439d-b0cf-ad65272ed44c","orderQty":4,"pegOffsetValueEp":0,"priceEp":86770000,"relatedPosTerm":1,"relatedReqNum":5,"side":"Buy","stopLossEp":0,"stopPxEp":0,"symbol":"BTCUSD","takeProfitEp":0,"timeInForce":"GoodTillCancel","tradeType":"Trade","transactTimeNs":1573717286765896560,"userID":67534}],"positions":[{"accountID":675340001,"assignedPosBalanceEv":0,"avgEntryPriceEp":86843828,"bankruptCommEv":75056,"bankruptPriceEp":130000,"buyLeavesQty":0,"buyLeavesValueEv":0,"buyValueToCostEr":1150750,"createdAtNs":0,"crossSharedBalanceEv":99998186,"cumClosedPnlEv":0,"cumFundingFeeEv":0,"cumTransactFeeEv":11,"currency":"BTC","dataVer":5,"deleveragePercentileEr":0,"displayLeverageEr":1000000,"estimatedOrdLossEv":0,"execSeq":1192303,"freeCostEv":0,"freeQty":-13,"initMarginReqEr":1000000,"lastFundingTime":1573703858883133252,"lastTermEndTime":0,"leverageEr":0,"liquidationPriceEp":130000,"maintMarginReqEr":500000,"makerFeeRateEr":0,"markPriceEp":86732335,"orderCostEv":0,"posCostEv":1611,"positionMarginEv":99924933,"positionStatus":"Normal","riskLimitEv":10000000000,"sellLeavesQty":0,"sellLeavesValueEv":0,"sellValueToCostEr":1149250,"side":"Buy","size":13,"symbol":"BTCUSD","takerFeeRateEr":0,"term":1,"transactTimeNs":1573717286765896560,"unrealisedPnlEv":-192,"updatedAtNs":0,"usedBalanceEv":1803,"userID":67534,"valueEv":149694}],"sequence":1315725,"timestamp":1573717286767188294,"type":"incremental"}

```


<a name="aopunsub"/>

### Unsubscribe Account-Order-Position (AOP)
* Request：

```
{
  "id": <id>,
  "method": "aop.unsubscribe",
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
  "method": "market24h.subscribe",
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

```
> {
  "method": "market24h.subscribe",
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
  "market24h": {
    "open": <open priceEp>,
    "high": <high priceEp>,
    "low": <low priceEp>,
    "close": <close priceEp>,
    "indexPrice": <index priceEp>,
    "markPrice": <mark priceEp>,
    "openInterest": <open interest>,
    "fundingRate": <funding rateEr>,
    "predFundingRate": <predicated funding rateEr>,
    "symbol": "<symbol>",
    "turnover": <turnoverEv>,
    "volume": <volume>
  },
  "timestamp": <timestamp>
}
```

| Field         | Type   | Description                                | Possible values |
|---------------|--------|--------------------------------------------|--------------|
| open priceEp  | Integer| The scaled open price in last 24 hours     |              |
| high priceEp  | Integer| The scaled highest price in last 24 hours  |              |
| low priceEp   | Integer| The scaled lowest price in last 24 hours   |              |
| close priceEp | Integer| The scaled close price in last 24 hours    |              |
| index priceEp | Integer| Scaled index price                         |              |
| mark priceEp  | Integer| Scaled mark price                          |              |
| open interest | Integer| current open interest                      |              |
| funding rateEr| Integer| Scaled funding rate                        |              |
| predicated funding rateEr| Integer| Scaled predicated funding rate  |              |
| timestamp     | Integer| Timestamp in nanoseconds                   |              |
| symbol        | String | Contract symbol name                       | [Trading symbols](#symbpricesub) |
| turnoverEv    | Integer| The scaled turnover value in last 24 hours |              |
| volume        | Integer| Symbol trade volume in last 24 hours       |              |
  
* Sample:

```
< {
  "market24h": {
    "close": 87425000,
    "fundingRate": 10000,
    "high": 92080000,
    "indexPrice": 87450676,
    "low": 87130000,
    "markPrice": 87453092,
    "open": 90710000,
    "openInterest": 7821141,
    "predFundingRate": 7609,
    "symbol": "BTCUSD",
    "timestamp": 1583646442444219017,
    "turnover": 1399362834123,
    "volume": 125287131
  },
  "timestamp": 1576490244024818000
}
```

<a name="symbpricesub"/>

### Subscribe tick event for symbol price

   * Every trading symbol has a suite of other symbols, each symbol follows same patterns,
   i.e. `index` symbol follows a pattern `.<BASECURRENCY>`,
        `mark` symbol follows a pattern `.M<BASECURRENCY>`,
        predicated funding rate's symbol follows a pattern `.<BASECURRENCY>FR`,
        while funding rate symbol follows a pattern `.<BASECURRENCY>FR8H`
   * Price is retrieved by subscribing symbol tick.
   * all available symbols

   | symbol | index symbol |  mark symbol | Predicated Funding rate symbol | Funding rate symbol |
   |--------|--------------|--------------|--------------------------------|---------------------|
   | BTCUSD | .BTC         | .MBTC        | .BTCFR                         | .BTCFR8H |
   | XRPUSD | .XRP         | .MXRP        | .XRPFR                         | .XRPFR8H |
   | ETHUSD | .ETH         | .METH        | .ETHFR                         | .ETHFR8H |
   | LINKUSD| .LINK        | .MLINK       | .LINKFR                        | .LINKFR8H|
   | XTZUSD | .XTZ         | .MXTZ        | .XTZFR                         | .XTZFR8H |
   | LTCUSD | .LTC         | .MLTC        | .LTCFR                         | .LTCFR8H |
   | GOLDUSD| .GOLD        | .MGOLD       | .GOLDFR                        | .GOLDFR8H|
   | ADAUSD | .ADA         | .MADA        | .ADAFR                         | .ADAFR8H |
   | BCHUSD | .BCH         | .MBCH        | .BCHFR                         | .BCHFR8H |
   | COMPUSD| .COMP        | .MCOMP       | .COMPFR                        | .COMPFR8H|
   | ALGOUSD| .ALGO        | .MALGO       | .ALGOFR                        | .ALGOFR8H|
   | YFIUSD | .YFI         | .MYFI        | .YFIFR                         | .YFIFR8H |
   | DOTUSD | .DOT         | .MDOT        | .DOTFR                         | .DOTFR8H |
   | UNIUSD | .UNI         | .MUNI        | .UNIFR                         | .UNIFR8H |
   | BATUSD | .BAT         | .MBAT        | .BATFR                         | .BATFR8H |
   | CHZUSD | .CHZ         | .MCHZ        | .CHZFR                         | .CHZFR8H |
   | MANAUSD| .MANA        | .MMANA       | .MANAFR                        | .MANAFR8H|
   | ENJUSD | .ENJ         | .MENJ        | .ENJFR                         | .ENJFR8H |
   |SUSHIUSD| .SUSHI       | .MSUSHI      | .SUSHIFR                       |.SUSHIFR8H|
   | SNXUSD | .SNX         | .MSNX        | .SNXFR                         | .SNXFR8H |
   | GRTUSD | .GRT         | .MGRT        | .GRTFR                         | .GRTFR8H |
   | MKRUSD | .MKR         | .MMKR        | .MKRFR                         | .MKRFR8H |
   | TRXUSD | .TRX         | .MTRX        | .TRXFR                         | .TRXFR8H |
   | EOSUSD | .EOS         | .MEOS        | .EOSFR                         | .EOSFR8H |
   | ONTUSD | .ONT         | .MONT        | .ONTFR                         | .ONTFR8H |
   | NEOUSD | .NEO         | .MNEO        | .NEOFR                         | .NEOFR8H |
   | ZECUSD | .ZEC         | .MZEC        | .ZECFR                         | .ZECFR8H |
   | FILUSD | .FIL         | .MFIL        | .FILFR                         | .FILFR8H |
   | KSMUSD | .KSM         | .MKSM        | .KSMFR                         | .KSMFR8H |
   | XMRUSD | .XMR         | .MXMR        | .XMRFR                         | .XMRFR8H |
   | QTUMUSD| .QTUM        | .MQTUM       | .QTUMFR                        |.QTUMFR8H |
   | XLMUSD | .XLM         | .MXLM        | .XLMFR                         | .XLMFR8H |
   | ATOMUSD| .ATOM        | .MATOM       | .ATOMFR                        | .ATOMFR8H|
   | LUNAUSD| .LUNA        | .MLUNA       | .LUNAFR                        | .LUNAFR8H|
   | SOLUSD | .SOL         | .MSOL        | .SOLFR                         | .SOLFR8H |


* Request

   * The symbol in params can be replace by any symbol. 

```
{
    "method": "tick.subscribe",
        "params": [ <symbol> ],
        "id": <id>
}
```

* Response

ack message

```
{
    "error": null,
        "id": <id>,
        "result": {
            "status": "success"
        }
}
```

push event

```
{
    "tick": {
        "last": <priceEp>,
            "scale": <scale>,
            "symbol": <symbol>
            "timestmp": <timestamp_nano>
    }
}
```


* Sample

```
> {"method":"tick.subscribe","params":[".BTC"],"id":1580631267153}
< {"error":null,"id":1580631267153,"result":{"status":"success"}}
< {"tick":{"last":93385362,"scale":4,"symbol":".BTC","timestmp":1580635719408000000}}
< {"tick":{"last":93390304,"scale":4,"symbol":".BTC","timestmp":1580635719821000000}}
< {"tick":{"last":93403484,"scale":4,"symbol":".BTC","timestmp":1580635721424000000}}
```

