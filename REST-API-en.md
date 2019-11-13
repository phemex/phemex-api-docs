
# Public Rest API for Phemex

## General API Information
* The base endpoint is: **https://api.phemex.com** or for the testnet is:  **https://testnet-api.phemex.com** 
* All endpoints return a JSON object.
* Phemex provides HTTP Rest API for client to operate Orders.

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

HTTP Request:

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

HTTP Response:

### Cacnel Order Request

```json
DELETE /orders/orderID=<xxx>&symbol=<xxx>


```

### Cacnel All Orders Request

```json
DELETE /orders/all


```

