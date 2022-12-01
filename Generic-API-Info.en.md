## Table of Contents

* [Rate Limits](#ratelimit)
    * [IP RateLimit](#ipratelimit)
    * [API RateLimit Rules](#apiratelimitrule)
    * [API Groups](#apigroup)
    * [New Contract API RateLimit Rules](#newcontractapiratelimitrule)

<a name="ratelimit"/>

## Rate Limits

* In order to prevent API abuse, Phemex applies `API RateLimit` and `IP Ratelimit` on all requests.

* Rest API has a request capacity in one-minute window on user basis.

* IP has request capacity in 5-minute window.

* Ratelimit of API is ***independant*** of that in WEB/APP, so if one gets rate limited in API, one can still place/cancel
  orders via WEB or APP.

<a name="ipratelimit"/>

### IP Ratelimit

Currently Phemex restrict every IP 5,000 requests in 5-minute window.

If the rate limit has been breached, user request would be blocked in the following 5 minutes.

<a name="apiratelimitrule"/>

### API RateLimit Rules

* All Phemex APIs are divided into 3 groups, `contract`, `spotOrder` and `others`.
* All APIs in the same group share the same total group capacity.
* Every API consume its own corresponding weight.
* If the ratelimit is violated, http status 429 will be returned, together with a reset time in seconds
  header `x-ratelimit-retry-after-<groupName>`

* Ratelimit headers
  Below headers are returned with API response. Postfix `-<groupName>` is empty if the group is others

| Header name  | Description |
|--------------|-------------|
| x-ratelimit-remaining-*groupName*   | Remaining request permits in this minute |
| x-ratelimit-capacity-*groupName*    | Request ratelimit capacity |
| x-ratelimit-retry-after-*groupName* | Reset timeout in seconds for current ratelimited user |

* Group capacity

| Group Name | Capacity |
|------------|---------|
| Contract   |  500/minute |
| SpotOrder  |  500/minute |
| Others     |  100/minute |

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

<a name="otherGroup"/>

APIs which is not in contract or spotOrder group are categorized into other group.

As the number of interfaces in Other group may grow over time, the list below is not exhausted, the items in the
table highlight interfaces that has weight other than 1, or interfaces that Phemex deems important.

| Path | Method | Weight | Description  |
|------|--------|--------|--------------|
| /exchange/public/md/kline | GET | 10     | kline query  |


<a name="newcontractapiratelimitrule"/>

### New Contract API RateLimit Rules (for VAPI/VIP only)

* Contract API currently employs new rate-limit rules based on symbols, according to Phemex internal configuration of uid individually.
* Under the new throttling rules, Contract API consumes both contract group capacity (5000/minute) and symbol group capacity (500/minute) at the same time, but the each capacity of different symbols are independent from one another.
* Contract API that may involve all symbols like 'cancel all' consumes request capacity under a special group named 'CONTACT_ALL_SYM'.

* Ratelimit headers added in the symbol dimension. 

| Header name                                     | Description                                                               |
|-------------------------------------------------|---------------------------------------------------------------------------|
| x-ratelimit-remaining-*groupName_symbol*        | Remaining request ratelimit in the symbol group in this minute              |
| x-ratelimit-capacity-*groupName_symbol*         | Total request capacity in the symbol group                            |
| x-ratelimit-retry-after-*groupName_symbol*      | Reset timeout in seconds for current ratelimited user in the symbol group |

* Special Contract Group Capacity

| Group Name      | Capacity    |
|-----------------|-------------|
| Contract        | 5000/minute |
| Contract_SYMBOL | 500/minute  |
| CONTACT_ALL_SYM | 500/minute  |


