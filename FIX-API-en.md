FIX API
=======

# Introduction

FIX (Financial Information eXchange) is a standard electronic messaging protocol which can be used to place orders, receive order updates and executions, and cancel orders. Our FIX API is based on the FIX 4.2 specification and modeled after FIX implementations of other popular cryptocurrency exchanges.

* TestNet FIX endpoint URL: **tcp+ssl://testnet-fixapi.phemex.com:<port>**
* MainNet FIX endpoint URL: **tcp+ssl://fixapi.phemex.com:<port>**

Clients should connect to the endpoint using SSL.
Sequence numbers are started from 1 for each session, Resend request and sequence reset messages are supported.

*Please join our telegram group [Phemex FIX](https://t.me/Phemex_FIX) if you have any questions.*

# Messages
All messages should include the following header:

This documentation uses ; to represent the FIX field separator (byte 0x01). It should be replaced by 0x01 in actual messages.

|Tag | Name           | Required | Description|
|----|----------------|----------|------------|
| 8  | BeginString    | Y        | Must be set to "FIX.4.2"|
| 9  | BodyLength     | Y        | Number of bytes after this field up to and including the delimiter immediately preceding the CheckSum.|
| 35 | MsgType        | Y        | Message type |
| 49 | SenderCompID   | Y        | Comp ID of the party sending the message. It is also used as Client API key (for messages from the client) |
| 56 | TargetCompID   | Y        | Comp ID of the party the message is sent to. Must be set to "PHEMEX" (for messages from the client)|

Messages should also include a sequence number MsgSeqNum (34) and a timestamp SendingTime (52). Sequence numbers start at 1 and must be incremented with every message. Messages with duplicate or out-of-order sequence numbers will be rejected. Sequence number need to be reset if error happens.

# Logon (A)

Sent by the client to initiate a FIX session. Must be the first message sent after a connection is established. Only one session can be established per connection; additional Logon messages are rejected.

## Request

|Tag | Name            | Required | Description|
|----|-----------------|----------|------------|
|35  | MsgType         | Y        | A = Logon  |
|98  | EncryptMethod   | Y        | Method of encryption. Must be set to "0" (None) |
|108 | HeartBInt       | Y        | Session heartbeat interval in seconds. Must be set to "30" |
|96  | RawData         | Y        | Signature (see below) |
|95  | RawDataLength   | Y        | Lengh of the signature, Must be 64 |

## Signature

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

## Response

|Tag | Name            | Required | Description|
|----|-----------------|----------|------------|
| 35 | MsgType         | Y        |            |
| 98 | EncryptMethod   | Y        | Method of encryption. |
| 108| HeartBInt       | Y        | Session heartbeat interval in seconds. |

# HeartBeat (0)
This message can be initiated by both the client and the Phemex FIX gateway with 30 seconds interval.

|Tag | Name            | Required | Description|
|----|-----------------|----------|------------|
| 35 | MsgType         | Y        | 0 = HeartBeat |
| 112| TestReqID       | N        | Required if the heartbeat is a response to a TestRequest (35=1). The value in this field should echo the TestReqID (112) received in the TestRequest. |

# TestRequest (1)
This message can be initiated by both the client and the Phemex FIX gateway.

|Tag | Name            | Required | Description|
|----|-----------------|----------|------------|
| 35 | MsgType         | Y        | 1 = TestRequest |
|112 | TestReqID       | Y        | Identifier included in Test Request message to be returned in resulting Heartbeat. |

# Logout (5)
This message can be initiated by both client and the Phemex FIX gateway.

|Tag | Name            | Required | Description|
|----|-----------------|----------|------------|
| 35 | MsgType         | Y        | 5 = Logout |

# New Order Single (D)
This message is initiated by the client to send a limit or market order.

|Tag | Name            | Required | Description|
|----|-----------------|----------|------------|
| 35 | MsgType         | Y        | D = New Order Single |
| 21 | HandlInst       | Y        | Must be set to "1" (AutomatedExecutionNoIntervention). |
| 11 | ClOrdID         | Y        | Client specified identifier of the order. Must be unique within the session. |
| 55 | Symbol          | Y        | Symbol name. Possible values: BTCUSD, ETHUSD, XRPUSD. |
| 40 | OrdType         | Y        | Order type applicable to the order: 1 = Market, 2 = Limit. |
| 38 | OrderQty        | Y        | Total order quantity of the order. |
| 44 | Price           | N        | Price of order. Required if OrdType (40) = 2 = Limit. |
| 54 | Side            | Y        | Side of the order. Possible values:  1 = Buy, 2 = Sell. |
| 59 | TimeInForce     | Y        | Time qualifier of the order: 1 = Good Till Cancel = GTC, 3 = Immediate or Cancel = IOC, 4 = Fill or Kill = FOK. |
| 18 | ExecInst        | N        | Indicates the order is: E = Reduce only, 6 = Post only,  normal new order if not present. |

If the order is accepted, an ExecutionReport (8) with ExecType=0 (New Ack) will be returned. Otherwise, an ExecutionReport with ExecType=8 (Rejected) will be returned.

# Order Cancel Request (F)
This message is initiated by the client to request to cancel an order.

|Tag | Name            | Required | Description|
|----|-----------------|----------|------------|
| 35 | MsgType         | Y        | F = Order Cancel Request |
| 11 | ClOrdID         | Y        | Client-assigned order ID of the order. |
| 41 | OrigClOrdID     | Y        | Client Order ID of the order being canceled. |
| 54 | Side            | Y        | Side of the order. Possible values:  1 = Buy, 2 = Sell. |
| 55 | Symbol          | Y        | Symbol name. Possible values: BTCUSD, ETHUSD, XRPUSD. |

If the order is successfully cancelled, an ExecutionReport (8) with ExecType=4 (Cancelled) will be returned. Otherwise, an OrderCancelReject (9) will be returned.

# Order Cancel Reject (9)
This message is initiated by the Phemex gateway to notify the client that an OrderCancelRequest (F) failure.

|Tag | Name            | Required | Description|
|----|-----------------|----------|------------|
| 35 | MsgType         | Y        | 9 = Order Cancel Reject|
| 11 | ClOrdID         | Y        | Client identifier as specified in the Cancel request. |
| 41 | OrigClOrdID     | Y        | Original Client specified identifier of the order. |
| 39 | OrdStatus       | Y        | 8 = Rejected|
| 58 | Text            | Y        | Order cancel reject reason description. |
|434 | CxlRejResponseTo| Y        | Indicates whether the message is being generated as an amend reject or a cancel reject: 1 = Order Cancel Request. |

# Execution Report (8)
The Phemex FIX gateway will sned this execution report for New Order (D), Order Cancel (F) requests or any order status report.

|Tag | Name            | Required | Description|
|----|-----------------|----------|------------|
| 35 | MsgType         | Y        | 8 = Execution Report |
| 11 | ClOrdID         | Y        | Client identifier as specified in the Cancel request. |
| 37 | OrderID         | Y        | Order ID as assigned by the exchange. |
| 55 | Symbol          | Y        | Symbol name. Possible values: BTCUSD, ETHUSD, XRPUSD. |
| 54 | Side            | Y        | Side of the order. Possible values:  1 = Buy, 2 = Sell. |
| 38 | OrderQty        | Y        | Total order quantity of the order. |
| 44 | Price           | N        | Price of order. Required if OrdType (40) = 2 = Limit. |
| 150| ExecType        | Y        | Execution Type that indicates the reason for the generation of the Execution Report: 0 = New, 1 = Partially Filled, 2 = Fully Filled, 4 = Canceled, 8 = Rejected. |
| 39 | OrdStatus       | Y        | Order Status after applying the transaction that is being communicated: 0 = New, 1 = Partially Filled, 2 = Fully Filled, 4 = Canceled. |
| 14 | CumQty          | Y        | Cumulative execution size. |
| 151| LeavesQty       | Y        | Open order quantity. |
| 6  | AvgPx           | N        | Calculated average price of all fills on this order. The value would be zero if no executions. Applicable for order status execution report only. |
| 31 | LastPx          | N        | Execution price. Only present if this message was the result of a fill. |
| 32 | LastQty         | N        | Execution size. Only present if this message was the result of a fill. |
| 103| OrdRejReason    | N        | Code to identify reason for order rejection.  5 = Unknown Order if no order found for OrderStatusRequest(H). |
| 893| LastFragment    | N        | Indicates if this message is the last of a fragmented set of messages. |
| 58 | Text            | N        | Free text. |

# Reject (3)
This message is sent by the server in response to an invalid client message.

|Tag | Name            | Required | Description|
|----|-----------------|----------|------------|
| 35 | MsgType         | Y        | 3 = Reject |
| 45 | RefSeqNum       | Y        | Sequence number of the message which caused the rejection. |
| 371| RefTagID        | N        | Tag number of the rejected field. |
| 372| RefMsgType      | N        | Message type of the rejected message. |
| 58 | Text            | N        | Free text. |

# Order Status Request (H)
This message is initiated by the client to request to query order status.

The Phemex FIX gateway will respond with Execution Report (8) message(s) with ExecType (150) = I for all the matched orders.

* If order ID present, the Phemex FIX gateway returns the single order matching the given symbol and order ID. Otherwise it returns all the open orders matching the given symbol.
* If no order found, the Phemex FIX gateway returns an Execution Report (8) message with ExecType (150) = Rejected and OrdRejReason (103) = Unknown order.

*Note conditional order is not supported by this request.*

|Tag | Name            | Required | Description|
|----|-----------------|----------|------------|
| 35 | MsgType         | Y        | H = Order Status Request |
| 11 | ClOrdID         | Y        | Client-assigned order ID of the order. |
| 37 | OrderID         | N        | If OrderID present, query a single order with the given symbol. Otherwise query the list of open orders' with the given symbol. |
| 55 | Symbol          | Y        | Symbol name. Possible values: BTCUSD, ETHUSD, XRPUSD. |

# Order Mass Cancel Request (q)
This message is initiated by the client to request to cancel all open orders. An order mass cancel report will be responded and all the affected orders will be restated with latest state.

*Note conditional order is not supported by this request.*

|Tag | Name            | Required | Description|
|----|-----------------|----------|------------|
| 35 | MsgType         | Y        | q = Order Mass Cancel Request |
| 11 | ClOrdID         | Y        | Client specified identifier of this mass cancel request.|
| 530| MassCancelRequestType | Y  | Specifies the scope of the mass cancel request: 1 = Cancel orders for a Symbol. |
| 55 | Symbol          | Y        | Symbol name. Possible values: BTCUSD, ETHUSD, XRPUSD. |

# Order Mass Cancel Report (r)
This message is sent by the Phemex FIX gateway in response to Order Mass Cancel Request.

|Tag | Name            | Required | Description|
|----|-----------------|----------|------------|
| 35 | MsgType         | Y        | r = Order Mass Cancel Report |
| 11 | ClOrdID         | Y        | Client specified identifier of this mass cancel request.|
| 55 | Symbol          | Y        | Symbol name. Possible values: BTCUSD, ETHUSD, XRPUSD. |
| 530| MassCancelRequestType | Y  | Specifies the scope of the mass cancel request: 1 = Cancel orders for a Symbol. |
| 531| MassCancelResponse    | Y  | Indicates the action taken on the cancel request: 0 = Cancel request rejected, 1 = Cancel orders for a security. |
| 532| MassCancelRejectReason| N  | The code Indicating the reason why the Mass Cancel Request was rejected: 8 = Invalid or Unknown Market Segment(8), 99 = Other. Required if: Mass Cancel Response = Cancel Request Rejected. |
| 533| TotalAffectedOrders	 | Y  | Optional field used to indicate the total number of orders affected by the Order Mass Cancel Request. |
| 58 | Text            | N        | Free text. |

