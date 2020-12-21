## Table of Contents

* [Rate Limits](#ratelimit)
   * [IP RateLimit](#ipratelimit)
   * [API RateLimit Rules](#apiratelimitrule)
   * [API Groups](#apigroup)


<a name="ratelimit"/>

## Rate Limits

   * In order to protect the exchange, Phemex apply `API RateLimit` and `IP Ratelimit` on all requests.

   * Rest API has a request capacity in one minute window on user basis.

   * IP has request capacity in 5 minutes window.

   * Ratelimit of API is ***independant*** of that in WEB/APP, so if one get ratelimited in API, one can place/cancel orders via WEB or APP.

<a name="ipratelimit"/>

### IP Ratelimit

   Currently Phemex restrict every IP 5,000 request in 5 minutes window.

   If exceeded this IP capacity, the user would be blocked in the following 5 minutes.

<a name="apiratelimitrule"/>

### API RateLimit Rules
   
   * All Phemex APIs are divided into 3 groups, `contract`, `spotOrder` and `others`.
   * All APIs in the same group share a request capacity.
   * Every API consume request shares at its own weight.
   * If exceeded the ratelimit, http status 429 will returned, together with a reset seconds header `x-ratelimit-retry-after-<groupName>`

   * Ratelimit headers
   Below headers are returned with API response. Postfix `-<groupName>` is empty if group is others

   | Header name  | Description |
   |--------------|-------------|
   | x-ratelimit-remaining-*groupName*   | Remaining request permits in this minute |
   | x-ratelimit-capacity-*groupName*    | Request ratelimit capacity |
   | x-ratelimit-retry-after-*groupName* | Reset timeout in seconds for current ratelimited user |

   * Group capacity

   | Group Name | Capacity |
   |------------|---------|
   | Contract   |  500/minutes |
   | SpotOrder  | trail, premium 500/minutes |
   | Others     | 100/minutes  |


<a name="apigroup"/>

### Api Groups

<a name="contractAPIGroup"/>
   * Contract group

   Contract group is for contract trading, it contains following api.

| Path | Method | Weight | Description |
|------|--------|--------|-------------|
| /orders | POST | 1 | Place new order  |
| /orders/replace | PUT | 1 | Amend order |
| /orders/cancel | DELETE | 1 | Cancel order |
| /orders/all | DELETE | 3 | Cancel all order by symbol |
| /orders | DELETE | 1 | cancel orders |
| /orders/activeList | GET | 1 | Query open orders by symbol |
| /orders/active | GET | 1 | Query open order by orderID  |
| /accounts/accountPositions | GET | 1 | Query account & position by currency |
| /accounts/positions | GET | 25 | Query positions with un-realized-pnl |

   * SpotOrder group

<a name="spotAPIGroup"/>
   SpotOrder group is for spot trading, which contains following api.

| Path | Method | Weight | Description |
|------|--------|--------|-------------|
| /spot/orders | POST | 1 | Place spot order |
| /spot/orders | PUT | 1 | Amend spot order |
| /spot/orders | DELETE | 2 | Cancel spot order  |
| /spot/orders/all | DELETE | 2 | Cancel spot orders by symbol |
| /spot/orders/active | GET | 1 | Query open spot order |
| /spot/orders | GET | 1 | Query all open spot orders by symbol |

   * Others

   APIs which is not in contract or spotOrder group is categoried into other group.


