## Table of Contents * [Phemex Public API](#publicapi) * [General Public API Information](#general)
* [REST API Standards](#restapi)
  * [REST API List](#restapilist)
    * [Trade API List](#orderapilist)
      * [Place Order With Put Method, *Prefered*](#placeorderwithput)
      * [Place Order](#placeorder)
      * [Amend order by orderID](#amendorder)
      * [Cancel Single Order by orderID or clOrdID](#cancelsingleorder)
      * [Bulk Cancel Orders](#cancelorder)
      * [Cancel All Orders](#cancelall)
      * [Query Account Positions](#queryaccountpositions)
      * [Switch Position Mode Synchronously](#switchpositionmodesync)
      * [Set Leverage](#setleverage)
      * [Set RiskLimit](#setrisklimit)
      * [Assign Position Balance](#assignpositionbalance)
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
    * [Future Data Api List](#futureDataAPIList)
    * [Withdraw](#withdraw)
* [Websocket API Standards](#wsapi)
  * [Session Management](#sessionmanagement)
  * [API Rate Limits](#wsapiratelimits)
  * [WebSocket API List](#wsapilist)
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



<a name="placeorderwithput"/>

#### Place order with argument in url query string

* HTTP Request

```
PUT /g-orders/create?clOrdID=<clOrdID>&symbol=<symbol>&reduceOnly=<reduceOnly>&closeOnTrigger=<closeOnTrigger>&orderQtyRq=<orderQtyRq>&ordType=<ordType>&priceRp=<priceRp>&side=<side>&posSide=<posSide>&text=<text>&timeInForce=<timeInForce>&stopPxRp=<stopPxRp>&takeProfitRp=<takeProfitRp>&stopLossRp=<stopLossRp>&pegOffsetValueRp=<pegOffsetValueRp>&pegPriceType=<pegPriceType>&triggerType=<triggerType>&tpTrigger=<tpTrigger>&tpSlTs=<tpSlTs>&slTrigger=<slTrigger>
```

| Field            | Type    | Required | Description                                                  | Possible values                                              |
| ---------------- | ------- | -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| clOrdID          | String  | -        | client order id, max length is 40                            |                                                              |
| symbol           | String  | Yes      | Which symbol to place order                                  | [Trading symbols](#symbpricesub)                             |
| reduceOnly       | Boolean | -        | whether reduce position side only. Enable this flag, i.e. reduceOnly=true, position side won't change | true, false                                                  |
| closeOnTrigger   | Boolean | -        | implicitly reduceOnly, plus cancel other orders in the same direction(side) when necessary | true, false                                                  |
| orderQtyRq       | String  | -        | Order real quantity                                          | "1"                                                          |
| ordType          | String  | -        | Order type, default to Limit                                 | Market,Limit,Stop,StopLimit,MarketIfTouched,LimitIfTouched,<br />ProtectedMarket,MarketAsLimit,StopAsLimit,<br />MarketIfTouchedAsLimit,Bracket,BoTpLimit,BoSlLimit,BoSlMarket |
| priceRp          | String  | -        | Real price, required for limit order                         | "1"                                                          |
| side             | String  | Yes      | Order direction, Buy or Sell                                 | Buy, Sell                                                    |
| posSide          | String  | Yes      | Position direction                                           | "Merged" for oneway mode , <br />"Long" / "Short" for hedge mode |
| text             | String  | -        | Order comments                                               |                                                              |
| timeInForce      | String  | -        | Time in force. default to GoodTillCancel                     | GoodTillCancel, ImmediateOrCancel, FillOrKill, PostOnly      |
| stopPxRp         | String  | -        | Trigger price for stop orders                                | "1"                                                          |
| takeProfitRp     | String  | -        | Real take profit price                                       | "1"                                                          |
| stopLossRp       | String  | -        | Real stop loss price                                         | "1"                                                          |
| pegOffsetValueRp | String  | -        | Trailing offset from current price. Negative value when position is long, positive when position is short | "1"                                                          |
| pegPriceType     | String  | -        | Trailing order price type                                    | LastPeg, MidPricePeg, MarketPeg, PrimaryPeg, TrailingStopPeg, TrailingTakeProfitPeg |
| triggerType      | String  | -        | Trigger source                                               | ByMarkPrice, ByIndexPrice, ByLastPrice, ByAskPrice, ByBidPrice, ByMarkPriceLimit, ByLastPriceLimit |
| tpTrigger        | String  | -        | Trigger source                                               | ByMarkPrice, ByIndexPrice, ByLastPrice, ByAskPrice, ByBidPrice, ByMarkPriceLimit, ByLastPriceLimit |
| slTrigger        | String  | -        | Trigger source                                               | ByMarkPrice, ByIndexPrice, ByLastPrice, ByAskPrice, ByBidPrice, ByMarkPriceLimit, ByLastPriceLimit |

* HTTP Response:

```json
{
  "code": 0,
  "data": {
    "actionTimeNs": 1580547265848034600,
    "bizError": 0,
    "clOrdID": "137e1928-5d25-fecd-dbd1-705ded659a4f",
    "closedPnlRv": "1271.9",
    "closedSizeRq": "0.01",
    "cumQtyRq": "0.01",
    "cumValueRv": "1271.9",
    "displayQtyRq": "0.01",
    "execInst": "ReduceOnly",
    "execStatus": "Init",
    "leavesQtyRq": "0.01",
    "leavesValueRv": "1271.9",
    "ordStatus": "Init",
    "orderID": "ab90a08c-b728-4b6b-97c4-36fa497335bf",
    "orderQtyRq": "0.01",
    "orderType": "Limit",
    "pegOffsetValueRp": "1271.9",
    "pegPriceType": "LastPeg",
    "priceRq": "98970000",
    "reduceOnly": true,
    "side": "Sell",
    "stopDirection": "Rising",
    "stopPxRp": "1271.9",
    "symbol": "BTCUSDT",
    "timeInForce": "GoodTillCancel",
    "transactTimeNs": 0,
    "trigger": "ByMarkPrice"
  },
  "msg": "string"
}
```

* Important fields description
* More order type examples



<a name="placeorder"/>

#### Place Order

* HTTP Request:

  Request fields are the same as [above place-order](#placeorderwithput)

```json
POST /g-orders

body:
{
  "clOrdID": "137e1928-5d25-fecd-dbd1-705ded659a4f",
  "closeOnTrigger": true,
  "displayQtyRq": "0.01",
  "ordType": "Limit",
  "orderQtyRq": "0.01",
  "pegOffsetValueRp": "1271.9",
  "pegPriceType": "LastPeg",
  "posSide": "Long",
  "priceRp": "1271.9",
  "reduceOnly": true,
  "side": "Buy",
  "slTrigger": "ByMarkPrice",
  "stopLossRp": "1271.9",
  "stopPxRp": "1271.9",
  "symbol": "BTCUSDT",
  "takeProfitRp": "1271.9",
  "text": "string",
  "timeInForce": "GoodTillCancel",
  "tpTrigger": "ByMarkPrice",
  "triggerType": "ByMarkPrice"
}
```

* Response

​	Response is the same as [above place-order](#placeorderwithput)



<a name="amendorder"/>

#### Amend order by orderID

* HTTP Request

```
PUT /g-orders/replace?symbol=<symbol>&orderID=<orderID>&origClOrdID=<origClOrdID>&clOrdID=<clOrdID>&price=<price>&priceRp=<priceRp>&orderQtyRq=<orderQtyRq>&stopPxRp=<stopPxRp>&takeProfitRp=<takeProfitRp>&stopLossRp=<stopLossRp>&pegOffsetValueRp=<pegOffsetValueRp>&pegPriceType=<pegPriceType>&triggerType=<triggerType>&posSide=<posSide>
```

| Field            | Required | Description                           |
| ---------------- | -------- | ------------------------------------- |
| symbol           | Yes      | order symbol, cannot be changed       |
| orderID          | Yes      | order id, cannot be changed           |
| origClOrdID      | -        | original clOrderID, cannot be changed |
| clOrdID          | -        | new clOrdID                           |
| priceRp          | -        | new order price, real value           |
| orderQtyRq       | Yes      | new orderQty, real value              |
| stopPxRp         | Yes      | new stop price, real value            |
| takeProfitRp     | Yes      | new stop profit price, real value     |
| stopLossRp       | Yes      | new stop loss price, real value       |
| pegOffsetValueRp | Yes      | new trailing offset, real value       |
| pegPriceType     | Yes      | new peg price type                    |
| triggerType      | Yes      | new triggerType                       |
| posSide          | Yes      | posSide to check, can not be changed  |

* Response

```json
{
  "code": 0,
  "data": {
    "actionTimeNs": 0,
    "bizError": 0,
    "clOrdID": "137e1928-5d25-fecd-dbd1-705ded659a4f",
    "closedPnlRv": "1271.9",
    "closedSizeRq": "0.01",
    "cumQtyRq": "0.01",
    "cumValueRv": "1271.9",
    "displayQtyRq": "0.01",
    "execInst": "ReduceOnly",
    "execStatus": "Init",
    "leavesQtyRq": "0.01",
    "leavesValueRv": "1271.9",
    "ordStatus": "Init",
    "orderID": "ab90a08c-b728-4b6b-97c4-36fa497335bf",
    "orderQtyRq": "0.01",
    "orderType": "Limit",
    "pegOffsetValueRp": "1271.9",
    "pegPriceType": "LastPeg",
    "priceRp": "1271.9",
    "reduceOnly": true,
    "side": "Sell",
    "stopDirection": "Rising",
    "stopPxRp": "1271.9",
    "symbol": "BTCUSDT",
    "timeInForce": "GoodTillCancel",
    "transactTimeNs": 0,
    "trigger": "ByMarkPrice",
    "takeProfitRp":"1271.9",
    "stopLossRp":"1271.9"
  },
  "msg": "string"
}
```



<a name="cancelsingleorder"/>

#### Cancel Single Order by orderID

* HTTP Request

```
DELETE /g-orders/cancel?orderID=<orderID>&posSide=<posSide>&symbol=<symbol>
```

| Field   | Type   | Required | Description                  |
| ------- | ------ | -------- | ---------------------------- |
| orderID | String | Yes      | order id, cannot be changed  |
| symbol  | String | Yes      | which symbol to cancel order |
| posSide | String | Yes      | position direction           |

* Response

```json
{
  "code": 0,
  "data": {
    "actionTimeNs": 450000000,
    "bizError": 0,
    "clOrdID": "137e1928-5d25-fecd-dbd1-705ded659a4f",
    "closedPnlRv": "1271.9",
    "closedSizeRq": "0.01",
    "cumQtyRq": "0.01",
    "cumValueRv": "1271.9",
    "displayQtyRq": "0.01",
    "execInst": "ReduceOnly",
    "execStatus": "Init",
    "leavesQtyRq": "0.01",
    "leavesValueRv": "0.01",
    "ordStatus": "Init",
    "orderID": "ab90a08c-b728-4b6b-97c4-36fa497335bf",
    "orderQtyRq": "0.01",
    "orderType": "Limit",
    "pegOffsetValueRp": "1271.9",
    "pegPriceType": "LastPeg",
    "priceRq": "0.01",
    "reduceOnly": true,
    "side": "Sell",
    "stopDirection": "Rising",
    "stopPxRp": "0.01",
    "symbol": "BTCUSDT",
    "timeInForce": "GoodTillCancel",
    "transactTimeNs": 450000000,
    "trigger": "ByMarkPrice"
  },
  "msg": "string"
}
```



<a name="cancelorder"/>

#### Bulk Cancel Orders

* Request

```
DELETE /g-orders?symbol=<symbol>&orderID=<orderID1>,<orderID2>,<orderID3>&posSide=<posSide>
```

| Field   | Type   | Required | Description                                          |
| ------- | ------ | -------- | ---------------------------------------------------- |
| orderID | String | Yes      | list of order ids to be cancelled, cannot be changed |
| symbol  | String | Yes      | which symbol to cancel order                         |
| posSide | String | Yes      | position direction                                   |

* Response

```json
{
  "code": 0,
  "data": {
    "actionTimeNs": 450000000,
    "bizError": 0,
    "clOrdID": "137e1928-5d25-fecd-dbd1-705ded659a4f",
    "closedPnlRv": "1271.9",
    "closedSizeRq": "0.01",
    "cumQtyRq": "0.01",
    "cumValueRv": "1271.9",
    "displayQtyRq": "0.01",
    "execInst": "ReduceOnly",
    "execStatus": "Init",
    "leavesQtyRq": "0.01",
    "leavesValueRv": "1271.9",
    "ordStatus": "Init",
    "orderID": "ab90a08c-b728-4b6b-97c4-36fa497335bf",
    "orderQtyRq": "0.01",
    "orderType": "Limit",
    "pegOffsetValueRp": "1271.9",
    "pegPriceType": "LastPeg",
    "priceRq": "0.01",
    "reduceOnly": true,
    "side": "string",
    "stopDirection": "Rising",
    "stopPxRp": "1271.9",
    "symbol": "BTCUSDT",
    "timeInForce": "GoodTillCancel",
    "transactTimeNs": 450000000,
    "trigger": "ByMarkPrice"
  },
  "msg": "string"
}
```



<a name="cancelall"/>

#### Cancel All Orders

* Cancel all orders for hedge supported symbols.
* In order to cancel all orders, include conditional order and active order, one must invoke this API twice with different arguments.
  * `untriggered=false` to cancel active order including triggerred conditional order.
  * `untriggered=true` to cancel conditional order, the order is not triggerred.

* Request

```
DELETE /g-orders/all?symbol=<symbol>&untriggered=<untriggered>&text=<text>
```

| Field       | Type    | Required | Description                          |
| ----------- | ------- | -------- | ------------------------------------ |
| symbol      | String  | -        | list of symbols to cancel all orders |
| untriggered | boolean | -        |                                      |
| text        | String  | -        |                                      |

* Response

```json
{
  "code": 0,
  "data": 0,
  "msg": "string"
}
```



<a name="queryaccountpositions"/>

#### Query Account Positions

* Request

```
GET /g-accounts/accountPositions?currency=<currency>&symbol=<symbol>
```

| Field    | Type   | Required | Description |
| -------- | ------ | -------- | ----------- |
| symbol   | String | -        |             |
| currency | String | Yes      |             |

* Response

```json
{
  "code": 0,
  "data": {
    "account": {
      "accountBalanceRv": "1271.9",
      "accountId": 123450001,
      "bonusBalanceRv": "1271.9",
      "currency": "BTC",
      "totalUsedBalanceRv": "1271.9",
      "userID": 12345
    },
    "positions": [
      {
        "accountID": 123450001,
        "assignedPosBalanceRv": "1271.9",
        "avgEntryPrice": "1271.9",
        "avgEntryPriceRp": "1271.9",
        "bankruptCommRv": "1271.9",
        "bankruptPriceRp": "1271.9",
        "buyValueToCostRr": "0.1",
        "crossMargin": true,
        "cumClosedPnlRv": "1271.9",
        "cumFundingFeeRv": "1271.9",
        "cumTransactFeeRv": "1271.9",
        "curTermRealisedPnlRv": "1271.9",
        "currency": "BTC",
        "deleveragePercentileRr": "0.1",
        "estimatedOrdLossRv": "1271.9",
        "execSeq": 0,
        "initMarginReqRr": "0.1",
        "lastFundingTimeNs": 450000000,
        "lastTermEndTimeNs": 450000000,
        "leverageRr": "0",
        "liquidationPriceRp": "1271.9",
        "maintMarginReqRr": "0.1",
        "makerFeeRateRr": "0.1",
        "markPriceRp": "1271.9",
        "posCostRv": "1271.9",
        "posMode": "Hedged",
        "posSide": "Long",
        "positionMarginRv": "1271.9",
        "positionStatus": "Normal",
        "riskLimitRv": "0.1",
        "sellValueToCostRr": "0.1",
        "side": "Sell",
        "size": "0",
        "symbol": "BTC",
        "takerFeeRateRr": "0.1",
        "term": 0,
        "transactTimeNs": 450000000,
        "usedBalanceRv": "1271.9",
        "userID": 12345,
        "valueRv": "1271.9"
      }
    ]
  },
  "msg": "string"
}
```



<a name="switchpositionmodesync"/>

#### Switch Position Mode Synchronously

* Request

```
PUT /g-positions/switch-pos-mode-sync?symbol=<symbol>&targetPosMode=<targetPosMode>
```

| Field         | Type   | Required | Description                    | Possible values |
| ------------- | ------ | -------- | ------------------------------ |-----------------|
| symbol        | String | Yes      | symbol to switch position mode |                 |
| targetPosMode | String | Yes      | the target position mode       | OneWay, Hedged  |

* Response

```json
{
  "code": 0,
  "data": "string",
  "msg": "string"
}
```



<a name="setleverage"/>

#### Set Leverage

* Request

```
PUT /g-positions/leverage?leverageRr=<leverage>&longLeverageRr=<longLeverageRr>&shortLeverageRr=<shortLeverageRr>&symbol=<symbol>
```

| Field           | Type   | Required | Description                                                  |
| --------------- | ------ | -------- | ------------------------------------------------------------ |
| symbol          | String | Yes      | symbol to set leverage                                       |
| leverageRr      | String | -        | new leverage value, if leverageRr exists, the position side is merged. <br />either leverageRr or longLeverageRr and shortLeverageRr should exist. |
| longLeverageRr  | String | -        | new long leverage value, if  longLeverageRr exists, the position side is hedged.<br />either leverageRr or longLeverageRr and shortLeverageRr should exist. |
| shortLeverageRr | String | -        | new short leverage value, if shortLeverageRr exists, the position side is hedged<br />either leverageRr or longLeverageRr and shortLeverageRr should exist. |

* Response

```json
{
  "code": 0,
  "data": "string",
  "msg": "string"
}
```



<a name="setrisklimit"/>

#### Set RiskLimit

* Request

```
PUT /g-positions/riskLimit?posSide=<posSide>&riskLimitRv=<riskLimitRv>&symbol=<symbol>
```

| Field       | Type   | Required | Description                        |
| ----------- | ------ | -------- | ---------------------------------- |
| symbol      | String | Yes      | symbol to be set risk limt         |
| riskLimitRv | String | Yes      | real value of risk limit to be set |
| posSide     | String | Yes      | position side to set risk limit    |

* Response

```json
{
  "code": 0,
  "data": "string",
  "msg": "string"
}
```



<a name="assignpositionbalance"/>

#### Assign Position Balance

* Request

```
POST /g-positions/assign?posBalanceRv=<posBalanceRv>&posSide=<posSide>&symbol=<symbol>
```

| Field        | Type   | Required | Description                              |
| ------------ | ------ | -------- | ---------------------------------------- |
| symbol       | String | Yes      | symbol to assign position balance        |
| posSide      | String | Yes      | position side to assign position balance |
| posBalanceRv | String | Yes      | the position balance value               |

* Response

```json
{
  "code": 0,
  "data": "string",
  "msg": "string"
}
```



<a name="queryopenorder"/>

#### Query open orders by symbol

* Order status includes `New`, `PartiallyFilled`, `Filled`, `Canceled`, `Rejected`, `Triggered`, `Untriggered`;
* Open order status includes `New`, `PartiallyFilled`, `Untriggered`;

* Request

```
GET /g-orders/activeList?symbol=<symbol>
```

| Field   | Type   | Description                                | Possible values |
|---------|--------|--------------------------------------------|--------------|
| symbol  | String | which symbol needs to query | [Trading symbols](#symbpricesub)  |


* Response
  * Full order

```
to be added
```

<a name="queryorder"/>

#### Query closed orders by symbol

* This API is for ***closed*** orders. For open orders, please use [open order query](#queryopenorder)

* Request


```
GET /exchange/order/v2/orderList?symbol=<symbol>&currency=<currency>&ordStatus=<ordStatus>&ordType=<ordType>&start=<start>&end=<end>&offset=<offset>&limit=<limit>&withCount=<withCount>
```

| Field     | Type    | Required | Description                                                         | Possible values                                                                                                                                                                                                      |
|----|----|----|---------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| symbol    | String  | No       | which symbol needs to query                                         | [Trading symbols](#symbpricesub)                                                                                                                                                                                     |
|currency|String|Yes| which currency needs to query                                       |                                                                                                                                                                                                                   |
| ordStatus | Integer | No       | order status code list filter                                       | New(5), PartiallyFilled(6), Untriggered(1), Filled(7), Canceled(8)                                                                                                                                                   |
| ordType   | Integer | No       | order type code list filter                                         | Market(1),Limit(2),Stop(3),StopLimit(4),MarketIfTouched(5),LimitIfTouched(6),ProtectedMarket(7),MarketAsLimit(8),StopAsLimit(9),MarketIfTouchedAsLimit(10),Bracket(11),BoTpLimit(12),BoSlLimit(13),BoSlMarket(14)    |
| start     | Integer | Yes      | start time range, Epoch millis，available only from the last 2 month ||
| end       | Integer | Yes      | end time range, Epoch millis                                        ||
| offset    | Integer | Yes      | offset to resultset                                                 ||
| limit     | Integer | Yes      | limit of resultset, max 200                                         ||
| withCount | boolean | No       | if true, result info will contains count info.                      | true,false                                                                                                                                                                                                           |

- Response
  - sample response

```
{
    "code": 0,
    "msg": "OK",
    "data": {
        "total": 3,
        "rows": [
            {
                "createdAt": 1666179379726,
                "symbol": "ETHUSDT",
                "orderQtyRq": "0.78",
                "side": 1,
                "priceRp": "1271.9",
                "execQtyRq": "0.78",
                "leavesQtyRq": "0",
                "execPriceRp": "1271.9",
                "orderValueRv": "992.082",
                "leavesValueRv": "0",
                "cumValueRv": "992.082",
                "stopDirection": 0,
                "stopPxRp": "0",
                "trigger": 0,
                "actionBy": 1,
                "execFeeRv": "0.0012719",
                "ordType": 2,
                "ordStatus": 7,
                "clOrdId": "2739dc9",
                "orderId": "2739dc90-41c6-449f-8774-0a62c8d8e320",
                "execStatus": 6,
                "bizError": 0,
                "totalPnlRv": null,
                "avgTransactPriceRp": null,
                "orderDetailsVos": null,
                "tradeType": 1
            },
            {
                "createdAt": 1666179345898,
                "symbol": "ETHUSDT",
                "orderQtyRq": "0.78",
                "side": 1,
                "priceRp": "1271.9",
                "execQtyRq": "0.78",
                "leavesQtyRq": "0",
                "execPriceRp": "1271.9",
                "orderValueRv": "992.082",
                "leavesValueRv": "0",
                "cumValueRv": "992.082",
                "stopDirection": 0,
                "stopPxRp": "0",
                "trigger": 0,
                "actionBy": 1,
                "execFeeRv": "0.0992082",
                "ordType": 2,
                "ordStatus": 7,
                "clOrdId": "ae20aef",
                "orderId": "ae20aef9-a62e-49d6-947e-39a9b2be39dd",
                "execStatus": 6,
                "bizError": 0,
                "totalPnlRv": null,
                "avgTransactPriceRp": null,
                "orderDetailsVos": null,
                "tradeType": 1
            },
            {
                "createdAt": 1666179331578,
                "symbol": "ETHUSDT",
                "orderQtyRq": "0.07",
                "side": 1,
                "priceRp": "1271.9",
                "execQtyRq": "0.07",
                "leavesQtyRq": "0",
                "execPriceRp": "1271.9",
                "orderValueRv": "89.033",
                "leavesValueRv": "0",
                "cumValueRv": "89.033",
                "stopDirection": 0,
                "stopPxRp": "0",
                "trigger": 0,
                "actionBy": 1,
                "execFeeRv": "0.0089033",
                "ordType": 2,
                "ordStatus": 7,
                "clOrdId": "74a6899",
                "orderId": "74a68994-f63e-4a6d-82b6-6f560f7e22a7",
                "execStatus": 6,
                "bizError": 0,
                "totalPnlRv": null,
                "avgTransactPriceRp": null,
                "orderDetailsVos": null,
                "tradeType": 1
            }
        ]
    }
}
```

<a name="queryorderbyid"/>

#### Query user order by orderID or Query user order by client order ID
* Request

to be added


<a name="querytrade"/>

#### Query user trade

* Request


```
GET /exchange/order/v2/tradingList?symbol=<symbol>&currency=<currency>&execType=<execType>&offset=<offset>&limit=<limit>&withCount=<withCount>
```



| Field     | Type    | Required | Description                                    | Possible values                                                    |
|-----------|---------|----------|------------------------------------------------|--------------------------------------------------------------------|
| symbol    | String  | No       | which symbol needs to query                    | [Trading symbols](#symbpricesub)                                   |
| currency  | String  | Yes      | which currency needs to query                  |                                                                    |
| execType  | Integer | No       | order status code list filter                  | New(5), PartiallyFilled(6), Untriggered(1), Filled(7), Canceled(8) |
| offset    | Integer | Yes      | offset to resultset                            |                                                                    |
| limit     | Integer | Yes      | limit of resultset, max 200                    |                                                                    |
| withCount | boolean | No       | if true, result info will contains count info. | true,false                                                         |

- Response
  - sample response

```
{
    "code": 0,
    "msg": "OK",
    "data": {
        "total": 4,
        "rows": [
            {
                "createdAt": 1666226932259,
                "symbol": "ETHUSDT",
                "currency": "USDT",
                "action": 1,
                "tradeType": 1,
                "execQtyRq": "0.01",
                "execPriceRp": "1271.9",
                "side": 1,
                "orderQtyRq": "0.78",
                "priceRp": "1271.9",
                "execValueRv": "12.719",
                "feeRateRr": "0.0001",
                "execFeeRv": "0.0012719",
                "ordType": 2,
                "execId": "8718cae",
                "execStatus": 6
            },
            {
                "createdAt": 1666226903754,
                "symbol": "ETHUSDT",
                "currency": "USDT",
                "action": 1,
                "tradeType": 1,
                "execQtyRq": "0.77",
                "execPriceRp": "1271.9",
                "side": 1,
                "orderQtyRq": "0.78",
                "priceRp": "1271.9",
                "execValueRv": "979.363",
                "feeRateRr": "0.0001",
                "execFeeRv": "0.0979363",
                "ordType": 2,
                "execId": "c6db276",
                "execStatus": 6
            },
            {
                "createdAt": 1666226903754,
                "symbol": "ETHUSDT",
                "currency": "USDT",
                "action": 1,
                "tradeType": 1,
                "execQtyRq": "0.78",
                "execPriceRp": "1271.9",
                "side": 1,
                "orderQtyRq": "0.78",
                "priceRp": "1271.9",
                "execValueRv": "992.082",
                "feeRateRr": "0.0001",
                "execFeeRv": "0.0992082",
                "ordType": 2,
                "execId": "378d083",
                "execStatus": 6
            },
            {
                "createdAt": 1666226903754,
                "symbol": "ETHUSDT",
                "currency": "USDT",
                "action": 1,
                "tradeType": 1,
                "execQtyRq": "0.07",
                "execPriceRp": "1271.9",
                "side": 1,
                "orderQtyRq": "0.07",
                "priceRp": "1271.9",
                "execValueRv": "89.033",
                "feeRateRr": "0.0001",
                "execFeeRv": "0.0089033",
                "ordType": 2,
                "execId": "8b8a8a0",
                "execStatus": 6
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

- Request：

```

  GET /md/v2/orderbook?symbol=<symbol>

```

| Field   | Type    | Description           | Possible values                   |
|---------|---------|-----------------------|-----------------------------------|
| symbol  | String  | Contract symbol name  | [Trading symbols](#symbpricesub)  |

- Response:

```
  {
    "error": null,
    "id": 0,
    "result": {
    "orderbook_p": {
      "asks": [
        [
          <priceRp>,
          <size>
        ],
        ...
        ...
        ...
      ],
      "bids": [
        [
          <priceRp>,
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

| Field      | Type     | Description                 | Possible values                  |
|------------|----------|-----------------------------|----------------------------------|
| timestamp  | Integer  | Timestamp in nanoseconds    |                                  |
| priceRp    | String   | Real book level price       |                                  |
| sizeRq     | String   | Real book level size        |                                  |
| sequence   | Integer  | current message sequence    ||
| symbol     | String   | Contract symbol name        | [Trading symbols](#symbpricesub) |

- Sample：

```
  GET /md/v2/orderbook?symbol=BTCUSDT
```

```
  {
      "error": null,
      "id": 0,
      "result": {
          "depth": 30,
          "orderbook_p": {
              "asks": [
                  [
                      "20675.7",
                      "0.736"
                  ],
                  [
                      "20676.2",
                      "0.613"
                  ],
                  ...
                  ...
                  ...
              ],
              "bids": [
                  [
                      "20672.4",
                      "0.818"
                  ],
                  [
                      "20672.2",
                      "0.614"
                  ],
                  ...
                  ...
                  ...
              ]
          },
          "sequence": 77770771,
          "symbol": "BTCUSDT",
          "timestamp": 1666860123727907896,
          "type": "snapshot"
      }
  }

```

<a name="querykline"/>

### Query kline

NOTE:

1. please be noted that kline interfaces have [rate limits](https://github.com/phemex/phemex-api-docs/blob/master/Generic-API-Info.en.md#rate-limits) rule,  please check the Other group under [api groups](https://github.com/phemex/phemex-api-docs/blob/master/Generic-API-Info.en.md#api-groups)


```
  GET /exchange/public/md/v2/kline/last?symbol=<symbol>&resolution=<resolution>&limit=<limit>
```

| Field      | Type    | Required | Description                 | Possible values                                                                                                                                                       |
|------------|---------|----------|-----------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| symbol     | String  | Yes      | which symbol needs to query | [Trading symbols](#symbpricesub)                                                                                                                                      |
| resolution | Integer | Yes      | Kline Interval              | MINUTE_1(60),MINUTE_5(300),MINUTE_15(900),MINUTE_30(1800),HOUR_1(3600),HOUR_4(14400),DAY_1(86400),WEEK_1(604800),MONTH_1(2592000),SEASON_1(7776000),YEAR_1(31104000)  |
| limit      | Integer | No       | records limit               | 5                                                                                                                                                                     |


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


* Value of `resolution`s

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

* Value of `limit`s

| limit    | Description |
|----------|-------------|
| 5        | limit 5     |
| 10       | limit 10    |
| 50      | limit 50    |
| 100     | limit 100   |
| 500     | limit 500   |
| 1000    | limit 1000  |


**NOTE**, for backward compatibility reason, phemex also provides kline query with from/to, however, this interface is **NOT** recommended.

```
GET /exchange/public/md/v2/kline/list?symbol=<symbol>&to=<to>&from=<from>&resolution=<resolution>
```


| Field       | Type    | Required    | Description            | Possible Values                                                                                                |
|-------------|---------|-------------|------------------------|----------------------------------------------------------------------------------------------------------------|
|symbol       | String  | Yes         | symbol name            | BTCUSD,ETHUSD,uBTCUSD,cETHUSD,XRPUSD...                                                                        | 
| from        | Integer | Yes         | start time in seconds  | value aligned in resolution boundary                                                                           |
| to          | Integer | Yes         | end time in seconds    | value aligned in resolution boundary; Number of k-lines return between [`from`, `to`) should be less than 1000 | 
| resolution  | Integer | Yes         | kline interval         | the same as described above                                                                                    |




<a name="querytrade"/>

#### Query Trade

- Request：
  ```
  GET /md/v2/trade?symbol=<symbol>
  ```

| Field   | Type    | Description           | Possible values                   |
|---------|---------|-----------------------|-----------------------------------|
| symbol  | String  | Contract symbol name  | [Trading symbols](#symbpricesub)  |

- Response:
  ```
  {
    "error": null,
    "id": 0,
    "result": {
    "sequence": <sequence>,
    "symbol": <symbol>,
    "trades_p": [
                    [
                        <timestamp>,
                        <Side>,
                        <PriceRp>,
                        <SizeRq>
                    ],
                    [
                        <timestamp>,
                        <Side>,
                        <PriceRp>,
                        <SizeRq>
                    ],
                    ...
                    ...
                    ...
                ],
    "type": "snapshot"
    }
  }
  ```

| Field      | Type    |Description| Possible values                   |
|------------|---------|----|-----------------------------------|
| timestamp  | Integer |Timestamp in nanoseconds||
| Side       | String  |Trade Side, Buy or Sell||
| priceRp    | String  |Real trade price||
| sizeRq     | String  |Real trade size||
| sequence   | Integer |current message sequence||
| symbol     | String  |Contract symbol name| [Trading symbols](#symbpricesub)  |

- Sample：

```
  GET /md/v2/trade?symbol=BTCUSDT
  
  {
    "error": null,
    "id": 0,
    "result": {
        "sequence": 77766796,
        "symbol": "BTCUSDT",
        "trades_p": [
            [
                1666860389799499000,
                "Buy",
                "20702.1",
                "0.949"
            ],
            [
                1666860377636154600,
                "Sell",
                "20675.5",
                "0.785"
            ],
            ...
            ...
            ...
        ],
        "type": "snapshot"
    }
  }
```

<a name="query24hrsticker"/>


```
  GET /md/v2/ticker/24hr?symbol=<symbol>
```

|Field|Type|Required|Description| Possible values                   |
|----|----|----|----|-----------------------------------|
|symbol|String|Yes|which symbol needs to query| [Trading symbols](#symbpricesub)  |

```
  GET /md/v2/ticker/24hr?symbol=BTCUSDT
```

- Response
  - sample response

```
{
    "error": null,
    "id": 0,
    "result": {
        "closeRp": "20731",
        "fundingRateRr": "0.0001",
        "highRp": "20818.8",
        "indexPriceRp": "20737.09857143",
        "lowRp": "20425.2",
        "markPriceRp": "20737.788944",
        "openInterestRv": "0",
        "openRp": "20709",
        "predFundingRateRr": "0.0001",
        "symbol": "BTCUSDT",
        "timestamp": 1667222412794076700,
        "turnoverRv": "139029311.7517",
        "volumeRq": "6747.727"
    }
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

### Subscribe OrderBook for new Model

On each successful subscription, DataGW will immediately send the current Order Book snapshot to client and all later order book updates will be published.

* Request
```
{
  "id": <id>,
  "method": "orderbook_p.subscribe",
  "params": [
    "<new_symbol>"
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
  "method": "orderbook_p.subscribe",
  "params": [
    "BTCUSDT"
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
}

```

| Field       | Type    | Description             | Possible values |
|-------------|---------|-------------------------|-----------------|
| side        | String  | Price level side        | bid, ask        |
| priceEp     | String  | Raw price               |                 |
| qty         | String  | Price level size        |                 |
| sequence    | Integer | Latest message sequence |          |
| depth       | Integer | Market depth            | 30              |
| type        | String  | Message type            | snapshot, incremental |


* Sample：

```
< {"depth":30,"orderbook_p":{"asks":[["20702.9","0.718"],["20703.9","0.524"],["20704.9","0"],["20720.8","0"]],"bids":[["20703.1","0"],["20701.3","0"],["20701.2","0"],["20700.5","1.622"],["20473.7","1.074"],["20441.3","0.904"]]},"sequence":77668172,"symbol":"BTCUSDT","timestamp":1666854171201355264,"type":"incremental"}
< {"depth":30,"orderbook_p":{"asks":[],"bids":[["20700.5","0"],["20340.5","0.06"]]},"sequence":77668209,"symbol":"BTCUSDT","timestamp":1666854173705089711,"type":"incremental"}
```

<a name="bookunsub"/>

### Unsubscribe OrderBook

It unsubscribes all orderbook related subscriptions.

* Request

```
{
  "id": <id>,
  "method": "orderbook_p.unsubscribe",
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
  "method": "trade_p.subscribe",
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
  "method": "trade_p.subscribe",
  "params": [
    "BTCUSDT"
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
      "<price>",
      "<qty>"
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

| Field     | Type   | Description                             | Possible values |
|-----------|--------|-----------------------------------------|-----------------|
| timestamp | Integer| Timestamp in nanoseconds for each trade ||
| side      | String | Execution taker side                    | bid, ask        |
| price     | String| Raw price                               |                 |
| qty       | String| Execution size                          |                 |
| sequence  | Integer| Latest message sequence                 ||
| symbol    | String | Contract symbol name                    ||
| type      | String | Message type                            |snapshot, incremental |


* Sample
```
< {
    "sequence": 77702250,
    "symbol": "BTCUSDT",
    "trades_p": [
        [
            1666856076819029800,
            "Sell",
            "20700.3",
            "0.649"
        ]
    ],
    "type": "incremental"
}

< {
    "sequence": 77663551,
    "symbol": "BTCUSDT",
    "trades_p": [
        [
            1666856062351916300,
            "Sell",
            "20703.6",
            "0.669"
        ],
        [
            1666854025545354000,
            "Buy",
            "20699",
            "0.001"
        ]
    ],
    "type": "snapshot"
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
  "method": "trade_p.unsubscribe",
  "params": [
  ]
}

# unsubscribe all trade subsciptions for a symbol
{
  "id": <id>,
  "method": "trade_p.unsubscribe",
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
  "method": "kline_p.subscribe",
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
  "method": "kline_p.subscribe",
  "params": [
    "BTCUSDT",
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
      <lastClose>,
      <open>,
      <high>,
      <low>,
      <close>,
      <volume>,
      <turnover>,
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

| Field      | Type    | Description                                    | Possible values |
|------------|---------|------------------------------------------------|-----------------|
| timestamp  | Integer | Timestamp in nanoseconds for each trade        ||
| interval   | Integer | Kline interval type                            | 60, 300, 900, 1800, 3600, 14400, 86400, 604800, 2592000, 7776000, 31104000 |
| lastClose  | String  | Unscaled last close price                      |                 |
| open       | String  | Unscaled open price                              |                 |
| high       | String  | Unscaled high price                              |                 |
| low        | String  | Unscaled low price                               |                 |
| close      | String  | Unscaled close price                             |                 |
| volume     | String  | Trade voulme during the current kline interval ||
| turnover   | String  | Unscaled turnover value                          |                 |
| sequence   | Integer | Latest message sequence                        ||
| symbol     | String  | Contract symbol name                           ||
| type       | String  | Message type                                   |snapshot, incremental |


* Sample
```
< {
    "kline_p": [
        [
            1666856340,
            60,
            "20689.5",
            "20686.2",
            "20695.4",
            "20686.2",
            "20691.6",
            "2.742",
            "56731.5609"
        ],
        [
            1666856280,
            60,
            "20700.1",
            "20712.7",
            "20712.7",
            "20689.1",
            "20689.5",
            "4.407",
            "91208.3065"
        ]        
    ],
    "sequence": 77711279,
    "symbol": "BTCUSDT",
    "type": "snapshot"
}

< {
    "kline_p": [
        [
            1666856520,
            60,
            "20685",
            "20684.8",
            "20684.8",
            "20675.2",
            "20675.2",
            "3.547",
            "73353.8417"
        ]
    ],
    "priceScale": 0,
    "sequence": 77715046,
    "symbol": "BTCUSDT",
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
  "method": "kline_p.unsubscribe",
  "params": []
}

# unsubscribe all Kline subscriptions of a symbol
{
  "id": <id>,
  "method": "kline_p.unsubscribe",
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
  "method": "aop_p.subscribe",
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
  "method": "aop_p.subscribe",
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
{"accounts_p":[{"accountBalanceRv":"1508.452588802237","accountID":9328670003,"bonusBalanceRv":"0","currency":"USDT","totalUsedBalanceRv":"343.132599666883","userID":932867}],"orders_p":[{"accountID":9328670003,"action":"New","actionBy":"ByUser","actionTimeNs":1666858780876924611,"addedSeq":77751555,"apRp":"0","bonusChangedAmountRv":"0","bpRp":"0","clOrdID":"c0327a7d-9064-62a9-28f6-2db9aaaa04e0","closedPnlRv":"0","closedSize":"0","code":0,"cumFeeRv":"0","cumQty":"0","cumValueRv":"0","curAccBalanceRv":"1508.489893982237","curAssignedPosBalanceRv":"24.62786650928","curBonusBalanceRv":"0","curLeverageRr":"-10","curPosSide":"Buy","curPosSize":"0.043","curPosTerm":1,"curPosValueRv":"894.0689","curRiskLimitRv":"1000000","currency":"USDT","cxlRejReason":0,"displayQty":"0.003","execFeeRv":"0","execID":"00000000-0000-0000-0000-000000000000","execPriceRp":"20723.7","execQty":"0","execSeq":77751555,"execStatus":"New","execValueRv":"0","feeRateRr":"0","leavesQty":"0.003","leavesValueRv":"63.4503","message":"No error","ordStatus":"New","ordType":"Market","orderID":"fa64c6f2-47a4-4929-aab4-b7fa9bbc4323","orderQty":"0.003","pegOffsetValueRp":"0","posSide":"Long","priceRp":"21150.1","relatedPosTerm":1,"relatedReqNum":11,"side":"Buy","slTrigger":"ByMarkPrice","stopLossRp":"0","stopPxRp":"0","symbol":"BTCUSDT","takeProfitRp":"0","timeInForce":"ImmediateOrCancel","tpTrigger":"ByLastPrice","tradeType":"Amend","transactTimeNs":1666858780881545305,"userID":932867},{"accountID":9328670003,"action":"New","actionBy":"ByUser","actionTimeNs":1666858780876924611,"addedSeq":77751555,"apRp":"0","bonusChangedAmountRv":"0","bpRp":"0","clOrdID":"c0327a7d-9064-62a9-28f6-2db9aaaa04e0","closedPnlRv":"0","closedSize":"0","code":0,"cumFeeRv":"0","cumQty":"0.003","cumValueRv":"62.1753","curAccBalanceRv":"1508.452588802237","curAssignedPosBalanceRv":"24.62786650928","curBonusBalanceRv":"0","curLeverageRr":"-10","curPosSide":"Buy","curPosSize":"0.046","curPosTerm":1,"curPosValueRv":"956.2442","curRiskLimitRv":"1000000","currency":"USDT","cxlRejReason":0,"displayQty":"0.003","execFeeRv":"0.03730518","execID":"b6c8d16b-c777-510c-8476-80f399b2d5ad","execPriceRp":"20725.1","execQty":"0.003","execSeq":77751555,"execStatus":"TakerFill","execValueRv":"62.1753","feeRateRr":"0.0006","lastLiquidityInd":"RemovedLiquidity","leavesQty":"0","leavesValueRv":"0","message":"No error","ordStatus":"Filled","ordType":"Market","orderID":"fa64c6f2-47a4-4929-aab4-b7fa9bbc4323","orderQty":"0.003","pegOffsetValueRp":"0","posSide":"Long","priceRp":"21150.1","relatedPosTerm":1,"relatedReqNum":11,"side":"Buy","slTrigger":"ByMarkPrice","stopLossRp":"0","stopPxRp":"0","symbol":"BTCUSDT","takeProfitRp":"0","timeInForce":"ImmediateOrCancel","tpTrigger":"ByLastPrice","tradeType":"Trade","transactTimeNs":1666858780881545305,"userID":932867}],"positions_p":[{"accountID":9328670003,"assignedPosBalanceRv":"30.861734862748","avgEntryPriceRp":"20787.917391304","bankruptCommRv":"0.0000006","bankruptPriceRp":"0.1","buyLeavesQty":"0","buyLeavesValueRv":"0","buyValueToCostRr":"0.10114","createdAtNs":0,"crossSharedBalanceRv":"1165.319989135354","cumClosedPnlRv":"0","cumFundingFeeRv":"0.089061821453","cumTransactFeeRv":"0.57374652","curTermRealisedPnlRv":"-0.662808341453","currency":"USDT","dataVer":11,"deleveragePercentileRr":"0","displayLeverageRr":"0.79941382","estimatedOrdLossRv":"0","execSeq":77751555,"freeCostRv":"0","freeQty":"-0.046","initMarginReqRr":"0.1","lastFundingTime":1666857600000000000,"lastTermEndTime":0,"leverageRr":"-10","liquidationPriceRp":"0.1","maintMarginReqRr":"0.01","makerFeeRateRr":"-1","markPriceRp":"20735.47347096","minPosCostRv":"0","orderCostRv":"0","posCostRv":"30.284669572349","posMode":"Hedged","posSide":"Long","positionMarginRv":"1196.181723398102","positionStatus":"Normal","riskLimitRv":"1000000","sellLeavesQty":"0","sellLeavesValueRv":"0","sellValueToCostRr":"0.10126","side":"Buy","size":"0.046","symbol":"BTCUSDT","takerFeeRateRr":"-1","term":1,"transactTimeNs":1666858780881545305,"unrealisedPnlRv":"-2.41242033584","updatedAtNs":0,"usedBalanceRv":"30.861734862748","userID":932867,"valueRv":"956.2442"},{"accountID":9328670003,"assignedPosBalanceRv":"9.473634984","avgEntryPriceRp":"20786.455555556","bankruptCommRv":"1.153171711445","bankruptPriceRp":"1000000","buyLeavesQty":"0","buyLeavesValueRv":"0","buyValueToCostRr":"0.10114","createdAtNs":0,"crossSharedBalanceRv":"1165.319989135354","cumClosedPnlRv":"0","cumFundingFeeRv":"-0.074563385402","cumTransactFeeRv":"0.44898744","curTermRealisedPnlRv":"-0.374424054598","currency":"USDT","dataVer":10,"deleveragePercentileRr":"0","displayLeverageRr":"0.63759936","estimatedOrdLossRv":"0","execSeq":77731059,"freeCostRv":"0","freeQty":"0.036","initMarginReqRr":"0.1","lastFundingTime":1666857600000000000,"lastTermEndTime":0,"leverageRr":"-10","liquidationPriceRp":"1000000","maintMarginReqRr":"0.01","makerFeeRateRr":"-1","markPriceRp":"20735.47347096","minPosCostRv":"0","orderCostRv":"0","posCostRv":"9.473634984","posMode":"Hedged","posSide":"Short","positionMarginRv":"1173.640452407909","positionStatus":"Normal","riskLimitRv":"1000000","sellLeavesQty":"0","sellLeavesValueRv":"0","sellValueToCostRr":"0.10126","side":"Sell","size":"0.036","symbol":"BTCUSDT","takerFeeRateRr":"-1","term":1,"transactTimeNs":1666858780876924611,"unrealisedPnlRv":"1.83535504544","updatedAtNs":0,"usedBalanceRv":"9.473634984","userID":932867,"valueRv":"748.3124"},{"accountID":9328670003,"assignedPosBalanceRv":"156.56916092763","avgEntryPriceRp":"1563.815","bankruptCommRv":"0.187657798123","bankruptPriceRp":"1042.55","buyLeavesQty":"0","buyLeavesValueRv":"0","buyValueToCostRr":"0.33433334","createdAtNs":0,"crossSharedBalanceRv":"1165.319989135354","cumClosedPnlRv":"-89.82","cumFundingFeeRv":"0.104061681397","cumTransactFeeRv":"0.4835887","curTermRealisedPnlRv":"-0.328183513333","currency":"USDT","dataVer":14,"deleveragePercentileRr":"0","displayLeverageRr":"2.99999994","estimatedOrdLossRv":"0","execSeq":77731060,"freeCostRv":"0","freeQty":"-0.3","initMarginReqRr":"0.33333333","lastFundingTime":1666857600000000000,"lastTermEndTime":1666755074905395643,"leverageRr":"3","liquidationPriceRp":"1058.2","maintMarginReqRr":"0.01","makerFeeRateRr":"-1","markPriceRp":"1558.51230846","minPosCostRv":"0","orderCostRv":"0","posCostRv":"156.56916092763","posMode":"Hedged","posSide":"Long","positionMarginRv":"156.381503129507","positionStatus":"Normal","riskLimitRv":"500000","sellLeavesQty":"0","sellLeavesValueRv":"0","sellValueToCostRr":"0.33473333","side":"Buy","size":"0.3","symbol":"ETHUSDT","takerFeeRateRr":"-1","term":2,"transactTimeNs":1666858780876924611,"unrealisedPnlRv":"-1.590807462","updatedAtNs":0,"usedBalanceRv":"156.56916092763","userID":932867,"valueRv":"469.1445"},{"accountID":9328670003,"assignedPosBalanceRv":"146.228068892505","avgEntryPriceRp":"1455.495","bankruptCommRv":"0.349516231597","bankruptPriceRp":"1941.75","buyLeavesQty":"0","buyLeavesValueRv":"0","buyValueToCostRr":"0.33433334","createdAtNs":0,"crossSharedBalanceRv":"1165.319989135354","cumClosedPnlRv":"0","cumFundingFeeRv":"-0.159460679685","cumTransactFeeRv":"0.2619891","curTermRealisedPnlRv":"-0.102528420315","currency":"USDT","dataVer":14,"deleveragePercentileRr":"0","displayLeverageRr":"2.99323302","estimatedOrdLossRv":"0","execSeq":77731060,"freeCostRv":"0","freeQty":"0.3","initMarginReqRr":"0.33333333","lastFundingTime":1666857600000000000,"lastTermEndTime":0,"leverageRr":"3","liquidationPriceRp":"1927.21","maintMarginReqRr":"0.01","makerFeeRateRr":"-1","markPriceRp":"1558.51230846","minPosCostRv":"0","orderCostRv":"0","posCostRv":"145.898817344505","posMode":"Hedged","posSide":"Short","positionMarginRv":"145.878552660908","positionStatus":"Normal","riskLimitRv":"500000","sellLeavesQty":"0","sellLeavesValueRv":"0","sellValueToCostRr":"0.33473333","side":"Sell","size":"0.3","symbol":"ETHUSDT","takerFeeRateRr":"-1","term":1,"transactTimeNs":1666858780876924611,"unrealisedPnlRv":"-30.905192538","updatedAtNs":0,"usedBalanceRv":"146.228068892505","userID":932867,"valueRv":"436.6485"}],"sequence":68744,"timestamp":1666858780883525030,"type":"incremental","version":0}

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
  "method": "aop_p.unsubscribe",
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
  "method": "perp_market24h_pack_p.subscribe",
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
  "method": "perp_market24h_pack_p.subscribe",
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
    "data": [
        [
            <symbol>,
            <openRp>,
            <highRp>,
            <lowRp>,
            <lastRp>,
            <volumeRq>,
            <turnoverRv>,
            <openInterestRv>,
            <indexRp>,
            <markRp>,
            <fundingRateRr>,
            <predFundingRateRr>
        ]
    ],
    "fields": [
        "symbol",
        "openRp",
        "highRp",
        "lowRp",
        "lastRp",
        "volumeRq",
        "turnoverRv",
        "openInterestRv",
        "indexRp",
        "markRp",
        "fundingRateRr",
        "predFundingRateRr"
    ],
    "method": "perp_market24h_pack_p.update",
    "timestamp": 1666862556850547000,
    "type": "snapshot"
}
```

| Field             | Type    | Description                                  | Possible values                  |
|-------------------|---------|----------------------------------------------|----------------------------------|
| symbol            | String  | Contract symbol name                         | [Trading symbols](#symbpricesub) |
| openRp            | String  | The unscaled open price in last 24 hours     |                                  |
| highRp            | String  | The unscaled highest price in last 24 hours  |                                  |
| lowRp             | String  | The unscaled lowest price in last 24 hours   |                                  |
| lastRp            | String  | The unscaled close price in last 24 hours    |                                  |
| volumeRq          | String  | Symbol trade volume in last 24 hours         |                                  |
| turnoverRv        | String  | The unscaled turnover value in last 24 hours |                                  |
| openInterestRv    | String  | current open interest                        |                                  |
| indexRp           | String  | Unscaled index price                         |                                  |
| markRp            | String  | Unscaled mark price                          |                                  |
| fundingRateRr     | String  | Unscaled funding rate                        |                                  |
| predFundingRateRr | String  | Unscaled predicated funding rate             |                                  |



* Sample:

```
< {
    "data": [
        [
            "ETHUSDT",
            "1533.72",
            "1594.17",
            "1510.05",
            "1547.52",
            "545942.34",
            "848127644.5712",
            "0",
            "1548.31694379",
            "1548.44513153",
            "0.0001",
            "0.0001"
        ],
        [
            "BTCUSDT",
            "20614.5",
            "21628.4",
            "19258.6",
            "20626.3",
            "8819.819",
            "182892627.4297",
            "0",
            "20641.8167574",
            "20643.52572781",
            "0.0001",
            "0.0001"
        ]
    ],
    "fields": [
        "symbol",
        "openRp",
        "highRp",
        "lowRp",
        "lastRp",
        "volumeRq",
        "turnoverRv",
        "openInterestRv",
        "indexRp",
        "markRp",
        "fundingRateRr",
        "predFundingRateRr"
    ],
    "method": "perp_market24h_pack_p.update",
    "timestamp": 1666862556850547000,
    "type": "snapshot"
}
```

<a name="symbpricesub"/>

### Subscribe tick event for symbol price

* Every trading symbol has a suite of other symbols, each symbol follows same patterns,
  i.e. `index` symbol follows a pattern `.<BASECURRENCY><QUOTECURRENCY>`,
  `mark` symbol follows a pattern `.M<BASECURRENCY><QUOTECURRENCY>`,
  predicated funding rate's symbol follows a pattern `.<BASECURRENCY><QUOTECURRENCY>FR`,
  while funding rate symbol follows a pattern `.<BASECURRENCY><QUOTECURRENCY>FR8H`
* Price is retrieved by subscribing symbol tick.
* all available symbols (pfr=predicated funding rate)

| symbol  | index symbol  | mark symbol       | pfr symbol   | funding rate symbol |
|---------|---------------|-------------------|---------------|--------------|
| BTCUSDT | .BTCUSDT      | .MBTCUSDT         | .BTCUSDTFR       | .BTCUSDTFR8H            |
| ETHUSDT | .ETHUSDT      | .METHUSDT         | .ETHUSDTFR       | .ETHUSDTFR8H            |
| XRPUSDT | .XRPUSDT      | .MXRPUSDT         | .XRPUSDTFR       | .XRPUSDTFR8H            |
| ADAUSDT | .ADAUSDT      | .MADAUSDT         | .ADAUSDTFR       | .ADAUSDTFR8H            |


* Request

  * The symbol in params can be replace by any symbol.

```
{
    "method": "tick_p.subscribe",
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
        "last": <price>,
        "symbol": <symbol>
        "timestamp": <timestamp_nano>
    }
}

```


* Sample

```
{"tick_p":{"last":"20639.38692364","symbol":".BTCUSDT","timestamp":1666863393552000000}}
{"tick_p":{"last":"20639.15408363","symbol":".BTCUSDT","timestamp":1666863394538132741}}
```

