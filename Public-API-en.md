
# Phemex Public API

## General Public API Information
* Phemex provides HTTP Rest API for client to operate Orders, all endpoints return a JSON object.
* The Rest API base endpoint is: **https://api.phemex.com**, or for the testnet is:  **https://testnet-api.phemex.com** 
* Phemex provides WebSocket API for client to receive market data, order and position updates.
* The WebSocket API url is: **wss://phemex.com/ws**, or for the testnet is:  **wss://testnet.phemex.com/ws** 

# REST API Standards
## HTTP Return Codes

* HTTP `403` return code is used when IP break Rate Limit.
* HTTP `429` return code is used when breaking a request rate limit.
* HTTP `5XX` return codes are used for Phemex internal errors. Note: This doesn't means the operation failed, the execution status is **UNKNOWN** and could be Succeed.


## HTTP REST Request Header 

Every HTTP Rest Request must have the following Headers:
* x-phemex-access-token : This is public API Token from Phemex site.
* x-phemex-request-expiry : This describes the Unix EPoch millisecond to expire the request, normally it should be (Now() + 1 minute)
* x-phemex-request-signature : This is HMAC SHA256 signature of the http request, its formula is : HMacSha256( URL Path + QueryString + Expiry + body )


## API Rate Limits
* Every Client has the API call rate limit as 200 per minute.
* Every HTTP Rest response will contain a `X-RateLimit-Remaining` header which has the remain request count in this round.
* When client gets HTTP RESPONSE Code 429 means it reached limit, `X-RateLimit-Retry-After` header means the seconds that client should wait before next round.


## Endpoint security type
* Each API call must be signed and pass to server in HTTP header `x-phemex-request-signature`.
* Endpoints use `HMAC SHA256` signatures. The `HMAC SHA256 signature` is a keyed `HMAC SHA256` operation. Use your `secretKey` as the key and the string `URL Path + QueryString + Expiry + body )` as the value for the HMAC operation.
* The `signature` is **case sensitive**.


### Signature Example 1: HTTP GET Request
* API REST Request URL: https://api.phemex.com/accounts/accountPositions?currency=BTC
   * Request Path: /accounts/accountPositions
   * Request Query: currency=BTC
   * Request Body: <null>
   * Request Expiry: 154567778
   * Signature: HMacSha256( /accounts/accountPositions + currency=BTC + 154567778 )

### Signature Example 2: HTTP POST Request
* API REST Request URL: https://api.phemex.com/orders
   * Request Path: /orders
   * Request Query: <null>
   * Request Body: {"symbol":"BTCUSD","clOrdID":"uuid-1573058952273","side":"Sell","priceEp":93185000,"orderQty":7,"ordType":"Limit","reduceOnly":false,"timeInForce":"GoodTillCancel","takeProfitEp":0,"stopLossEp":0}
   * Request Expiry: 154567778
   * Signature: HMacSha256( /orders + currency=BTC + 154567778 + {"symbol":"BTCUSD","clOrdID":"uuid-1573058952273","side":"Sell","priceEp":93185000,"orderQty":7,"ordType":"Limit","reduceOnly":false,"timeInForce":"GoodTillCancel","takeProfitEp":0,"stopLossEp":0})
   
   
## REST API List

### Place Order 

* HTTP Request:

```json
POST /orders

{
"actionBy":"FromOrderPlacement",
"symbol":"BTCUSD",
"clOrdID":"uuid-1573058952273",
"side":"Sell",
"priceEp":93185000,
"orderQty":7,
"ordType":"Limit",
"reduceOnly":false,
"triggerType":"UNSPECIFIED",
"pegPriceType":"UNSPECIFIED",
"timeInForce":"GoodTillCancel",
"takeProfitEp":0,
"stopLossEp":0
}

```

* HTTP Response:

### Cacnel Order Request

```json
DELETE /orders/orderID=<xxx>&symbol=<xxx>


```

### Cacnel All Orders Request

```json
DELETE /orders/all


```



# Websocket API Standards

## Session Management
* Each client is required to actively send heartbeat (ping) message to Phemex data gateway ('DataGW' in short) with interval less than 30 seconds, otherwise DataGW will drop the connection. If a client sends a ping message, DataGW will reply with a pong message.
* Clients can use WS built-in ping message or the application level ping message to DataGW as heartbeat. The heartbeat interval is recommended to be set as *5 seconds*, and actively reconnect to DataGW if don't receive messages in *3 heartbeat intervals*.

## API Rate Limits
* Every Client has subscription limit up to 20 per-connection.
* When client exccedes the subscription limit, the connection will be dropped by DataGW.


## WebSocket API List

### Heartbeat
* Request：
```json
{"id": <id>, "method": "server.ping", "params": []}
```

* Response:
```json
{"error":null,"id":<id>,"result":"pong"}
```

* Sample：
```json
> {"id": 1234, "method": "server.ping", "params": []}
< {"error":null,"id":1234,"result":"pong"}
```

### API User Authentication
Market trade/orderbook are published publicly without user authentication.
While for client private account/position/order data, the client should send user.auth message to Data Gateway to authenticate the session.

* Request

```json
{"method": "user.auth", "params": ["API", "<token>", "<signature>", <expiry>], "id": 1234}
```

| Field       | Type   | Description      | Possible values |
|-------------|--------|------------------|-----------------|
| type        | String | token type       | API             |
| token       | String | access token     |                 |
| signature   | String | signature generated by a funtion as HMacSha256(access-token + expiry) ||
| expiry      | Integer| expiry in epoch second which is between now and now+5 minutes. ||

* Sample:

```json
> {"method": "user.auth", "params": ["API", "806066b0-f02b-4d3e-b444-76ec718e1023", "8c939f7a6e6716ab7c4240384e07c81840dacd371cdcf5051bb6b7084897470e", 157009123213], "id": 1234}
< {"error":null,"id":1234,"result":{"status":"success"}}
```


### Subscribe OrderBook 
When Client subscribe to OrderBook, Data Gateway will immediately send the current Order Book snapshot to client. 

* Request 
```json
{"id": <id>, "method": "orderbook.subscribe", "params": ["<symbol>"]}
```

* Response
```json
{"error":null,"id":<id>,"result":{"status":"success"}}
```

* Sample：
```json
> {"id":1234, "method": "orderbook.subscribe", "params": ["BTCUSD"]}
< {"error":null,"id":1234,"result":{"status":"success"}}
```


#### OrderBook Message:

* Message Format：
 
```json
{"book":{"asks":[["<price>","<qty>"],["<prie>","<qty>"]],"bids":[["<price>","<qty>"],["<prie>","<qty>"]]},"depth":"<depth>","sequence":<sequence>,"symbol":"<symbol>","type":"<type>"}
```

| Field       | Type   | Description      | Possible values |
|-------------|--------|------------------|-----------------|
| side        | String | price level side | bid, ask        |
| sequence    | Integer| latest message sequence |          |
| depth       | Integer| market depth     |                 |
| type        | String | message type     | snapshot, incremental |
  

* Sample：
 
```json
< {"book":{"asks":[[112710000,6239]],"bids":[]},"depth":100,"sequence":11433,"symbol":"BTCUSD","type":"incremental"}
< {"book":{"asks":[[112710000,6240],[112720000,10715],[112730000,5508],[112740000,10369],[112760000,6531],[112770000,4050],[112780000,11655],[112790000,2926],[112820000,12345],[112840000,4974],[112850000,14943],[112860000,9094],[112870000,3197],[112880000,2110],[112900000,2680],[112910000,7115],[112920000,8283],[112940000,12006],[112950000,23126],[112960000,8337],[112970000,6194],[112980000,3211],[112990000,15660],[113010000,4768],[113020000,10700],[113040000,9493],[113050000,9116],[113060000,3244],[113070000,4517],[113080000,7165],[113090000,8266],[113100000,8791],[113110000,25330],[113140000,14207],[113150000,5405],[113180000,6729],[113190000,13190],[113200000,3161],[113210000,21047],[113220000,6500],[113230000,14260],[113240000,6406],[113250000,3131],[113260000,1419],[113270000,11411],[113280000,9944],[113290000,6412],[113300000,2499],[113310000,10922],[113320000,6241],[113330000,20190],[113340000,9612],[113350000,3231],[113360000,14082],[113370000,6354],[113380000,10542],[113390000,5454],[113400000,10179],[113410000,2098],[113420000,10847],[113430000,6607],[113440000,4511],[113460000,1053],[113470000,7307],[113490000,13161],[113500000,9660],[113510000,13346],[113520000,2990],[113530000,7860],[113540000,4258],[113560000,13760],[113570000,12311],[113580000,9341],[113590000,7730],[113600000,1222],[113610000,31722],[113620000,24895],[113630000,15907],[113660000,2318],[113690000,4303],[113710000,6089],[113720000,16478],[113740000,14026],[113750000,13597],[113760000,13939],[113770000,5800],[113780000,23854],[113800000,9615],[113810000,5727],[113820000,3469],[113840000,7026],[113860000,4679],[113880000,6702],[113890000,9760],[113900000,17147],[113910000,11451],[113920000,2270],[113940000,13231],[113960000,15123],[113980000,1113]],"bids":[[98065000,100],[98005000,1990],[88030000,18470],[88010000,6072],[87990000,14041],[87980000,9960],[87970000,5657],[87960000,8121],[87950000,9124],[87940000,6169],[87930000,11073],[87920000,30521],[87890000,8602],[87870000,6539],[87840000,29245],[87800000,25975],[87790000,14806],[87760000,8329],[87750000,4409],[87740000,10169],[87730000,12616],[87720000,6236],[87700000,10262],[87690000,5127],[87670000,4954],[87660000,16190],[87650000,5179],[87640000,8840],[87630000,7351],[87620000,6411],[87610000,4325],[87600000,15220],[87590000,1502],[87570000,18821],[87560000,23083],[87540000,22458],[87530000,13219],[87520000,14664],[87510000,12328],[87500000,1686],[87490000,17736],[87480000,8427],[87470000,18272],[87460000,16570],[87450000,18492],[87440000,3767],[87430000,3834],[87410000,18767],[87390000,6325],[87380000,9262],[87370000,7588],[87360000,9796],[87330000,4574],[87320000,9038],[87260000,5742],[87250000,18739],[87240000,8384],[87220000,13103],[87210000,11590],[87200000,14449],[87190000,2644],[87180000,6027],[87170000,2612],[87160000,5295],[87150000,19365],[87140000,7487],[87120000,2259],[87110000,2734],[87080000,15210],[87060000,7814],[87050000,5401],[87040000,9451],[87030000,14342],[87010000,13314],[87000000,20168],[86990000,4162],[86970000,6188],[86960000,11235],[86940000,1702],[86930000,1990],[86920000,2853],[86910000,14572],[86900000,8002],[86890000,3164],[86880000,9704],[86870000,13546],[86860000,8925],[86850000,4511],[86830000,11265],[86820000,12288],[86810000,18203],[86800000,14108],[86790000,2533],[86780000,14546],[86760000,5742],[86750000,15569],[86730000,2830],[86720000,13425],[86710000,13759],[86700000,16944]]},"depth":100,"sequence":11438,"symbol":"BTCUSD","type":"snapshot"}

```

###  Unsubscribe OrderBook
It unsubscribes all orderbook related subscriptions.

* Request

```json
{"id": 1234, "method": "orderbook.unsubscribe", "params": []}
```

* Response:

```json
{"error":null,"id":1234,"result":{"status":"success"}}
```


### Subscribe Trade
After each Trade Subscribe, Data Gateway will publish the 1000 history trades and latest trades to the client 

* Request

```json
{"id": <id>, "method": "trade.subscribe", "params": ["<symbol>"]}
```

* Response:

```json
{"error":null,"id":<id>,"result":{"status":"success"}}

```

* Sample:

```json
> {"id": 1234, "method": "trade.subscribe", "params": ["BTCUSD"]}
< {"error":null,"id":1234,"result":{"status":"success"}}
```



#### Trade Message Format：

```json
{"trade":[["<timestamp>","<side>","<price>","<qty>"],["<timestamp>","<side>","<prie>","<qty>"]],"sequence":<sequence>,"timestamp":"<timestamp2>","symbol":"<symbol>","type":"<type>"}
```

| Field       | Type   | Description      | Possible values |
|-------------|--------|------------------|-----------------|
| timestamp2  | Integer| transaction timestmap in nanoseconds for the message ||
| timestamp   | Integer| timestamp in nanoseconds for each trade ||
| side        | String | trade side       | bid, ask        |
| sequence    | Integer| latest message sequence ||
| symbol      | String | trade symbol     ||
| type        | String | message type     |snapshot, incremental |
  

* Sample
```json
< {"sequence":4,"symbol":"BTCUSD","timestamp":1568689385584237387,"trades":[[1568689385584237387,"sell","999","1"],[1568689385584237387,"sell","999","1"],[1568689385584237387,"buy","1000","1"],[1568689385584237387,"buy","1000","1"],[1568690355567123,"sell","999","1"],[1568689385584232,"sell","999","1"],[1568689385584111,"sell","999","10"]],"type":"snapshot"}
< {"sequence":10,"symbol":"BTCUSD","timestamp":1568727884871777954,"trades":[[1568727884871288,"sell","999","1"]],"type":"incremental"}
```

### Unsubscribe  Trade
It unsubscribes will trade subscriptions.

* Request

```json
{"id": <id>, "method": "trade.unsubscribe", "params": []}
```

* Response:

```json
{"error":null,"id":1234,"result":{"status":"success"}}
```


### Subscribe Account-Order-Position (AOP)
AOP subscription requires the session been authorized successfully. DataGW extracts the user information from the given token and sends AOP messages back to client accordingly. 0 or more latest account snapshot messages will be sent to client immediately on subscription, and incremental messages will be sent for later updates. Each account snapshot contains a trading account information, holding positions, and open / max 100 closed / max 100 filled order event message history.

* Request

```json
{"id": <id>, "method": "aop.subscribe", "params": []}
```

* Response:

```json
{"error":null,"id":1234,"result":{"status":"success"}}
```

* Sample
```json
> {"id": 1234, "method": "aop.subscribe", "params": []}
< {"error":null,"id":1234,"result":{"status":"success"}}
```




#### Account-Order-Position (AOP) Message Sample:

```json
{"accounts":[{"accountBalanceEv":9992165009,"accountID":604630001,"currency":"BTC","totalUsedBalanceEv":10841771568,"userID":60463}],"orders":[{"accountID":604630001,...}],"positions":[{"accountID":604630001,...}],"sequence":11450, "timestamp":<timestamp>, "type":"<type>"}

```

| Field       | Type   | Description      | Possible values |
|-------------|--------|------------------|-----------------|
| timestamp   | Integer| transaction timestamp in nanoseconds | |
| sequence    | Integer| latest message sequence |          |
| symbol      | String |                  |                 |
| type        | String | message type     | snapshot, incremental |



* Sample:

```json
<  {"accounts":[{"accountBalanceEv":96049635244492,"accountID":659220002,"bonusBalanceEv":0,"currency":"USD","totalUsedBalanceEv":1032761766,"userID":65922}],"orders":[{"accountID":659220002,"action":"New","actionTimeNs":0,"addedSeq":394480,"bonusChangedAmountEv":0,"clOrdID":"1573124279267067022","closedPnlEv":0,"closedSize":0,"code":0,"cumQty":0,"cumValueEv":0,"curAccBalanceEv":96049635244492,"curAssignedPosBalanceEv":0,"curBonusBalanceEv":0,"curLeverageEr":0,"curPosSide":"Buy","curPosSize":18132,"curPosTerm":376,"curPosValueEv":168912309,"curRiskLimitEv":5000000000,"cxlRejReason":0,"displayQty":0,"execFeeEv":0,"execID":"00000000-0000-0000-0000-000000000000","execPriceEp":0,"execQty":0,"execSeq":394480,"execStatus":"New","execValueEv":0,"feeRateEr":0,"leavesQty":297,"leavesValueEv":2773237,"message":"No error","nthItem":1,"ordStatus":"New","ordType":"Limit","orderID":"85906b94-3784-4e15-ac19-590bc5af4aff","orderQty":297,"pegOffsetValueEp":0,"priceEp":1867500,"relatedPosTerm":376,"relatedReqNum":66987,"side":"Sell","stopLossEp":0,"stopPxEp":0,"symbol":"ETHUSD","takeProfitEp":0,"timeInForce":"PostOnly","totalItems":1,"transactTimeNs":1573131627774194463,"userID":0,"vsAccountID":0,"vsUserID":0}],"positions":[{"accountID":659220002,"assignedPosBalanceEv":0,"avgEntryPriceEp":1863140,"bankruptCommEv":34,"bankruptPriceEp":500,"buyLeavesQty":427619,"buyLeavesValueEv":3967934705,"buyValueToCostEr":5146250,"createdAtNs":0,"crossSharedBalanceEv":96048602482726,"cumClosedPnlEv":-169296225,"cumFundingFeeEv":0,"cumTransactFeeEv":97346,"currency":"USD","dataVer":66991,"deleveragePercentileEr":0,"displayLeverageEr":1000000,"estimatedOrdLossEv":0,"execSeq":394480,"freeCostEv":-146741524,"freeQty":0,"initMarginReqEr":5000000,"lastFundingTime":1573117589973342546,"lastTermEndTime":1573130627750900195,"leverageEr":0,"liquidationPriceEp":500,"maintMarginReqEr":1000000,"makerFeeRateEr":0,"markPriceEp":1861319,"orderCostEv":204199840,"posCostEv":8565966,"positionMarginEv":96048611213787,"positionStatus":"Normal","riskLimitEv":5000000000,"sellLeavesQty":137554,"sellLeavesValueEv":1284157893,"sellValueToCostEr":5153750,"side":"Buy","size":18132,"symbol":"ETHUSD","takerFeeRateEr":0,"term":376,"transactTimeNs":1573131627774194463,"unrealisedPnlEv":-165129,"updatedAtNs":0,"usedBalanceEv":212930935,"userID":65922,"valueEv":168912309}],"sequence":257977,"timestamp":1573131627776792763,"type":"incremental"}

```


### Unsubscribe Account-Order-Position (AOP)
* Request：

```json
{"id": <id>, "method": "aop.unsubscribe", "params": []}

```

* Response:

```json
{"error":null,"id":<id>,"result":{"status":"success"}}
```


