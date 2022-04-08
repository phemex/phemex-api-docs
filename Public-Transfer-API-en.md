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
    * [Transfer Api List](#transferApiList)
      * [Transfer Between Spot and Futures](#assetsTransfer)
      * [Query Transfer History](#assetsTransferQuery)
      * [Spot Sub To Main Transfer](#spotSubToMainTransfer)
      * [Query Spot Sub To Main Transfer](#spotSubToMainTransferQuery)
      * [Futures Sub To Main Transfer](#futuresSubToMainTransfer)
      * [Query Futures Sub To Main Transfer](#futuresSubToMainTransferQuery)
      * [Universal Transfer](#universalTransfer)
    * [Convert Api List](#convertApiList)
      * [RFQ Quote](#rfqQuote)
      * [Convert](#convert)
      * [Query Convert History](#convertQuery)

      
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
* RateLimit group of contract trading api is ***SPOTORDER***.  
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

| currency | value scale factor | min value  |  max value | need addr arg |
|--------|---------------|------------|------------|------------|
|BTC|8|1|5e+18|0|
|USDT|8|1|5e+18|0|
|ETH|8|1|5e+18|0|
|XRP|8|1|5e+18|1|
|LINK|8|1|5e+18|0|
|XTZ|8|1|5e+18|0|
|LTC|8|1|5e+18|0|
|ADA|8|1|5e+18|0|
|TRX|8|1|5e+18|0|
|ONT|8|1|5e+18|0|
|BCH|8|1|5e+18|0|
|NEO|8|1|5e+18|0|
|EOS|8|1|5e+18|1|
|COMP|8|1|5e+18|0|
|LEND|8|1|5e+18|0|
|YFI|8|1|5e+18|0|
|DOT|8|1|5e+18|0|
|UNI|8|1|5e+18|0|
|AAVE|8|1|5e+18|0|
|DOGE|8|1|5e+18|0|
|BAT|8|1|5e+18|0|
|CHZ|8|1|5e+18|0|
|MANA|8|1|5e+18|0|
|ENJ|8|1|5e+18|0|
|SUSHI|8|1|5e+18|0|
|SNX|8|1|5e+18|0|
|GRT|8|1|5e+18|0|
|MKR|8|1|5e+18|0|
|ALGO|8|1|5e+18|0|
|VET|8|1|5e+18|0|
|ZEC|8|1|5e+18|0|
|FIL|8|1|5e+18|0|
|KSM|8|1|5e+18|0|
|XMR|8|1|5e+18|0|
|QTUM|8|1|5e+18|0|
|XLM|8|1|5e+18|1|
|ATOM|8|1|5e+18|1|
|LUNA|8|1|5e+18|1|
|SOL|8|1|5e+18|0|
|AXS|8|1|5e+18|0|
|MATIC|8|1|5e+18|0|
|SHIB|2|1|5e+18|0|
|FTM|8|1|5e+18|0|
|DYDX|8|1|5e+18|0|


<a name="spotSymList"/>

* Spot symbol and its scale factor

All spot symbols use the same price and ratio scale factors (price scale facor(Ep) = 8, ratio scale factor(Er) = 8).

|  symbol|  type | price scale factor | ratio scale factor |  baseTickSize|  baseTickSizeEv|  quoteTickSize|  quoteTickSizeEv|  baseQtyPrecision|  quoteQtyPrecision|  pricePrecision|  minOrderValue|  minOrderValueEv|  maxBaseOrderSize|  maxBaseOrderSizeEv|  maxOrderValue|  maxOrderValueEv|  defaultTakerFee|  defaultTakerFeeEr|  defaultMakerFee|  defaultMakerFeeEr|
|---------|---------|---------|---------|-------------|-----------|-----------------|------------|------------|-------------|------|--------|-------|------|--------|-------|------|------|-------|-------|--------|
|sBTCUSDT|Spot|8|8|0.000001 BTC|100|0.01 USDT|1000000|6|2|2|10 USDT|1000000000|1000 BTC|100000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sETHUSDT|Spot|8|8|0.00001 ETH|1000|0.01 USDT|1000000|5|2|2|10 USDT|1000000000|10000 ETH|1000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sXRPUSDT|Spot|8|8|0.1 XRP|10000000|0.00001 USDT|1000|1|2|5|10 USDT|1000000000|5000000 XRP|500000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sLINKUSDT|Spot|8|8|0.01 LINK|1000000|0.0001 USDT|10000|2|2|4|10 USDT|1000000000|5000000 LINK|500000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sXTZUSDT|Spot|8|8|0.01 XTZ|1000000|0.0001 USDT|10000|2|2|4|10 USDT|1000000000|2000000 XTZ|200000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sLTCUSDT|Spot|8|8|0.00001 LTC|1000|0.01 USDT|1000000|5|2|2|10 USDT|1000000000|100000 LTC|10000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sADAUSDT|Spot|8|8|0.1 ADA|10000000|0.00005 USDT|5000|1|2|5|10 USDT|1000000000|5000000 ADA|500000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sTRXUSDT|Spot|8|8|0.1 TRX|10000000|0.00005 USDT|5000|1|2|5|10 USDT|1000000000|5000000 TRX|500000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sONTUSDT|Spot|8|8|0.1 ONT|10000000|0.0005 USDT|50000|1|2|4|10 USDT|1000000000|5000000 ONT|500000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sBCHUSDT|Spot|8|8|0.00001 BCH|1000|0.01 USDT|1000000|5|2|2|10 USDT|1000000000|10000 BCH|1000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sNEOUSDT|Spot|8|8|0.001 NEO|100000|0.001 USDT|100000|3|2|3|10 USDT|1000000000|5000000 NEO|500000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sEOSUSDT|Spot|8|8|0.01 EOS|1000000|0.0001 USDT|10000|2|2|4|10 USDT|1000000000|5000000 EOS|500000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sDOGEUSDT|Spot|8|8|1 DOGE|100000000|0.000001 USDT|100|0|2|6|10 USDT|1000000000|5000000 DOGE|500000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sBATUSDT|Spot|8|8|0.01 BAT|1000000|0.00001 USDT|1000|2|2|5|10 USDT|1000000000|300000 BAT|30000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sCHZUSDT|Spot|8|8|0.01 CHZ|1000000|0.00001 USDT|1000|2|2|5|10 USDT|1000000000|700000 CHZ|70000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sMANAUSDT|Spot|8|8|0.01 MANA|1000000|0.00001 USDT|1000|2|2|5|10 USDT|1000000000|400000 MANA|40000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sENJUSDT|Spot|8|8|0.01 ENJ|1000000|0.00001 USDT|1000|2|2|5|10 USDT|1000000000|200000 ENJ|20000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sSUSHIUSDT|Spot|8|8|0.001 SUSHI|100000|0.001 USDT|100000|3|2|3|10 USDT|1000000000|40000 SUSHI|4000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sSNXUSDT|Spot|8|8|0.001 SNX|100000|0.001 USDT|100000|3|2|3|10 USDT|1000000000|30000 SNX|3000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sGRTUSDT|Spot|8|8|0.01 GRT|1000000|0.00001 USDT|1000|2|2|5|10 USDT|1000000000|200000 GRT|20000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sUNIUSDT|Spot|8|8|0.001 UNI|100000|0.001 USDT|100000|3|2|3|10 USDT|1000000000|15000 UNI|1500000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sAAVEUSDT|Spot|8|8|0.0001 AAVE|10000|0.01 USDT|1000000|4|2|2|10 USDT|1000000000|1000 AAVE|100000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sYFIUSDT|Spot|8|8|0.00001 YFI|1000|0.01 USDT|1000000|5|2|2|10 USDT|1000000000|10 YFI|1000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sCOMPUSDT|Spot|8|8|0.0001 COMP|10000|0.01 USDT|1000000|4|2|2|10 USDT|1000000000|1000 COMP|100000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sMKRUSDT|Spot|8|8|0.00001 MKR|1000|0.01 USDT|1000000|5|2|2|10 USDT|1000000000|100 MKR|10000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sDOTUSDT|Spot|8|8|0.001 DOT|100000|0.001 USDT|100000|3|2|3|10 USDT|1000000000|10000 DOT|1000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sALGOUSDT|Spot|8|8|0.01 ALGO|1000000|0.00001 USDT|1000|2|2|5|10 USDT|1000000000|300000 ALGO|30000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sVETUSDT|Spot|8|8|0.1 VET|10000000|0.000001 USDT|100|1|2|6|10 USDT|1000000000|2000000 VET|200000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sZECUSDT|Spot|8|8|0.0001 ZEC|10000|0.01 USDT|1000000|4|2|2|10 USDT|1000000000|2000 ZEC|200000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sFILUSDT|Spot|8|8|0.0001 FIL|10000|0.01 USDT|1000000|4|2|2|10 USDT|1000000000|3000 FIL|300000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sKSMUSDT|Spot|8|8|0.0001 KSM|10000|0.01 USDT|1000000|4|2|2|10 USDT|1000000000|1000 KSM|100000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sXMRUSDT|Spot|8|8|0.0001 XMR|10000|0.01 USDT|1000000|4|2|2|10 USDT|1000000000|1500 XMR|150000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sQTUMUSDT|Spot|8|8|0.001 QTUM|100000|0.001 USDT|100000|3|2|3|10 USDT|1000000000|30000 QTUM|3000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sXLMUSDT|Spot|8|8|0.01 XLM|1000000|0.00001 USDT|1000|2|2|5|10 USDT|1000000000|1000000 XLM|100000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sATOMUSDT|Spot|8|8|0.001 ATOM|100000|0.001 USDT|100000|3|2|3|10 USDT|1000000000|20000 ATOM|2000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sLUNAUSDT|Spot|8|8|0.001 LUNA|100000|0.001 USDT|100000|3|2|3|10 USDT|1000000000|40000 LUNA|4000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sSOLUSDT|Spot|8|8|0.001 SOL|100000|0.001 USDT|100000|3|2|3|10 USDT|1000000000|10000 SOL|1000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sAXSUSDT|Spot|8|8|0.001 AXS|100000|0.001 USDT|100000|3|2|3|10 USDT|1000000000|10000 AXS|1000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sMATICUSDT|Spot|8|8|0.01 MATIC|1000000|0.00001 USDT|1000|2|2|5|10 USDT|1000000000|300000 MATIC|30000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sSHIBUSDT|Spot|8|8|1 SHIB|100|0.00000001 USDT|1|0|2|8|10 USDT|1000000000|5000000000 SHIB|500000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sFTMUSDT|Spot|8|8|0.01 FTM|1000000|0.00001 USDT|1000|2|2|5|10 USDT|1000000000|200000 FTM|20000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|
|sDYDXUSDT|Spot|8|8|0.001 DYDX|100000|0.001 USDT|100000|3|2|3|10 USDT|1000000000|30000 DYDX|3000000000000|5000000 USDT|500000000000000|0.001|100000|0.001|100000|


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

* Transfer Status

| Transfer Status | Description                     | 
|-----------------|---------------------------------|
| 3               | Rejected                        |
| 6               | Got error and wait for recovery |
| 10              | Success                         |
| 11              | Failed                          |
| others          | Under processing                |


<a name="restapilist"/>

## REST API List

<a name="transferApiList"/>

### Transfer API List

<a name="assetsTransfer"/>

#### Transfer Between Spot and Futures 

* Http Request

```
POST /assets/transfer
```

| Field    | Type    | Required  | Description           | Possible Values                            |
|----------|---------|-----------|-----------------------|--------------------------------------------|
| amountEv | Long    | True      | amountEv to transfer  | 100000 ...                                 |
| moveOp   | Integer | True      | direction             | 1 - futures to spot, 2 - spot to futures   |
| currency | String  | True      | currency to transfer  | BTC, ETH, USD ...                          |

* Request

```
{
  "amountEv": 0,
  "currency": "string",
  "moveOp": 0
}
```

* Response

```
{
  "amountEv": 0,
  "currency": "string",
  "linkKey": "string",
  "side": 0,
  "status": 0,
  "userId": 0
}
```

<a name="assetsTransferQuery"/>

#### Query Transfer History

* Http Request

```
GET /assets/transfer?currency=BTC
```

| Field    | Type    | Required | Description               | Possible Values                 |
|----------|---------|----------|---------------------------|---------------------------------|
| currency | String  | True     | the currency to query     | BTC,ETH ...                     |
| start    | Long    | False    | start time in millisecond | default 2 days ago from the end |
| end      | Long    | False    | end time in millisecond   | default now                     |
| offset   | Integer | False    | page start from 0         | start from 0, default 0         |
| limit    | Integer | False    | page size                 | default 20, max 200             |

* Response

```
[
  {
    "amountEv": 0,
    "bizType": 0,
    "createTime": 0,
    "currency": "string",
    "linkKey": "string",
    "side": 0,
    "status": 0,
    "userId": 0
  }
]
```

<a name="spotSubToMainTransfer"/>

#### Spot Sub To Main Transfer (for sub-account only)

* Http Request

```
POST /assets/spots/sub-accounts/transfer
```

| Field      | Type    | Required | Description          | Possible Values                                       |
|------------|---------|----------|----------------------|-------------------------------------------------------|
| amountEv   | Long    | True     | amountEv to transfer | 100000 ...                                            |
| currency   | String  | True     | currency to transfer | BTC, ETH, USD ...                                     |
| requestKey | String  | False    | unique request key   | Unique request Key, system will generate if its empty |

* Request
```
{
  "amountEv": 0,
  "currency": "string",
  "requestKey": "string"
}
```

* Response

```
{
  "amountEv": 0,
  "currency": "string",
  "fromUserId": 0,
  "requestKey": "string",
  "toUserId": 0
}
```

<a name="spotSubToMainTransferQuery"/>

#### Query Spot Sub To Main Transfer

* Http Request

```
GET /assets/spots/sub-accounts/transfer?currency=BTC
```

| Field    | Type    | Required | Description               | Possible Values                 |
|----------|---------|----------|---------------------------|---------------------------------|
| currency | String  | True     | the currency to query     | BTC,ETH ...                     |
| start    | Long    | False    | start time in millisecond | default 2 days ago from the end |
| end      | Long    | False    | end time in millisecond   | default now                     |
| offset   | Integer | False    | page start from 0         | start from 0, default 0         |
| limit    | Integer | False    | page size                 | default 20, max 200             |

* Response

```
[
  {
    "amountEv": 0,
    "createAt": 0,
    "currency": "string",
    "fromUserId": 0,
    "id": 0,
    "requestKey": "string",
    "status": 0,
    "toUserId": 0
  }
]
```


<a name="futuresSubToMainTransfer"/>

#### Futures Sub To Main Transfer (for sub-account only)

* Http Request

```
POST /assets/futures/sub-accounts/transfer
```

| Field      | Type    | Required | Description          | Possible Values                                       |
|------------|---------|----------|----------------------|-------------------------------------------------------|
| amountEv   | Long    | True     | amountEv to transfer | 100000 ...                                            |
| currency   | String  | True     | currency to transfer | BTC, ETH, USD ...                                     |
| requestKey | String  | False    | unique request key   | Unique request Key, system will generate if its empty |

* Request
```
{
  "amountEv": 0,
  "currency": "string",
  "requestKey": "string"
}
```

* Response

```
{
  "amountEv": 0,
  "bizCode": 0,
  "currency": "string",
  "fromUserId": 0,
  "requestKey": "string",
  "toUserId": 0
}
```

<a name="futuresSubToMainTransferQuery"/>

#### Query Futures Sub To Main Transfer

* Http Request

```
GET /assets/futures/sub-accounts/transfer?currency=BTC
```

| Field    | Type    | Required | Description               | Possible Values                 |
|----------|---------|----------|---------------------------|---------------------------------|
| currency | String  | True     | the currency to query     | BTC,ETH ...                     |
| start    | Long    | False    | start time in millisecond | default 2 days ago from the end |
| end      | Long    | False    | end time in millisecond   | default now                     |
| offset   | Integer | False    | page start from 0         | start from 0, default 0         |
| limit    | Integer | False    | page size                 | default 20, max 200             |

* Response

```
[
  {
    "fromUserId": 0,
    "toUserId": 0,
    "currency": "string",
    "amountEv": 0,
    "bizCode": 0,
    "requestKey": "string",
    "createAt": 0
  }
]
```
<a name="universalTransfer"/>

#### Universal Transfer(Main account only) - Transfer between sub to main, main to sub or sub to sub

* Http Request

```
POST /assets/universal-transfer
```

| Field       | Type    | Required | Description                 | Possible Values                                       |
|-------------|---------|----------|-----------------------------|-------------------------------------------------------|
| fromUserId  | Long    | True     | from user id                | Will set as main account if not given or 0            |
| toUserId    | Long    | True     | to user id                  | Will set as main account if not given or 0            |
| amountEv    | Long    | True     | amountEv to transfer        | 100000 ...                                            |
| bizType     | String  | True     | transfer for which biz type | SPOT, PERPETUAL                                       |
| requestKey  | String  | False    | unique request key          | Unique request Key, system will generate if its empty |

* Request
```
{
  "amountEv": 0,
  "bizType": "string",
  "currency": "string",
  "fromUserId": 0,
  "toUserId": 0
}
```

* Response

```
{
  "requestKey"
}
```

<a name="convertApiList"/>

### Convert API List

<a name="rfqQuote"/>

#### RFQ Quote

* Http Request

```
GET /assets/quote
```

| Field        | Type   | Required | Description          | Possible Values |
|--------------|--------|----------|----------------------|-----------------|
| fromCurrency | String | True     | from currency        | BTC,USD ...     |
| toCurrency   | String | True     | to currency          | BTC,USD ...     |
| fromAmountEv | Long   | True     | amountEv to transfer | 100000 ...      |


* Response

```
{
  "code": "string",
  "quoteArgs": {
    "expireAt": 0,
    "origin": 0,
    "price": "string",
    "proceeds": "string",
    "quoteAt": 0,
    "requestAt": 0,
    "ttlMs": 0
  }
}
```


<a name="convert"/>

#### Convert

* Http Request

```
POST /assets/convert
```

| Field        | Type   | Required | Description            | Possible Values |
|--------------|--------|----------|------------------------|-----------------|
| toCurrency   | String | True     | to currency            | BTC,USD ...     |
| fromCurrency | String | True     | from currency          | BTC,USD ...     |
| fromAmountEv | Long   | False    | amountEv to convert    | 100000 ...      |
| code         | String | True     | encrypted convert args | xxxxxxxx ...    |

* Request
```
{
  "code": "string",
  "fromAmountEv": 0,
  "fromCurrency": "string",
  "toCurrency": "string"
}
```

* Response

```
{
  "fromAmountEv": 0,
  "fromCurrency": "string",
  "linkKey": "string",
  "moveOp": 0,
  "status": 0,
  "toAmountEv": 0,
  "toCurrency": "string"
}
```

<a name="convertQuery"/>

#### Query Convert History 

* Http Request

```
GET /assets/convert
```

| Field        | Type     | Required | Description               | Possible Values                      |
|--------------|----------|----------|---------------------------|--------------------------------------|
| fromCurrency | String   | False    | from currency             | BTC,USD ...                          |
| toCurrency   | String   | False    | to currency               | BTC,USD ...                          |
| startTime    | Long     | False    | start time in millisecond | default 2 days ago from the end time |
| endTime      | Long     | False    | end time in millisecond   | default now                          |
| offset       | Integer  | False    | page start from 0         | start from 0, default 0              |
| limit        | Integer  | False    | page size                 | default 20, max 200                  |

* Response

```
[
  {
    "conversionRate": 0,
    "createTime": 0,
    "errorCode": 0,
    "fromAmountEv": 0,
    "fromCurrency": "string",
    "linkKey": "string",
    "status": 0,
    "toAmountEv": 0,
    "toCurrency": "string"
  }
]
```
