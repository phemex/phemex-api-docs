# Phemex FIX API

=======
# Introduction
FIX (Financial Information eXchange) is a standard electronic messaging protocol which can be used to place orders, receive order updates and executions, and cancel orders. Our FIX api is based on the FIX 4.2 specification and modeled after FIX implementations of other popular cryptocurrency exchanges.
FIX endpoint URL: *tcp+ssl://fix.phemex.com:port*
Clients should connect to the endpoint using SSL.
Sequence numbers are started from 1 for each session, Resend request and sequence reset messages are supported.
# Messages
All messages should include the following header:
This documentation uses ; to represent the FIX field separator (byte 0x01). It should be replaced by 0x01 in actual messages.
|Tag | Name | Example | Description|
|----|------|---------|------------|
|8|BeginString|FIX4.2|Must be set to "FIX.4.2"|
|9|BodyLength|162|Length of the message body in bytes|
|35|MsgType|8|Message type|
|49|SenderCompID|sjuzhsnewpz-sdfncs|Client API key (for messages from the client)|
|56|TargetCompID|PHEMEX|Must be set to "PHEMEX" (for messages from the client)|
Messages should also include a sequence number MsgSeqNum (34) and a timestamp SendingTime (52). Sequence numbers start at 1 and must be incremented with every message. Messages with duplicate or out-of-order sequence numbers will be rejected. Sequence number need to be reset if error happens.
# Logon (A)
Sent by the client to initiate a FIX session. Must be the first message sent after a connection is established. Only one session can be established per connection; additional Logon messages are rejected.
## request
|Tag | Name | Example | Description|
|----|------|---------|------------|
|35|MsgType|A||
|98|EncryptMethod|0|Must be set to "0" (None)|
|108|HeartBInt|30| Must be set to "30"|
|96|RawData|a8ef...3172|Signature (see below)|
|95|RawDataLength|64|Lengh of the RawData, Must be 64|
## signature
For security, the Logon message must be signed by the client. To compute the signature, concatenate the following fields, joined by the FIX field separator (byte 0x01), and compute the SHA256 HMAC using the API secret:
* SendingTime (52)
* MsgType (35)
* MsgSeqNum (34)
* SenderCompID (49)
* TargetCompID (56)
The resulting hash should be hex-encoded.
    from datetime import datetime
    import hmac
    api_key = 'YOUR_API_KEY'
    api_secret = 'YOUR_API_SECRET'
    sending_time = datetime.now().strftime('%Y%m%d-%H:%M:%S')
    sign_target = '\x01'.join([
        sending_time,  # SendingTime
        'A',  # MsgType
        '1',  # MsgSeqNum
        api_key,  # SenderCompID
        'PHEMEX',  # TargetCompID
    ])
    signature = hmac.new(api_secret.encode(), sign_target.encode(), 'sha256').hexdigest()
## response
|Tag | Name | Value |
|----|------|-------|
|35| MsgType|A|
|98| EncryptMethod|0|
|108|HeartBInt|30|
# HeartBeat (0)
Sent by either side if a message has not been received in the past 30 seconds. Should also be sent in response to a TestRequest (1).
|Tag | Name | Value | Description|
|----|------|---------|------------|
|35| MsgType|0|
|112|TestReqID|123|If this heartbeat is in response to a TestRequest, copied from the TestRequest.|
# TestRequest (1)
May be sent by either side at any time.
|Tag | Name | Value | Description|
|----|------|---------|------------|
|35|MsgType|1||
|112|TestReqID|123|Arbitrary string, to be echoed back by a Heartbeat.|
# Logout (5)
Sent by either side to terminate the session. The other side should respond with another Logout message to acknowledge session termination. The connection will be closed afterwards.
|Tag | Name | Value |
|----|------|-------|
|35| MsgType|5|
# New Order Single (D)
Sent by the client to submit a new order. Only limit orders are currently supported by the FIX API.
|Tag | Name | Value | Description|
|----|------|---------|------------|
|35|MsgType| D |
|21|HandlInst | 1 | Must be set to "1" (AutomatedExecutionNoIntervention)|
|11|ClOrdID | order123 | Arbituary client-selected string to identify the order; must be unique within the session|
|55|Symbol | BTCUSD |  Symbol name|
|40|OrdType | 2 | Must be set to "2" (Limit) or "1" (Market)|
|38|OrderQty |100 | Order size in base units|
|44|Price | 8000 | Limit price|
|54|Side | 1 | "1": buy; "2": sell |
|59|TimeInForce | 1 | Must be set to "1" (Good Till Cancel) or "3" (Immediate or Cancel) or "4" (Fill or Kill)|
|18|ExecInst|E|This paramter is optional. "E": reduce only, "6": post only, not supplied: standard|
If the order is accepted, an ExecutionReport (8) with ExecType=0 (New Ack) will be returned. Otherwise, an ExecutionReport with ExecType=8 (Rejected) will be returned.
# Order Cancel Request (F)
Sent by the client to request to cancel an order.
|Tag | Name | Value | Description|
|----|------|---------|------------|
|35|MsgType|F|
|41|OrigClOrdID|order123|Client-assigned order ID of the order|
If the order is successfully cancelled, an ExecutionReport (8) with ExecType=4 (Cancelled) will be returned. Otherwise, an OrderCancelReject (9) will be returned.
# Order Cancel Reject (9)
Sent by the server to notify the client that an OrderCancelRequest (F) failed.
|Tag | Name | Value | Description|
|----|------|---------|------------|
|35|MsgType|9||
|11|ClOrdID|cancel123|Copied from OrderCancelRequest|
|41|OrigClOrdID|order123|Copied from OrderCancelRequest|
|39|OrdStatus|8||
|58|CxlRejReason|OrderNotFound||
|434|CxlRejResponseTo|1|Always set to "1"|
# Execution Report (8)
Sent by the server whenever an order receives a fill, whenever the status of an order changes, or in response to a NewOrderSingle (D) or OrderCancelRequest (F) message from the client.
|Tag | Name | Value | Description|
|----|------|---------|------------|
| 35 | MsgType | 8 | |
| 11 | ClOrdID | order123 | Client-selected order ID.|
| 37 | OrderID | 123456 | Server-assigned order ID. |
| 55 | Symbol  | BTCUSD | Symbol name |
| 54 | Side  | 1  | "1": buy; "2": sell |
| 38 | OrderQty | 1.2 | Original order quantity|
| 44 | Price | 8000 | Original order price |
| 150| ExecType | 1 | Reason for this message (see below)|
| 39| OrdStatus | 0 | Order status (see below)|
| 14| CumQty | 0.4 | Quantity of order that has already been filled|
| 151 | LeavesQty | 0.8 | Quantity of order that is still open|
| 31 | LastPx | 7999.25 | Fill price. Only present if this message was the result of a fill|
| 32 | LastQty | 0.4 | Fill quantity. Only present if this message was the result of a fill|
| 58 |Text | 58 | Description of the reason the order was rejected or unsolicated cancelled|
## ExecType values
The ExecType (150) field indicates the reason why this ExecutionReport was sent.
|ExecType|Description|
|--------|-----------|
|0 | New order|
|1 | Partially fill for order|
|2 | Fully fill for order |
|4 | Order cancelled |
|8 | Response to a rejected NewOrderSingle (D) request|
## OrdStatus values
|OrdStatus|Description|
|---------|-----------|
|0 | New order|
|1 |Partially filled order|
|2 | Fully filled order|
|4 | Cancelled order|
# Reject (3)
Sent by the server in response to an invalid message.
|Tag | Name | Value | Description|
|----|------|---------|------------|
|35| MsgType| 3|
|45 |RefSeqNum| 2| Sequence number of the rejected message|
|371| RefTagID | 38 | Tag number of the rejected field|
|372| RefMsgType | D| Message type of the rejected message|
|58|Text|Missing quantity|Human-readable description of the reason for the rejection|
