## Table of Contents * [Phemex Public API](#publicapi) * [General Public API Information](#general)
* [REST API Standards](#restapi)
  * [REST API List](#restapilist)
    * [Market API List](#marketapilist)
      * [Query Product Information](#queryproductinfo)
    * [Trade API List](#orderapilist)
      * [Place Order With Put Method, *Prefered*](#placeorderwithput)
      * [Place Order](#placeorder)
      * [Amend order by orderID](#amendorder)
      * [Cancel Single Order by orderID or clOrdID](#cancelsingleorder)
      * [Bulk Cancel Orders](#cancelorder)
      * [Cancel All Orders](#cancelall)
      * [Query Account Positions](#queryaccountpositions)
      * [Query Account Positions with unrealized PNL](#queryaccountpositionsWithPnl)
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
      * [Query Funding Fees History](#futureDataFundingFeesHist)
      * [Query Orders History](#futureDataOrdersHist)
      * [Query Orders By Ids](#futureDataOrdersByIds)
      * [Query Trades History](#futureDataTradesHist)
      * [Query Trading Fees History](#futureDataTradingFeesHist)
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
    * [Subscribe account margin (RAS)](#rassub)
    * [Unsubscribe account margin (RAS)](#rasunsub)
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



<a name="marketapilist"/>

### Market API List

<a name="queryproductinfo"/>

#### Query Product Information

* Request：

```
GET /public/products
```

you can find products info with hedged mode under node 'perpProductsV2'


<a name="orderapilist"/>

### Trade API List


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


#### Query Account Positions with unrealized PNL

<a name="queryaccountpositionsWithPnl"/>

Below API presents unrealized pnl at `markprice` of positions with **considerable** cost, thus
its [ratelimit](/Generic-API-Info.en.md#contractAPIGroup) weight is very high.

* Request

```
GET /g-accounts/positions?currency=<currency>
```

* Response

```json
{
  "code": 0,
  "msg": "",
  "data": {
    "account": {
      "userID": 4200013,
      "accountId": 42000130003,
      "currency": "USDT",
      "accountBalanceRv": "730.97309163",
      "totalUsedBalanceRv": "1.02037554",
      "bonusBalanceRv": "0"
    },
    "positions": [
      {
        "accountID": 42000130003,
        "symbol": "XEMUSDT",
        "currency": "USDT",
        "side": "Buy",
        "positionStatus": "Normal",
        "crossMargin": false,
        "leverageRr": "-10",
        "initMarginReqRr": "0.1",
        "maintMarginReqRr": "0.01",
        "riskLimitRv": "200000",
        "sizeRq": "186",
        "valueRv": "9.951",
        "avgEntryPriceRp": "0.0535",
        "posCostRv": "1.00047354",
        "assignedPosBalanceRv": "1.086606978",
        "bankruptCommRv": "0.00001116",
        "bankruptPriceRp": "0.0001",
        "positionMarginRv": "730.97308047",
        "liquidationPriceRp": "0.0001",
        "deleveragePercentileRr": "0",
        "buyValueToCostRr": "0.10114",
        "sellValueToCostRr": "0.10126",
        "markPriceRp": "0.053036917",
        "markValueEv": 0,
        "unRealisedPosLossEv": 0,
        "estimatedOrdLossRv": "0",
        "usedBalanceRv": "1.086606978",
        "takeProfitEp": 0,
        "stopLossEp": 0,
        "cumClosedPnlRv": "0",
        "cumFundingFeeRv": "0",
        "cumTransactFeeRv": "0.0059706",
        "realisedPnlEv": 0,
        "unRealisedPnlRv": "-0.086133438",
        "cumRealisedPnlEv": 0,
        "term": 1,
        "lastTermEndTimeNs": 0,
        "lastFundingTimeNs": 0,
        "curTermRealisedPnlRv": "-0.0059706",
        "execSeq": 2260172450,
        "posSide": "Long",
        "posMode": "Hedged"
      },
      {
        "accountID": 42000130003,
        "symbol": "XEMUSDT",
        "currency": "USDT",
        "side": "None",
        "positionStatus": "Normal",
        "crossMargin": false,
        "leverageRr": "-10",
        "initMarginReqRr": "0.1",
        "maintMarginReqRr": "0.01",
        "riskLimitRv": "200000",
        "sizeRq": "0",
        "valueRv": "0",
        "avgEntryPriceRp": "0",
        "posCostRv": "0",
        "assignedPosBalanceRv": "0",
        "bankruptCommRv": "0",
        "bankruptPriceRp": "0",
        "positionMarginRv": "0",
        "liquidationPriceRp": "0",
        "deleveragePercentileRr": "0",
        "buyValueToCostRr": "0.10114",
        "sellValueToCostRr": "0.10126",
        "markPriceRp": "0.053036917",
        "markValueEv": 0,
        "unRealisedPosLossEv": 0,
        "estimatedOrdLossRv": "0",
        "usedBalanceRv": "0",
        "takeProfitEp": 0,
        "stopLossEp": 0,
        "cumClosedPnlRv": "0",
        "cumFundingFeeRv": "0",
        "cumTransactFeeRv": "0",
        "realisedPnlEv": 0,
        "unRealisedPnlRv": "0",
        "cumRealisedPnlEv": 0,
        "term": 1,
        "lastTermEndTimeNs": 0,
        "lastFundingTimeNs": 0,
        "curTermRealisedPnlRv": "0",
        "execSeq": 0,
        "posSide": "Short",
        "posMode": "Hedged"
      }
    ]
  }
}

```

<b>Note</b> Highly recommend calculating `unRealizedPnlEv` in client side with latest `markPrice` to avoid ratelimit
penalty.


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
Risk Limit Modification Notice:
For Hedged contracts, the Set Position Risk Limit API has been deprecated. It is no longer possible to manually set the Risk Limit. Instead, simply adjust the leverage multiplier as required, and the Risk Limit will be 
automatically adjusted.

New Risk Limit Configuration Explanation:
A new attribute called leverageMargin has been added to the properties of symbol pairs. By locating the corresponding index_id within the leverageMargins node, one can find the associated Risk Limit information. The 
configuration is as follows:
```json
{
  "index_id": 1,
  "items": [
    {
      "notionalValueRv": 50000,
      "maxLeverage": 20,
      "maintenanceMarginRateRr": "0.01",
      "maintenanceAmountRv": "0"
    },
    {
      "notionalValueRv": 100000,
      "maxLeverage": 10,
      "maintenanceMarginRateRr": "0.06",
      "maintenanceAmountRv": "0"
    },
    {
      "notionalValueRv": 200000,
      "maxLeverage": 5,
      "maintenanceMarginRateRr": "0.16",
      "maintenanceAmountRv": "0"
    },
    {
      "notionalValueRv": 300000,
      "maxLeverage": 2,
      "maintenanceMarginRateRr": "0.46",
      "maintenanceAmountRv": "0"
    },
    {
      "notionalValueRv": 500000,
      "maxLeverage": 1,
      "maintenanceMarginRateRr": "0.5",
      "maintenanceAmountRv": "0"
    }
  ]
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

### Future Data API List

#### Query Funding Fees History

* Http Request

```
GET /api-data/g-futures/funding-fees?symbol=<symbol>
```

| Field    | Type           | Required | Description               | Possible Values         |
|----------|----------------|----------|---------------------------|-------------------------|
| symbol   | String         | True     | the currency to query     | BTCUSDT ...             |
| offset   | Integer        | False    | page start from 0         | start from 0, default 0 |
| limit    | Integer        | False    | page size                 | default 20, max 200     |

* Response

```json
[
  {
    "symbol": "ETHUSDT",
    "currency": "USDT",
    "execQtyRq": "0.16",
    "side": "Buy",
    "execPriceRp": "1322.84500459",
    "execValueRv": "211.65520073",
    "fundingRateRr": "0.0001",
    "feeRateRr": "0.0001",
    "execFeeRv": "0.02116552",
    "createTime": 1671004800021
  }
]
```

<a name="futureDataOrdersHist"/>

#### Query Orders History

* Http Request

```
GET /api-data/g-futures/orders?symbol=<symbol>
```

| Field    | Type           | Required | Description               | Possible Values                 |
|----------|----------------|----------|---------------------------|---------------------------------|
| symbol   | String         | True     | the currency to query     | BTCUSDT ...                     |
| start    | Long           | False    | start time in millisecond | default 2 days ago from the end |
| end      | Long           | False    | end time in millisecond   | default now                     |
| offset   | Integer        | False    | page start from 0         | start from 0, default 0         |
| limit    | Integer        | False    | page size                 | default 20, max 200             |

* Response

```
[
    {
        "actionTimeNs": 1667562110213260743,
        "bizError": 0,
        "clOrdId": "cfffa744-712d-867a-e397-9888eec3f6d1",
        "closedPnlRv": "0",
        "closedSizeRq": "0",
        "cumQtyRq": "0.001",
        "cumValueRv": "20.5795",
        "displayQtyRq": "0.001",
        "leavesQtyRq": "0",
        "leavesValueRv": "0",
        "orderId": "743fc923-cb01-4261-88d1-b35dba2cdac0",
        "orderQtyRq": "0.001",
        "ordStatus": "Filled",
        "ordType": "Market",
        "priceRp": "21206.7",
        "reduceOnly": false,
        "side": "Buy",
        "stopDirection": "UNSPECIFIED",
        "stopLossRp": "0",
        "symbol": "BTCUSDT",
        "takeProfitRp": "0",
        "timeInForce": "ImmediateOrCancel",
        "transactTimeNs": 1667562110221077395
    }
]
```

<a name="futureDataOrdersByIds"/>

#### Query Orders By Ids

* Http Request

```
GET /api-data/g-futures/orders/by-order-id?symbol=<symbol>
```

| Field    | Type     | Required | Description            | Possible Values                                                                                                                     |
|----------|----------|----------|------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| symbol   | String   | True     | the currency to query  | BTCUSDT ...                                                                                                                         |
| orderID  | String   | False    | order id               | orderID and clOrdID can not be both empty. If both IDs are given, it will return list of orders which match both orderID or clOrdID |
| clOrdID  | String   | False    | client order id        | refer to orderID                                                                                                                    |

* Response

```json
[
    {
        "orderId": "743fc923-cb01-4261-88d1-b35dba2cdac0",
        "clOrdId": "cfffa744-712d-867a-e397-9888eec3f6d1",
        "symbol": "BTCUSDT",
        "side": "Buy",
        "ordType": "Market",
        "actionTimeNs": 1667562110213260743,
        "priceRp": "21206.7",
        "orderQtyRq": "0.001",
        "displayQtyRq": "0.001",
        "timeInForce": "ImmediateOrCancel",
        "reduceOnly": false,
        "takeProfitRp": "0",
        "stopLossRp": "0",
        "closedPnlRv": "0",
        "closedSizeRq": "0",
        "cumQtyRq": "0.001",
        "cumValueRv": "20.5795",
        "leavesQtyRq": "0",
        "leavesValueRv": "0",
        "stopDirection": "UNSPECIFIED",
        "ordStatus": "Filled",
        "transactTimeNs": 1667562110221077395,
        "bizError": 0
    }
]
```

<a name="futureDataTradesHist"/>

#### Query Trades History

* Http Request

```
GET /api-data/g-futures/trades?symbol=<symbol>
```

| Field    | Type           | Required | Description               | Possible Values                 |
|----------|----------------|----------|---------------------------|---------------------------------|
| symbol   | String         | True     | the currency to query     | BTCUSDT ...                     |
| start    | Long           | False    | start time in millisecond | default 2 days ago from the end |
| end      | Long           | False    | end time in millisecond   | default now                     |
| offset   | Integer        | False    | page start from 0         | start from 0, default 0         |
| limit    | Integer        | False    | page size                 | default 20, max 200             |

* Response

```
[
    {
        "action": "New",
        "clOrdID": "",
        "closedPnlRv": "0",
        "closedSizeRq": "0",
        "currency": "USDT",
        "execFeeRv": "0.00166",
        "execID": "5c3d96e1-8874-53b6-b6e5-9dcc4d28b4ab",
        "execPriceRp": "16600",
        "execQtyRq": "0.001",
        "execStatus": "MakerFill",
        "execValueRv": "16.6",
        "feeRateRr": "0.0001",
        "orderID": "fcdfeafa-ed68-45d4-b2bd-7bc27f2b2b0b",
        "orderQtyRq": "0.001",
        "ordType": "LimitIfTouched",
        "priceRp": "16600",
        "side": "Sell",
        "symbol": "BTCUSDT",
        "tradeType": "Trade",
        "transactTimeNs": 1669407633926215067
    }
]
```

<a name="futureDataTradingFeesHist"/>

#### Query Trading Fees History

* Http Request

```
GET /api-data/g-futures/trading-fees?symbol=<symbol>
```

| Field    | Type           | Required | Description               | Possible Values         |
|----------|----------------|----------|---------------------------|-------------------------|
| symbol   | String         | True     | the currency to query     | BTCUSDT ...             |
| offset   | Integer        | False    | page start from 0         | start from 0, default 0 |
| limit    | Integer        | False    | page size                 | default 20, max 200     |

* Response

```json
[
  {
    "id": 295,
    "userId": 939565,
    "symbol": "ETHUSDT",
    "currency": "USDT",
    "takerValueRv": "97.1104",
    "takerFeeRateRr": "0.0006",
    "makerValueRv": "97.164",
    "makerFeeRateRr": "0",
    "exchangeFeeRv": "0.06798264",
    "createTime": 1669766400000
  }
]
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

<a name="rassub"/>

### Subscribe account margin (RAS)

RAS subscription requires the session been authorized successfully. DataGW extracts the user information from the given token and sends RAS messages back to client accordingly. Latest account snapshot messages will be sent to client immediately on subscription, and incremental messages will be sent for later updates. Each account snapshot contains one risk unit for cross margin positions, and each one risk unit for each isolated position. And also one risk wallet for each currency.

* Request:

```
{
    "id": <id>,
    "method": "ras_p.subscribe",
    "params": {}
}
```

* Response:

```
{
    "error": null,
    "id": <id>,
    "result": {
        "stauts": "success"
    }
}
```

* Sample
```
> {
    "id": 1234,
    "method": "ras_p.subscribe",
    "params": {}
}

< {
    "error": null,
    "id": 1234,
    "result": {
        "stauts": "success"
    }
}
```

#### account margin (RAS) Message Sample:

```json
{"risk_units":[{"estAvailableBalanceRv":"1806.82960617341","lastUpdateTimeNs":"2024-06-07T02:01:51.246394043Z","marginRatioRr":"999","posSide":0,"riskMode":"CrossAsset","symbol":"","totalBalanceRv":"1806.82960617341","totalEquityRv":"1806.82960617341","userID":944384,"userStatus":"Normal","userType":"Normal","valuationCurrency":"USDT","version":111},{"estAvailableBalanceRv":"-7.866995297237","fixedUsedRv":"8.180373498854","lastUpdateTimeNs":"2024-06-07T02:01:51.246394134Z","marginRatioRr":"14.28230196","posSide":3,"riskMode":"Isolated","symbol":"BTCUSDT","totalBalanceRv":"1075.34407386659","totalEquityRv":"1075.657452068207","totalPosCostRv":"1075.34407386659","totalPosMMRv":"74.741248391009","totalPosUnpnlRv":"0.313378201617","userID":944384,"userStatus":"Normal","userType":"Normal","valuationCurrency":"USDT","version":76}],"risk_wallets":[{"balanceRv":"2882.17368004","clReqVid":1,"currency":"USDT","lastUpdateTimeNs":"2024-06-07T02:01:51.246394235Z","userID":944384,"version":51}],"sequence":13144420,"timestamp":0,"type":"snapshot"}
```

| Field       | Type   | Description      | Possible values |
|-------------|--------|------------------|-----------------|
| timestamp   | Integer| Transaction timestamp in nanoseconds | |
| sequence    | Integer| Latest message sequence |          |
| type        | String | Message type     | snapshot, incremental |

#### Fields in RiskUnit

| Field    | Type     | Description    | Possible Values |
|----------|----------|----------------|-----------------|
| riskMode | String | "CrossAsset" for Cross Margin Positions, and "Isolated" for isolated position | 1, 3 |
| estAvailableBalanceRv | String | estimated available balance for new orders | |
| fixedUsedRv | String | margins allocated to fully hedged positions and bankrupt commission | |
| lastUpdateTimeNs | Integer | the time in ns the message is generated | |
| MarginRatioRr | String | the margin ratio level for the current risk unit | | 
| posSide | Integer | the position side | 1, 2, 3 |
| symbol      | String | Contract symbol name    |          |
| totalBalanceRv | String | sum of balanceRv of all risk wallet | |
| totalEquityRv | String | total equity excluding debt and interest | |
| totalPosCostRv | String | total initial margin of position(s) | |
| totalPosMMRv | String | total maintainence margin of position(s) | |
| totalPosUnpnlRv | String | sum of unrealised pnl of position(s) | |
| userID | Integer | user id | |
| userStatus | String | user status | "Unspecified/Normal" for normal, "Banned" for banned, "Liq*" for liquidation |
| userType | String| user type | always "Normal" for user |
| valuationCurrency | String | settle currency | |
| version | Integer | risk unit version | |

#### Fields of RiskWallet 

| Field    | Type    | Description   | Possible values |
|----------|---------|---------------|-----------------|
| balanceRv | String | available balance, including bonus and debt | | 
| currency | String | wallet currency | |
| lastUpdateTimeNs | Integer | the time in ns the message is generated | |
| userID | Integer | user id | |
| version | Integer | wallet version | |




<a name="rasunsub">

### Unsubscribe account margin (RAS)

* Request:

```
{
    "id": <id>,
    "method": "ras_p.unsubscribe",
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

