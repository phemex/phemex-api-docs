## Table of Contents
* [Phemex Public API](#publicapi)
  * [General Public API Information](#general)
* [REST API Standards](#restapi)
  * [HTTP Return Codes](#httpreturncodes)
  * [HTTP REST Request Header](#httprestheader)
  * [API Rate Limits](#apiratelimits)
  * [Endpoint security type](#securitytype)
    * [Signature Example 1: HTTP GET Request](#signatureexample1)
    * [Singature Example 2: HTTP GET Request with multiple query string](#signatureexample2)
    * [Signature Example 3: HTTP POST Request](#signatureexample3)
  * [Price/Ratio/Value Scales](#scalingfactors)
  * [REST API List](#restapilist)
    * [Market API List](#marketapilist)
      * [Query Product Information](#queryproductinfo)
    * [Trade API List](#orderapilist)
      * [Place Order](#placeorder)
      * [Cancel Single order by orderID](#cancelsingleorder)
      * [Cancel Order](#cancelorder)
      * [Cancel All Orders](#cancelall)
      * [Query trading account and positions](#querytradeaccount)
      * [Change position leverage](#changeleverage)
      * [Change position risklimt](#changerisklimit)
      * [Assign position balance in isolated margin mode](#assignposbalance)
      * [Query open orders by symbol](#queryopenorder)
      * [Query closed orders by symbol](#queryorder)
      * [Query order by orderID](#queryorderbyid)
      * [Query user trades by symbol](#querytrade)
    * [Market Data API List](#mdapilist)
      * [Query Order Book](#queryorderbook)
      * [Query Recent Trades](#querytrades)
      * [Query 24 Hours Ticker](#query24hrsticker)
    * [Asset Api List](#assetapilist)
      * [Query client and wallets](#clientwalletquery)
      * [Transfer self balance to parent or subclients](#walletransferout)
      * [Transfer from sub-client wallet](#walletransferin)
      * [Transfer between wallet and trading account](#transferwallettradingaccount)
      * [Query wallet/tradingaccount transfer history](#transferwallettradingaccountquery)
    * [Withdraw](#withdraw)
      * [Request withdraw](#requestwithdraw)
      * [Confirm withdraw](#confirmwithdraw)
      * [Cancel withdraw](#cancelwithdraw)
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
    * [Subscribe Account-Order-Position (AOP)](#aopsub)
    * [Unsubscribe Account-Order-Position (AOP)](#aopunsub)
    * [Subscribe 24 Hours Ticker](#tickersub)
    * [Unsubscribe 24 Hours Ticker](#tickerunsub)

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

<a name="httpreturncodes"/>

## 1. HTTP Return Codes

* HTTP `403` return code is used when IP break Rate Limit.
* HTTP `429` return code is used when breaking a request rate limit.
* HTTP `5XX` return codes are used for Phemex internal errors. Note: This doesn't means the operation failed, the execution status is **UNKNOWN** and could be Succeed.


<a name="httprestheader"/>

## 2. HTTP REST Request Header 

Every HTTP Rest Request must have the following Headers:
* x-phemex-access-token : This is  API-KEY (id field) from Phemex site.
* x-phemex-request-expiry : This describes the Unix EPoch SECONDS to expire the request, normally it should be (Now() + 1 minute)
* x-phemex-request-signature : This is HMAC SHA256 signature of the http request, its formula is : HMacSha256( URL Path + QueryString + Expiry + body )


<a name="apiratelimits"/>

## 3. API Rate Limits

* Every Client has the API call rate limit as 200 per minute.
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

* API REST Request URL: https://api.phemex.com/accounts/accountPositions?currency=BTC
   * Request Path: /accounts/accountPositions
   * Request Query: currency=BTC
   * Request Body: <null>
   * Request Expiry: 1575735514
   * Signature: HMacSha256( /accounts/accountPositions + currency=BTC + 1575735514 )

<a name="signatureexample2"/>

### 4.2 Singature Example 2: HTTP GET Request with multiple query string

* API REST Request URL: https://api.phemex.com/orders/activeList?ordStatus=New&ordStatus=PartiallyFilled&ordStatus=Untriggered&symbol=BTCUSD 
    * Request Path: /orders/activeList
    * Request Query: ordStatus=New&ordStatus=PartiallyFilled&ordStatus=Untriggered&symbol=BTCUSD
    * Request Body: <null>
    * Request Expire: 1575735951603
    * Signature: HMacSha256(/orders/activeList + ordStatus=New&ordStatus=PartiallyFilled&ordStatus=Untriggered&symbol=BTCUSD + 1575735951603)
    * signed string is `/orders/activeListordStatus=New&ordStatus=PartiallyFilled&ordStatus=Untriggered&symbol=BTCUSD1575735951603`

<a name="signatureexample3"/>

### 4.3 Signature Example 3: HTTP POST Request

* API REST Request URL: https://api.phemex.com/orders
   * Request Path: /orders
   * Request Query: <null>
   * Request Body: {"symbol":"BTCUSD","clOrdID":"uuid-1573058952273","side":"Sell","priceEp":93185000,"orderQty":7,"ordType":"Limit","reduceOnly":false,"timeInForce":"GoodTillCancel","takeProfitEp":0,"stopLossEp":0}
   * Request Expiry: 1575735514
   * Signature: HMacSha256( /orders + 1575735514 + {"symbol":"BTCUSD","clOrdID":"uuid-1573058952273","side":"Sell","priceEp":93185000,"orderQty":7,"ordType":"Limit","reduceOnly":false,"timeInForce":"GoodTillCancel","takeProfitEp":0,"stopLossEp":0})
   * signed string is `/orders1575735514{"symbol":"BTCUSD","clOrdID":"uuid-1573058952273","side":"Sell","priceEp":93185000,"orderQty":7,"ordType":"Limit","reduceOnly":false,"timeInForce":"GoodTillCancel","takeProfitEp":0,"stopLossEp":0}`

<a name="scalingfactors"/>

## 5. Price/Ratio/Value Scales

Fields with post-fix "Ep", "Er" or "Ev" have been scaled based on symbol setting.
* Fields with post-fix "Ep" are scaled prices
* Fields with post-fix "Er" are scaled ratios
* Fields with post-fix "Ev" are scaled values

| Symbol | Price scale | Ratio scale | Value scale |
|--------|-------------|-------------|-------------|
| BTCUSD | 10,000      | 100,000,000 | 100,000,000 |
| ETHUSD | 10,000      | 100,000,000 |      10,000 |
| XRPUSD | 10,000      | 100,000,000 |      10,000 |

<a name="restapilist"/>

## 6. REST API List

<a name="marketapilist"/>

### 6.1 Market API List

<a name="queryproductinfo"/>

#### 6.1.1 Query Product Information

* Request：
```json
GET /exchange/public/products
```

* Response:
```
{
  "code": 0,
  "msg": "OK",
  "data": [
    {
      "symbol": "<symbol>",
      "underlyingSymbol": "<underlyingSymbol>",
      "quoteCurrency": "<quoteCurrency>",
      "settlementCurrency": "<settlementCurrency>",
      "maxOrderQty": <maxOrderQty>,
      "maxPriceEp": <maxPriceEp>,
      "lotSize": <lotSize>,
      "tickSize": "<tickSize>",
      "contractSize": "<contractSize>",
      "priceScale": <priceScale>,
      "ratioScale": <ratioScale>,
      "valueScale": <valueScale>,
      "defaultLeverage": <defaultLeverage>,
      "maxLeverage": <maxLeverage>,
      "initMarginEr": "<initMarginEr>",
      "maintMarginEr": "<maintMarginEr>",
      "defaultRiskLimitEv": <defaultRiskLimitEv>,
      "deleverage": <deleverage>,
      "makerFeeRateEr": <makerFeeRateEr>,
      "takerFeeRateEr": <takerFeeRateEr>,
      "fundingInterval": <fundingInterval>,
      "description": "<description>"
    },
    ...
    ...
    ...
  ]
}
```

| Field                | Type   | Description                                | Possible values |
|----------------------|--------|--------------------------------------------|--------------|
| symbol               | String | Contract Symbol name                       | BTCUSD, ETHUSD, XRPUSD |
| underlyingSymbol     | String | The underlying index symbol                | .BTC, .ETH, .XRP       |
| quoteCurrency        | String | The currency for price quoting             | USD                    |
| settlementCurrency   | String | The settlement currency                    | BTC, USD               |
| maxOrderQty          | Integer| The maximum order size for the contract    |                        |
| maxPriceEp           | Integer| The maximum allowed order price for the contract    |               |
| lotSize              | Integer| Contract lot size                          |                        |
| tickSize             | Integer| Contract tick size                         |                        |
| contractSize         | String | The value of contract                      |                        |
| priceScale           | Integer| Price scaling fator                        |                        |
| ratioScale           | Integer| Ratio scaling fator                        |                        |
| valueScale           | Integer| Value scaling fator                        |                        |
| defaultLeverage      | Integer| Account default leverage                   |                        |
| maxLeverage          | Integer| Allowed maximum leverage                   |                        |
| initMarginEr         | Integer| Scaled initial margin ratio                |                        |
| maintMarginEr        | Integer| Scaled maintainance margin ratio           |                        |
| defaultRiskLimitEv   | Integer| Scaled default risk limit value (Position + Order) |                |
| deleverage           | Boolean| Indicates if deleverage enabled            |                        |
| makerFeeRateEr       | Integer| Scaled maker fee ratio                     |                        |
| takerFeeRateEr       | Integer| Scaled taker fee ratio                     |                        |
| fundingInterval      | Integer| Funding interval in hours                  | 8                      |
| description          | String | Contract description                       |                        |

* Sample：
```json
{
  "code": 0,
  "msg": "OK",
  "data": [
    {
      "symbol": "BTCUSD",
      "underlyingSymbol": ".BTC",
      "quoteCurrency": "USD",
      "settlementCurrency": "BTC",
      "maxOrderQty": 1000000,
      "maxPriceEp": 100000000000000,
      "lotSize": 1,
      "tickSize": "0.5",
      "contractSize": "1 USD",
      "priceScale": 4,
      "ratioScale": 8,
      "valueScale": 8,
      "defaultLeverage": 100,
      "maxLeverage": 100,
      "initMarginEr": "1000000",
      "maintMarginEr": "500000",
      "defaultRiskLimitEv": 10000000000,
      "deleverage": true,
      "makerFeeRateEr": -250000,
      "takerFeeRateEr": 750000,
      "fundingInterval": 8,
      "description": "BTCUSD is a BTC/USD perpetual contract priced on the .BTC Index. Each contract is worth 1 USD of Bitcoin. Funding is paid and received every 8 hours. At UTC time: 00:00, 08:00, 16:00."
    },
    {
      "symbol": "ETHUSD",
      "underlyingSymbol": ".ETH",
      "quoteCurrency": "USD",
      "settlementCurrency": "USD",
      "maxOrderQty": 500000,
      "maxPriceEp": 200000000,
      "lotSize": 1,
      "tickSize": "0.05",
      "contractSize": "0.005 ETH",
      "priceScale": 4,
      "ratioScale": 8,
      "valueScale": 4,
      "defaultLeverage": 20,
      "maxLeverage": 20,
      "initMarginEr": "5000000",
      "maintMarginEr": "1000000",
      "defaultRiskLimitEv": 5000000000,
      "deleverage": true,
      "makerFeeRateEr": -250000,
      "takerFeeRateEr": 750000,
      "fundingInterval": 8,
      "description": "ETHUSD is a ETH/USD perpetual contract priced on the .ETH Index. Each contract is worth 0.005 ETH of USD. Funding is paid and received every 8 hours. At UTC time: 00:00, 08:00, 16:00."
    },
    {
      "symbol": "XRPUSD",
      "underlyingSymbol": ".XRP",
      "quoteCurrency": "USD",
      "settlementCurrency": "USD",
      "maxOrderQty": 500000,
      "maxPriceEp": 2000000,
      "lotSize": 1,
      "tickSize": "0.0001",
      "contractSize": "5 XRP", 
      "priceScale": 4,
      "ratioScale": 8,
      "valueScale": 4,
      "defaultLeverage": 20,
      "maxLeverage": 20,
      "initMarginEr": "5000000",
      "maintMarginEr": "1000000",
      "defaultRiskLimitEv": 5000000000,
      "deleverage": true,
      "makerFeeRateEr": -250000,
      "takerFeeRateEr": 750000,
      "fundingInterval": 8,
      "description": "XRPUSD is a XRP/USD perpetual contract priced on the .XRP Index. Each contract is worth 5 XRP of USD. Funding is paid and received every 8 hours. At UTC time: 00:00, 08:00, 16:00."
    }
  ]
}
```

<a name="orderapilist"/>

### 6.2 Order API List

<a name="placeorder"/>

#### 6.2.1 Place Order 

* HTTP Request:

```json
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
  "stopLossEp": 0
}
```

* HTTP Response:

<a name="cancelsingleorder"/>

#### 6.2.2 Cancel Single Order

* Request
```
DELETE /orders/cancel?symbol={symbol}&orderID=#{orderID}
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
            "takeProfitEp": 0,
            "takeProfit": 0,
            "stopLossEp": 0,
            "closedPnlEv": 0,
            "closedPnl": 0,
            "closedSize": 0,
            "cumQty": 0,
            "cumValueEv": 0,
            "cumValue": 0,
            "leavesQty": 1,
            "leavesValueEv": 12493,
            "leavesValue": 0.00012493,
            "stopLoss": 0,
            "stopDirection": "UNSPECIFIED",
            "ordStatus": "New"
        }
}

```


<a name="cancelorder"/>

#### 6.2.3 Cancel Order

```
DELETE /orders/orderID=<xxx>&symbol=<xxx>
```

<a name="cancelall"/>

#### 6.2.4 Cancel All Orders

```
DELETE /orders/all?symbol={symbol}&untriggered={untriggered}&text={text}
```

| Field       | Type   | Required  | Description                    | Possible values         |
|-------------|--------|-----------|--------------------------------|-------------------------|
| symbol      | String | Yes       | which Symbol to cancel         | BTCUSD,ETHUSD,XRPUSD,.. |
| untriggerred| Boolean| No        | default to false, default cancel non-conditional order; if intending to cancel conditional order, set this to true| true,false|
| text        | comments| No       | comments of this operation, limited to 40 characters  |  |

<a name="querytradeaccount"/>

#### 6.2.5 Query trading account and positions

* Request

```
GET /accounts/accountPositions?currency={currency}
```
| Field       | Type   | Description                                | Possible values |
|-------------|--------|--------------------------------------------|--------------|
| currency | string | in url query parameter. which trading account | BTC,USD |

* Response

```json
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
                "unRealisedPosLossEv": 0,
                "unRealisedPosLoss": null,
                "estimatedOrdLossEv": 0,
                "estimatedOrdLoss": 0,
                "usedBalanceEv": 0,
                "usedBalance": 0,
                "takeProfitEp": 0,
                "takeProfit": null,
                "stopLossEp": 0,
                "stopLoss": null,
                "realisedPnlEv": 0,
                "realisedPnl":
                    null,
                "cumRealisedPnlEv":
                    0,
                "cumRealisedPnl":
                    null
            }
            ]
        }
}
```

<a name="changeleverage"/>

#### 6.2.6 Change leverage

* Request

```
PUT /positions/leverage?symbol={symbol}&leverage={leverage}&leverageEr={leverageEr}
```

| Field                | Type   | Description                                | Possible values |
|----------------------|--------|--------------------------------------------|--------------|
| symbol               | string | which postion needs to change              | BTCUSD, ETHUSD, XRPUSD .. |
| leverage             | integer   | unscaled leverage                       |  |
| leverageEr           | integer   | ratio scaled leverage, leverage wins when both leverage and leverageEr provided|  |


* Response

```
{
    "code": 0,
    "msg": "OK"
}
```

#### 6.2.7 Change position risklimit
<a name = "changerisklimit"/>

* Request

```
PUT /positions/riskLimit?symbol={symbol}&riskLimit={riskLimit}&riskLimitEv={riskLimitEv}
```
| Field                | Type   | Description                                | Possible values |
|----------------------|--------|--------------------------------------------|--------------|
| symbol               | string | which postion needs to change              | BTCUSD, ETHUSD, XRPUSD .. |
| riskLimit             | integer   | unscaled value, reference BTC/USD value scale   |  |
| riskLimitEv           | integer   | value scaled risklimit, riskLimitEv wins when both riskLimit and riskLimitEv provided|  |

<a name="assignposbalance"/>

#### 6.2.8 Assign position balance in isolated marign mode

* Request

***This api is POST***

```
POST /positions/assign?symbol={symbol}&posBalance={posBalance}&posBalanceEv={posBalanceEv}

```
| Field                | Type   | Description                                | Possible values |
|----------------------|--------|--------------------------------------------|--------------|
| symbol               | string | which postion needs to change              | BTCUSD, ETHUSD, XRPUSD .. |
| posBalance             | integer   | unscaled value                       |  |
| posBalanceEv           | integer   | value scaled for position balance, posBalanceEv wins when both posBalance and posBalanceEv provided|  |

<a name="queryopenorder"/>

#### 6.2.9 Query open orders by symbol

   * Order status includes `New`, `PartiallyFilled`, `Filled`, `Canceled`, `Rejected`, `Triggered`, `Untriggered`;
   * Open order status includes `New`, `PartiallyFilled`, `Untriggered`;

* Request

```
GET /orders/activeList?symbol={symbol}&ordStatus={ordStatus}
```

| Field                | Type   | Description                                | Possible values |
|----------------------|--------|--------------------------------------------|--------------|
| symbol | String | which symbol needs to query | BTCUSD, ETHUSD, XRPUSD .. |
| ordStatus | String | order status list filter | New, PartiallyFilled, Untriggered | 


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
                "takeProfitEp": 0,
                "takeProfit": 0,
                "stopLossEp": 0,
                "closedPnlEv": 0,
                "closedPnl": 0,
                "closedSize": 0,
                "cumQty": 0,
                "cumValueEv": 0,
                "cumValue": 0,
                "leavesQty": 0,
                "leavesValueEv": 0,
                "leavesValue": 0,
                "stopLoss": 0,
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
                "takeProfitEp": 0,
                "takeProfit": 0,
                "stopLossEp": 0,
                "closedPnlEv": 0,
                "closedPnl": 0,
                "closedSize": 0,
                "cumQty": 0,
                "cumValueEv": 0,
                "cumValue": 0,
                "leavesQty": 1,
                "leavesValueEv": 10096,
                "leavesValue": 0.00010096,
                "stopLoss": 0,
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
                "takeProfitEp": 0,
                "takeProfit": 0,
                "stopLossEp": 0,
                "closedPnlEv": 0,
                "closedPnl": 0,
                "closedSize": 0,
                "cumQty": 0,
                "cumValueEv": 0,
                "cumValue": 0,
                "leavesQty": 1,
                "leavesValueEv": 12493,
                "leavesValue": 0.00012493,
                "stopLoss": 0,
                "stopDirection": "UNSPECIFIED",
                "ordStatus": "New"
            }
            ]
        }
}

```

<a name="queryorder"/>

#### 6.2.10 Query closed orders by symbol

* Request

```
GET /exchange/order/list?symbol={symbol}&start={start}&end={end}&offset={offset}&limit={limit}&ordStatus={ordStatus}&withCount={withCount}
```

| Field                | Type   | Description                                | Possible values |
|----------------------|--------|--------------------------------------------|--------------|
| symbol | String | which symbol needs to query | BTCUSD, ETHUSD, XRPUSD .. |
| start  | Integer | start time range, Epoch millis | |
| end  | Integer | end time range, Epoch millis | |
| offset | Integer | offset to resultset | | 
| limit | Integer | limit of resultset  | | 
| ordStatus | String | order status list filter | New, PartiallyFilled, Untriggered | 

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

#### 6.2.11 Query user order by orderID
* Request

```
GET /exchange/order?symbol={symbol}&orderID={orderID1,orderID2}
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

#### 6.2.12 Query user trade

* Request

```
GET /exchange/order/trade?symbol={symbol}&start={start}&end={end}&limit={limit}&offset={offset}&withCount={withCount}
```

* Response
  * Response of this API includes normal trade, funding records,  liquidation, ADL trades,etc. `action` can distiguish these types.
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


### 6.2.11 Query user trades by symbol

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
| symbol      | String | Contract symbol name                       | BTCUSD, ETHUSD, XRPUSD |
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
          <price>,
          <size>
        ],
        ...
        ...
        ...
      ],
      "bids": [
        [
          <price>,
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
    "type": "snapshot"
  }
}
```

| Field       | Type   | Description                                | Possible values |
|-------------|--------|--------------------------------------------|--------------|
| price       | Integer| Scaled book level price                    |              |
| size        | Integer| Scaled book level size                     |              |
| sequence    | Integer| current message sequence                   |              |
| symbol      | String | Contract symbol name                       | BTCUSD, ETHUSD, XRPUSD |

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
    "symbol": "BTCUSD",
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
| symbol      | String | Contract symbol name                       | BTCUSD, ETHUSD, XRPUSD |
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
        <price>,
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
| side        | String | Trade side string                          | Buy, Sell    |
| price       | Integer| Scaled trade price                         |              |
| size        | Integer| Scaled trade size                          |              |
| sequence    | Integer| Current message sequence                   |              |
| symbol      | String | Contract symbol name                       | BTCUSD, ETHUSD, XRPUSD |

* Sample：
```json
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

#### 6.3.3 Query 24 Hours Ticker

* Request：
```json
GET /md/ticker/24hr?symbol=<symbol>&id=<id>
```

| Field       | Type   | Description                                | Possible values |
|-------------|--------|--------------------------------------------|--------------|
| symbol      | String | Contract symbol name                       | BTCUSD, ETHUSD, XRPUSD |
| id          | Integer| Optional. Request id                       |              |

* Response:
```
{
  "error": null,
  "id": 0,
  "result": {
    "open": <open price>,
    "high": <high price>,
    "low": <low price>,
    "close": <close price>,
    "openInterest": <open interest>,
    "symbol": "<symbol>",
    "turnover": <turnover>,
    "volume": <volume>
  }
}
```

| Field       | Type   | Description                                | Possible values |
|-------------|--------|--------------------------------------------|--------------|
| open price  | Integer| The scaled open price in last 24 hours     |              |
| high price  | Integer| The scaled highest price in last 24 hours  |              |
| low price   | Integer| The scaled lowest price in last 24 hours   |              |
| close price | Integer| The scaled close price in last 24 hours    |              |
| open interest| Integer| current open interest                     |              |
| symbol      | String | Contract symbol name                       | BTCUSD, ETHUSD, XRPUSD |
| turnover    | Integer| The scaled turnover value in last 24 hours |              |
| volume      | Integer| Symbol trade volume in last 24 hours       |              |

* Sample：
```json
GET /md/ticker/24hr?symbol=BTCUSD

{
  "error": null,
  "id": 0,
  "result": {
    "open": 78025000,
    "high": 82580000,
    "low": 76835000,
    "close": 81775000,
    "openInterest": 2080077,
    "symbol": "BTCUSD",
    "turnover": 701493973632,
    "volume": 55033491
  }
}
```

<a name="assetapilist"/>

### 6.4 Asset api list
   * Asset includes BTC in wallets, BTC in btc-trading account, USD in usd-trading account.
   * In wallet level, Main/parent client can transfer BTC between Sub-client and main/parent client.
   * In wallet level, Sub client can *only* transfer self BTC to main/parent client wallet.
   * client can *only* transfer its own asset between wallet and trading accounts. 

<a name="clientwalletquery"/>

#### 6.4.1 Query client and wallets

* Request

```json
/phemex-user/users/children?offset=0&limit=100&withCount=true
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
                "nickName": "adams",
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

<a name="walletransferout"/>

#### 6.4.2 Main/parent-client transfer self wallet balance to sub-client wallet. (Or Subclient transfer self wallet balance to main/parent client wallet )

* Request
   * Main/parent can transfer its wallet balance to its own subclients.
   * Sub-client can only transfer its wallet balance to its parent/main client.
   * When sub-client transfer its wallet balance, `clientCnt = 0`

```json
POST: /exchange/wallets/transferOut

Body:
{
    "amount": 0, // unscaled amount
    "amountEv": 0, // scaled amount, when both amount and amountEv are provided, amountEv wins.
    "clientCnt": 0, // client number, this is from api in children list; when sub-client issues this api, client must be 0.
    "currency": "string"
}

```

* Response
   * This api is sync, `code == 0` means succeeded. If timed-out, history can be queried.

```json
{"code":0,"msg":"OK","data":"OK"}
```

<a name="walletransferin"/>

#### 6.4.3 Transfer from sub-client wallet. Only main/parent client has priviledge.

* Request
```json
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

***More developer friendly api is yet to come***

<a name="transferwallettradingaccount"/>

#### 6.4.4 Transfer between wallet and trading accounts

* Request
```json
POST /exchange/margins

Body:
{
    "btcAmount": 0.00, 
    "btcAmountEv": 0, 
    "linkKey": "unique-str-for-this-request", 
    "moveOp": [1,2,3,4], 
    "usdAmount": 0.00,
    "usdAmountEv": 0
}
```
   * `btcAmount/btcAmountEv` is required when `moveOp` is `1, 2, 3`. `btcAmountEv` favors over `btcAmount` when both provided. `0.002 btcAmount` or `200000 btcAmountEv` is the minimum when `moveOp` is `3`.
   * `usdAmount/usdAmountEv` is required when `moveOp` is `4`. `usdAmountEv` favors over `usdAmount` when both provided. `1 usdAmount` or `10000 usdAmountEv` is the minimum.
   * All amount must be positive

| Field | Type | Required | Description | Possible values |
|-------|------|----------|--------------|----------------|
| moveOp | Integer | Yes | move operation, four types: 1- From BTC trading account to wallet; 2- From wallet to BTC trading account; 3 - From wallet to USD trading account; 4 - From USD trading account to wallet | 1,2,3,4 |
| linkKey | String | No |  used for idempotency, unique string for one request, recommend UUID. System can generate it when not provided | | 
| btcAmount | BigDecimal | - | unscaled BTC amount with at most 8 precision, optional | |
| btcAmountEv | Integer | - |scaled BTC amount | |
| usdAmount | BigDecimal | - | unscaled amount with at most 4 preision. | |
| usdAmountEv | Integer | - | scaled USD amount | |

* Response

   * `status == 10` in the response body implies the operation succeeded or not, as underlying operation is async.

```json
{
    "code": 0,
        "msg": "OK",
        "data": {
            "moveOp": 1,
            "fromCurrencyName": "BTC",
            "toCurrencyName": "BTC",
            "fromAmount": "0.10000000",
            "toAmount": "0.10000000",
            "linkKey": "2431ca9b-2dd4-44b8-91f3-2539bb62db2d",
            "status": 10,
        }
}

```

<a name="transferwallettradingaccountquery"/>

#### 6.4.5 Query wallet/tradingaccount transfer history

* Request

```json
GET /exchange/margins/transfer?start=0&end=0&offset=0&limit=50&withCount=true
```

| Filed | Type | Required |  Description | Possible values |
|-------|------|----------|--------------|-----------------|
| start | Integer | No | epoch millis of start of time range, include | |
| end | Integer | No | epoch millis of start of time range, exclude | |
| offset | Integer | No | default 0, offset of the resultset | |
| limit | Integer | No | default 50(subject to change) | |
| withCount | Boolean | No | whether total records is required | true,false|

* Response
```json
{
    "code": 0,
        "msg": "OK",
        "data": {
            "total": 589,
            "rows": [
            {
                "moveOp": 1,
                "fromCurrencyName": "BTC",
                "toCurrencyName": "BTC",
                "fromAmount": "0.10000000",
                "toAmount": "0.10000000",
                "linkKey": "2431ca9b-2dd4-44b8-91f3-2539bb62db2d",
                "status": 10,
                "createTime": 1580201427000
            },
            {
                "moveOp": 4,
                "fromCurrencyName": "USD",
                "toCurrencyName": "BTC",
                "fromAmount": "8310.12000000",
                "toAmount": "0.99748508",
                "linkKey": "r2-move-usd-200076-20200126",
                "status": 10,
                "createTime": 1579976362000
            },
            ...
                ]
        }
}
```
<a name="withdraw"/>

### 6.5 Withdraw
   * Several restrictions are required for withdraw: 1. bind Google 2FA, 2. Password change out of 24hour, 3. meet minimum BTC amount requirement.

<a name="requestwithdraw"/>

#### 6.5.1 Request Withdraw

* Request

```
POST /exchange/wallets/createWithdraw?otpCode={otpCode}
Body: 
{
      "address": "address_stored_in_phemex",// address must set before withdraw
      "amountEv": 1000000000, // scaled btc value
      "currency": "BTC" // fixed to BTC 
}
```
| Filed | Type | Required |  Description | Possible values |
|------|------|----------|--------------|-----------------|
| otpCode | String | Yes | In URL query, From Google 2FA| |


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
    "data": withdrawRequestId // subject to change to full withdraw request
}

```

<a name="confirmwithdraw"/>

#### 6.5.3 Confirm withdraw

   * After withdraw request submitted, a confirmation link is sent to registration email. The confirm code should be extracted out from the link and then passed in as url query parameter.

* Request
```
GET /exchange/wallets/confirm/withdraw?code={withdrawConfirmCode}
```

* Response

```
{
    "code": 0,
    "msg" : "OK"
}
```

<a name="cancelwithdraw"/>

#### 6.5.2 Cancel withdraw
   * Withdraw request can be canceled before mannual `review`;

* Request

```
POST /exchange/wallets/cancelWithdraw
Body:
{
    id: withdrawRequestId
}
```

<a name="withdrawaddrmgmt"/>

#### 6.5.3 Withdraw address management
   * Withdraw address management support create, remove and list. Recommend manage it from website.

* Request

```
POST /exchange/wallets/createWithdrawAddress?otpCode={optCode}
Body: 
{
    "address": "valid_btc_address",
    "currency": "BTC",
    "remark": "remark"
}
```

* Response
```
{
    "code": 0,
    "msg": "OK",
    "data": 1
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
| token       | String | Access token     |                 |
| signature   | String | Signature generated by a funtion as HMacSha256(access-token + expiry) ||
| expiry      | Integer| Expiry in epoch second which is between now and now+5 minutes  ||

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

* Message Format：
 
```
{
  "book": {
    "asks": [
      [
        <price>,
        <qty>
      ],
      .
      .
      .
    ],
    "bids": [
      [
        <price>,
        <qty>
      ],
      .
      .
      .
    ]
  },
  "depth": <depth>,
  "sequence": <sequence>,
  "symbol": "<symbol>",

```

| Field       | Type   | Description      | Possible values |
|-------------|--------|------------------|-----------------|
| side        | String | Price level side | bid, ask        |
| price       | Integer| Scaled price     |                 |
| qty         | Integer| Price level size. Non-zero qty indicates price level insertion or updation, and qty 0 indicates price level deletion |                 |
| sequence    | Integer| Latest message sequence |          |
| depth       | Integer| Market depth     |                 |
| type        | String | Message type     | snapshot, incremental |
  

* Sample：
 
```json
< {"book":{"asks":[[86765000,19609],[86770000,7402],[86775000,3807],[86780000,7395],[86785000,3599],[86790000,7253],[86795000,4019],[86800000,4366],[86805000,3216],[86810000,3107],[86815000,7453],[86820000,1771],[86825000,895],[86830000,3420],[86835000,1818],[86840000,1272],[86845000,1064],[86850000,195],[86855000,1630],[86860000,1017],[86865000,3509],[86870000,1105],[86875000,1262],[86880000,893],[86885000,862],[86890000,1030],[86895000,2315],[86900000,2994],[86905000,2026],[86910000,3387],[86915000,1382],[86920000,1202],[86925000,3150],[86930000,1773],[86935000,1778],[86940000,1384],[86945000,1842],[86950000,1019],[86955000,2660],[86960000,1599],[86965000,920],[86970000,1834],[86975000,752],[86980000,1384],[86985000,2471],[86990000,2133],[86995000,2981],[87000000,1091],[87005000,994],[87010000,1217],[87015000,1098],[87020000,526],[87025000,1779],[87030000,1098],[87035000,892],[87040000,2168],[87045000,822],[87050000,2410],[87055000,630],[87060000,1684],[87065000,2556],[87070000,19],[87080000,1445],[87085000,29],[87105000,2002],[87115000,658],[87120000,660],[87905000,991]],"bids":[[86760000,18995],[86755000,6451],[86750000,5311],[86745000,6867],[86740000,6180],[86735000,3127],[86730000,4852],[86725000,6213],[86720000,3902],[86715000,4510],[86710000,10063],[86705000,1118],[86700000,1891],[86695000,767],[86690000,20920],[86685000,2535],[86680000,1105],[86675000,645],[86670000,1424],[86665000,1773],[86660000,1464],[86655000,1160],[86650000,1462],[86645000,2446],[86640000,538],[86635000,506],[86630000,2291],[86625000,2981],[86620000,1712],[86615000,984],[86610000,1058],[86605000,1261],[86600000,1074],[86595000,1408],[86590000,717],[86585000,1582],[86580000,1950],[86575000,1540],[86570000,2960],[86565000,598],[86560000,759],[86555000,1266],[86550000,1943],[86545000,259],[86540000,2106],[86535000,2365],[86530000,857],[86525000,1200],[86520000,2371],[86515000,2103],[86510000,1468],[86505000,747],[86500000,1369],[86495000,2121],[86490000,3674],[86485000,1345],[86480000,1290],[86475000,1716],[86470000,1851],[86465000,1861],[86460000,1092],[86435000,21],[86430000,986],[86420000,1202],[86415000,22],[86405000,1199],[86390000,470],[86365000,920],[86360000,192],[86355000,474],[86350000,1838],[86335000,1104],[86285000,2205],[86280000,2390],[86275000,95],[86255000,2836],[86250000,589],[86240000,424],[86235000,937],[86225000,374],[86220000,1591],[86215000,517],[86210000,559],[86205000,702],[86190000,54]]},"depth":100,"sequence":1191904,"symbol":"BTCUSD","type":"snapshot"}
< {"book":{"asks":[[86775000,4621]],"bids":[]},"depth":100,"sequence":1191905,"symbol":"BTCUSD","type":"incremental"}
< {"book":{"asks":[],"bids":[[86755000,8097]]},"depth":100,"sequence":1191906,"symbol":"BTCUSD","type":"incremental"}
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

```
{
  "trades": [
    [
      <timestamp>,
      "<side>",
      <price>,
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
| price       | Integer| Scaled execution price  |                 |
| qty         | Integer| Execution size   |                 |
| sequence    | Integer| Latest message sequence ||
| symbol      | String | Contract symbol name     ||
| type        | String | Message type     |snapshot, incremental |
  

* Sample
```json
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


<a name="aopsub"/>

### 3.7 Subscribe Account-Order-Position (AOP)

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
```json
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

```json
< {"accounts":[{"accountBalanceEv":100000024,"accountID":675340001,"bonusBalanceEv":0,"currency":"BTC","totalUsedBalanceEv":1222,"userID":67534}],"orders":[{"accountID":675340001,"action":"New","actionBy":"ByUser","actionTimeNs":1573711481897337000,"addedSeq":1110523,"bonusChangedAmountEv":0,"clOrdID":"uuid-1573711480091","closedPnlEv":0,"closedSize":0,"code":0,"cumQty":2,"cumValueEv":23018,"curAccBalanceEv":100000005,"curAssignedPosBalanceEv":0,"curBonusBalanceEv":0,"curLeverageEr":0,"curPosSide":"Buy","curPosSize":2,"curPosTerm":1,"curPosValueEv":23018,"curRiskLimitEv":10000000000,"currency":"BTC","cxlRejReason":0,"displayQty":2,"execFeeEv":-5,"execID":"92301512-7a79-5138-b582-ac185223727d","execPriceEp":86885000,"execQty":2,"execSeq":1131034,"execStatus":"MakerFill","execValueEv":23018,"feeRateEr":-25000,"lastLiquidityInd":"AddedLiquidity","leavesQty":0,"leavesValueEv":0,"message":"No error","ordStatus":"Filled","ordType":"Limit","orderID":"e9a45803-0af8-41b7-9c63-9b7c417715d9","orderQty":2,"pegOffsetValueEp":0,"priceEp":86885000,"relatedPosTerm":1,"relatedReqNum":2,"side":"Buy","stopLossEp":0,"stopPxEp":0,"symbol":"BTCUSD","takeProfitEp":0,"timeInForce":"GoodTillCancel","tradeType":"Trade","transactTimeNs":1573712555309040417,"userID":67534},{"accountID":675340001,"action":"New","actionBy":"ByUser","actionTimeNs":1573711490507067000,"addedSeq":1110980,"bonusChangedAmountEv":0,"clOrdID":"uuid-1573711488668","closedPnlEv":0,"closedSize":0,"code":0,"cumQty":3,"cumValueEv":34530,"curAccBalanceEv":100000013,"curAssignedPosBalanceEv":0,"curBonusBalanceEv":0,"curLeverageEr":0,"curPosSide":"Buy","curPosSize":5,"curPosTerm":1,"curPosValueEv":57548,"curRiskLimitEv":10000000000,"currency":"BTC","cxlRejReason":0,"displayQty":3,"execFeeEv":-8,"execID":"80899855-5b95-55aa-b84e-8d1052f19886","execPriceEp":86880000,"execQty":3,"execSeq":1131408,"execStatus":"MakerFill","execValueEv":34530,"feeRateEr":-25000,"lastLiquidityInd":"AddedLiquidity","leavesQty":0,"leavesValueEv":0,"message":"No error","ordStatus":"Filled","ordType":"Limit","orderID":"7e03cd6b-e45e-48d9-8937-8c6628e7a79d","orderQty":3,"pegOffsetValueEp":0,"priceEp":86880000,"relatedPosTerm":1,"relatedReqNum":3,"side":"Buy","stopLossEp":0,"stopPxEp":0,"symbol":"BTCUSD","takeProfitEp":0,"timeInForce":"GoodTillCancel","tradeType":"Trade","transactTimeNs":1573712559100655668,"userID":67534},{"accountID":675340001,"action":"New","actionBy":"ByUser","actionTimeNs":1573711499282604000,"addedSeq":1111025,"bonusChangedAmountEv":0,"clOrdID":"uuid-1573711497265","closedPnlEv":0,"closedSize":0,"code":0,"cumQty":4,"cumValueEv":46048,"curAccBalanceEv":100000024,"curAssignedPosBalanceEv":0,"curBonusBalanceEv":0,"curLeverageEr":0,"curPosSide":"Buy","curPosSize":9,"curPosTerm":1,"curPosValueEv":103596,"curRiskLimitEv":10000000000,"currency":"BTC","cxlRejReason":0,"displayQty":4,"execFeeEv":-11,"execID":"0be06645-90b8-5abe-8eb0-dca8e852f82f","execPriceEp":86865000,"execQty":4,"execSeq":1132422,"execStatus":"MakerFill","execValueEv":46048,"feeRateEr":-25000,"lastLiquidityInd":"AddedLiquidity","leavesQty":0,"leavesValueEv":0,"message":"No error","ordStatus":"Filled","ordType":"Limit","orderID":"66753807-9204-443d-acf9-946d15d5bedb","orderQty":4,"pegOffsetValueEp":0,"priceEp":86865000,"relatedPosTerm":1,"relatedReqNum":4,"side":"Buy","stopLossEp":0,"stopPxEp":0,"symbol":"BTCUSD","takeProfitEp":0,"timeInForce":"GoodTillCancel","tradeType":"Trade","transactTimeNs":1573712618104628671,"userID":67534}],"positions":[{"accountID":675340001,"assignedPosBalanceEv":0,"avgEntryPriceEp":86875941,"bankruptCommEv":75022,"bankruptPriceEp":90000,"buyLeavesQty":0,"buyLeavesValueEv":0,"buyValueToCostEr":1150750,"createdAtNs":0,"crossSharedBalanceEv":99998802,"cumClosedPnlEv":0,"cumFundingFeeEv":0,"cumTransactFeeEv":-24,"currency":"BTC","dataVer":4,"deleveragePercentileEr":0,"displayLeverageEr":1000000,"estimatedOrdLossEv":0,"execSeq":1132422,"freeCostEv":0,"freeQty":-9,"initMarginReqEr":1000000,"lastFundingTime":1573703858883133252,"lastTermEndTime":0,"leverageEr":0,"liquidationPriceEp":90000,"maintMarginReqEr":500000,"makerFeeRateEr":0,"markPriceEp":86786292,"orderCostEv":0,"posCostEv":1115,"positionMarginEv":99925002,"positionStatus":"Normal","riskLimitEv":10000000000,"sellLeavesQty":0,"sellLeavesValueEv":0,"sellValueToCostEr":1149250,"side":"Buy","size":9,"symbol":"BTCUSD","takerFeeRateEr":0,"term":1,"transactTimeNs":1573712618104628671,"unrealisedPnlEv":-107,"updatedAtNs":0,"usedBalanceEv":1222,"userID":67534,"valueEv":103596}],"sequence":1310812,"timestamp":1573716998131003833,"type":"snapshot"}
< {"accounts":[{"accountBalanceEv":99999989,"accountID":675340001,"bonusBalanceEv":0,"currency":"BTC","totalUsedBalanceEv":1803,"userID":67534}],"orders":[{"accountID":675340001,"action":"New","actionBy":"ByUser","actionTimeNs":1573717286765750000,"addedSeq":1192303,"bonusChangedAmountEv":0,"clOrdID":"uuid-1573717284329","closedPnlEv":0,"closedSize":0,"code":0,"cumQty":0,"cumValueEv":0,"curAccBalanceEv":100000024,"curAssignedPosBalanceEv":0,"curBonusBalanceEv":0,"curLeverageEr":0,"curPosSide":"Buy","curPosSize":9,"curPosTerm":1,"curPosValueEv":103596,"curRiskLimitEv":10000000000,"currency":"BTC","cxlRejReason":0,"displayQty":4,"execFeeEv":0,"execID":"00000000-0000-0000-0000-000000000000","execPriceEp":0,"execQty":0,"execSeq":1192303,"execStatus":"New","execValueEv":0,"feeRateEr":0,"leavesQty":4,"leavesValueEv":46098,"message":"No error","ordStatus":"New","ordType":"Limit","orderID":"e329ae87-ce80-439d-b0cf-ad65272ed44c","orderQty":4,"pegOffsetValueEp":0,"priceEp":86770000,"relatedPosTerm":1,"relatedReqNum":5,"side":"Buy","stopLossEp":0,"stopPxEp":0,"symbol":"BTCUSD","takeProfitEp":0,"timeInForce":"GoodTillCancel","transactTimeNs":1573717286765896560,"userID":67534},{"accountID":675340001,"action":"New","actionBy":"ByUser","actionTimeNs":1573717286765750000,"addedSeq":1192303,"bonusChangedAmountEv":0,"clOrdID":"uuid-1573717284329","closedPnlEv":0,"closedSize":0,"code":0,"cumQty":4,"cumValueEv":46098,"curAccBalanceEv":99999989,"curAssignedPosBalanceEv":0,"curBonusBalanceEv":0,"curLeverageEr":0,"curPosSide":"Buy","curPosSize":13,"curPosTerm":1,"curPosValueEv":149694,"curRiskLimitEv":10000000000,"currency":"BTC","cxlRejReason":0,"displayQty":4,"execFeeEv":35,"execID":"8d1848a2-5faf-52dd-be71-9fecbc8926be","execPriceEp":86770000,"execQty":4,"execSeq":1192303,"execStatus":"TakerFill","execValueEv":46098,"feeRateEr":75000,"lastLiquidityInd":"RemovedLiquidity","leavesQty":0,"leavesValueEv":0,"message":"No error","ordStatus":"Filled","ordType":"Limit","orderID":"e329ae87-ce80-439d-b0cf-ad65272ed44c","orderQty":4,"pegOffsetValueEp":0,"priceEp":86770000,"relatedPosTerm":1,"relatedReqNum":5,"side":"Buy","stopLossEp":0,"stopPxEp":0,"symbol":"BTCUSD","takeProfitEp":0,"timeInForce":"GoodTillCancel","tradeType":"Trade","transactTimeNs":1573717286765896560,"userID":67534}],"positions":[{"accountID":675340001,"assignedPosBalanceEv":0,"avgEntryPriceEp":86843828,"bankruptCommEv":75056,"bankruptPriceEp":130000,"buyLeavesQty":0,"buyLeavesValueEv":0,"buyValueToCostEr":1150750,"createdAtNs":0,"crossSharedBalanceEv":99998186,"cumClosedPnlEv":0,"cumFundingFeeEv":0,"cumTransactFeeEv":11,"currency":"BTC","dataVer":5,"deleveragePercentileEr":0,"displayLeverageEr":1000000,"estimatedOrdLossEv":0,"execSeq":1192303,"freeCostEv":0,"freeQty":-13,"initMarginReqEr":1000000,"lastFundingTime":1573703858883133252,"lastTermEndTime":0,"leverageEr":0,"liquidationPriceEp":130000,"maintMarginReqEr":500000,"makerFeeRateEr":0,"markPriceEp":86732335,"orderCostEv":0,"posCostEv":1611,"positionMarginEv":99924933,"positionStatus":"Normal","riskLimitEv":10000000000,"sellLeavesQty":0,"sellLeavesValueEv":0,"sellValueToCostEr":1149250,"side":"Buy","size":13,"symbol":"BTCUSD","takerFeeRateEr":0,"term":1,"transactTimeNs":1573717286765896560,"unrealisedPnlEv":-192,"updatedAtNs":0,"usedBalanceEv":1803,"userID":67534,"valueEv":149694}],"sequence":1315725,"timestamp":1573717286767188294,"type":"incremental"}

```


<a name="aopunsub"/>

### 3.8 Unsubscribe Account-Order-Position (AOP)
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

### 3.9 Subscribe 24 Hours Ticker
On each successful subscription, DataGW will publish 24-hour ticker metrics for all symbols every 5 seconds.

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

```json
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

#### 24 Hours Ticker Message Format：

```
{
  "market24h": {
    "close": <24h close price>,
    "high": <24h highest price>,
    "low": <24h lowest price>,
    "open": <24h open price>,
    "openInterest": <open interest>,
    "symbol": "<symbol>",
    "turnover": <turnover>,
    "volume": <volume>
  },
  "timestamp": <timestamp>
}
```

| Field       | Type   | Description      | Possible values |
|-------------|--------|------------------|-----------------|
| timestamp         | Integer| Last update timestamp in nanoseconds ||
| symbol            | String | Contract symbol name    | BTCUSD, ETHUSD, XRPUSD |
| 24h open price    | Integer| Open price for last 24 hours |                 |
| 24h highest price | Integer| Highest price in last 24 hours |                 |
| 24h lowest price  | Integer| Lowest price in last 24 hours |                 |
| 24h close price   | Integer| Close price for last 24 hours |                 |
| 24h turnover      | Integer| Turnover for last 24 hours |                 |
| 24h volume        | Integer| Volume for last 24 hours |                 |
| open interest     | Integer| Current open interest for the related symbol|                 |
  
* Sample:

```json
< {
  "market24h": {
    "close": 70485000,
    "high": 71465000,
    "low": 70225000,
    "open": 71300000,
    "openInterest": 153502,
    "symbol": "BTCUSD",
    "turnover": 62419527476,
    "volume": 4419810
  },
  "timestamp": 1576490244024818000
}
```

<a name="tickerunsub"/>

### 3.10 Unsubscribe 24 Hours Ticker

It unsubscribes 24-hour ticker subscription.

* Request

```
{
  "id": <id>,
  "method": "market24h.unsubscribe",
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
