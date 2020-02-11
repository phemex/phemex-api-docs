| Code | Message | Description  |
|---------|-------------|-------------------|
| 19999 | REQUEST_IS_DUPLICATED | Request Rejected, Details: Duplicated request ID |
| 10001 | OM_DUPLICATE_ORDERID | Order Submission Error, Details: Duplicated order ID |
| 10002 | OM_ORDER_NOT_FOUND | Cancel/Amend Error, Details: Cannot find order ID |
| 10003 | OM_ORDER_PENDING_CANCEL | Request Rejected, Details: Cannot cancel while order is in pending cancel status |
| 10004 | OM_ORDER_PENDING_REPLACE | Request Rejected, Details: Cannot replace while order is in pending cancel status |
| 10005 | OM_ORDER_PENDING | Request Rejected, Details: Cannot process request while order is in pending status |
| 11001 | TE_NO_ENOUGH_AVAILABLE_BALANCE | Order Submission Error, Details: Insufficient available balance |
| 11002 | TE_INVALID_RISK_LIMIT | Failed to set risk limit, Details: Invalid risk limit value |
| 11003 | TE_NO_ENOUGH_BALANCE_FOR_NEW_RISK_LIMIT | Failed to set risk limit, Details: Insufficient available balance |
| 11004 | TE_INVALID_LEVERAGE | Failed to set leverage, Details: invalid input or new leverage is over maximum leverage |
| 11005 | TE_NO_ENOUGH_BALANCE_FOR_NEW_LEVERAGE | Failed to set leverage, Details: Insufficient available balance |
| 11006 | TE_CANNOT_CHANGE_POSITION_MARGIN_WITHOUT_POSITION | Failed to change Margin, Details: Position size is zero. Cannot change margin |
| 11007 | TE_CANNOT_CHANGE_POSITION_MARGIN_FOR_CROSS_MARGIN | Failed to change Margin, Details: Cannot change margin under CrossMargin |
| 11008 | TE_CANNOT_REMOVE_POSITION_MARGIN_MORE_THAN_ADDED | Failed to remove Margin, Details:over the maximum removable Margin |
| 11009 | TE_CANNOT_REMOVE_POSITION_MARGIN_DUE_TO_UNREALIZED_PNL | Failed to remove Margin, Details:over the maximum removable Margin |
| 11010 | TE_CANNOT_ADD_POSITION_MARGIN_DUE_TO_NO_ENOUGH_AVAILABLE_BALANCE | Failed to add Margin, Details: Insufficient available balance |
| 11011 | TE_REDUCE_ONLY_ABORT | ReduceOnly order abort, Details: Cannot accept reduce only order |
| 11012 | TE_REPLACE_TO_INVALID_QTY | Failed to modify order quantity, Details: Order quantity Error |
| 11013 | TE_CONDITIONAL_NO_POSITION | Conditional order failure, Details: Position size is zero. Cannot determine conditional order's quantity |
| 11014 | TE_CONDITIONAL_CLOSE_POSITION_WRONG_SIDE | Conditional order failure, Details: Close position conditional order has the same side |
| 11017 | TE_ADL_CANNOT_FIND_POSITION | ADL request failure, Details: Cannot find requested position on current account |
| 11019 | TE_FUNDING_ALREADY_SETTLED | Funding failure, Details: The current account already pays the funding fee |
| 11020 | TE_CANNOT_TRANSFER_OUT_DUE_TO_BONUS | Withdraw to wallet failure, Details: Withdraw to wallet needs to clean all bonus. However if bonus is used by position or order cost, withdraw fails. |
| 11022 | TE_REJECT_DUE_TO_BANNED | Request rejection, Details: Account is banned |
| 11023 | TE_REJECT_DUE_TO_IN_PROCESS_OF_LIQ | Request rejection, Details: Account is in the process of liquidation |
| 11024 | TE_REJECT_DUE_TO_IN_PROCESS_OF_ADL | Request rejection, Details: Account is in the process of auto-deleverage |
| 11027 | TE_SYMBOL_INVALID | Request rejection, Details: Invalid symbol ID or name |
| 11028 | TE_CURRENCY_INVALID | Request rejection, Details: Invalid currency ID or name |
| 11029 | TE_ACTION_INVALID | Request rejection, Details: Unrecognized request type |
| 11031 | TE_SO_NUM_EXCEEDS | Conditional order request failure, Details: Total conditional order number exceeds the max limit |
| 11032 | TE_AO_NUM_EXCEEDS | Order request failure, Details:Total active order number exceeds the max limit |
| 11033 | TE_ORDER_ID_DUPLICATE | Order request failure, Details:Duplicated order ID |
| 11034 | TE_SIDE_INVALID | Order request failure, Details:Invalid side |
| 11035 | TE_ORD_TYPE_INVALID | Order request failure, Details:Invalid OrderType |
| 11036 | TE_TIME_IN_FORCE_INVALID | Order request failure, Details:Invalid TimeInForce |
| 11037 | TE_EXEC_INST_INVALID | Order request failure, Details:Invalid ExecType |
| 11038 | TE_TRIGGER_INVALID | Conditional order request failure, Details:Invalid trigger type |
| 11039 | TE_STOP_DIRECTION_INVALID | Conditional order request failure, Details:Invalid stop direction type |
| 11043 | TE_RISING_TRIGGER_DIRECTLY | Conditional order request failure, Details: Conditional order would be triggered immediately |
| 11044 | TE_FALLING_TRIGGER_DIRECTLY | Conditional order request failure, Details: Conditional order would be triggered immediately |
| 11045 | TE_TRIGGER_PRICE_TOO_LARGE | Conditional order request failure, Details: Conditional order trigger price is too high |
| 11046 | TE_TRIGGER_PRICE_TOO_SMALL | Conditional order request failure, Details: Conditional order trigger price is too low |
| 11047 | TE_BUY_TP_SHOULD_GT_BASE | Conditional order request failure, Details: TakeProfile BUY conditional order trigger price needs to be greater than reference price |
| 11048 | TE_BUY_SL_SHOULD_LT_BASE | Conditional order request failure, Details: StopLoss BUY condition order price needs to be less than the reference price |
| 11049 | TE_BUY_SL_SHOULD_GT_LIQ | Conditional order request failure, Details: StopLoss BUY condition order price needs to be greater than liquidation price. Otherwise, it won't trigger |
| 11050 | TE_SELL_TP_SHOULD_LT_BASE | Conditional order request failure, Details: TakeProfile SELL conditional order trigger price needs to be less than reference price |
| 11051 | TE_SELL_SL_SHOULD_LT_LIQ | Conditional order request failure, Details: StopLoss SELL condition order price needs to be less than liquidation price. Otherwise, it won't trigger |
| 11052 | TE_SELL_SL_SHOULD_GT_BASE | Conditional order request failure, Details: StopLoss SELL condition order price needs to be greater than the reference price |
| 11054 | TE_PRICE_WORSE_THAN_BANKRUPT | Order request failure, Details: Order price cannot be worse than bankrupt price if this order could reduce position size |
| 11055 | TE_PRICE_TOO_SMALL | Order request failure, Details: Order price is too low |
| 11056 | TE_QTY_TOO_LARGE | Order request failure, Details: Order quantity is too large |
| 11057 | TE_QTY_NOT_MATCH_REDUCE_ONLY | Order request failure, Details: Doesn't allow ReduceOnly order without position |
| 11058 | TE_QTY_TOO_SMALL | Order request failure, Details: Order quantity is too small |
| 11059 | TE_TP_SL_QTY_NOT_MATCH_POS | Conditional order request failure, Details: Position size is zero. Do not accept any TakeProfit or StopLoss order |
| 11060 | TE_SIDE_NOT_CLOSE_POS | Conditional order request failure, Details: TakeProfit or StopLoss order has wrong side. Cannot close position |
| 11061 | TE_ORD_ALREADY_PENDING_CANCEL | Cancel request failure, Details: Repeated cancel request |
| 11062 | TE_ORD_ALREADY_CANCELED | Cancel request failure, Details: Order is already canceled |
| 11063 | TE_ORD_STATUS_CANNOT_CANCEL | Cancel request failure, Details: Order is not able to be canceled under current status |
| 11064 | TE_ORD_ALREADY_PENDING_REPLACE | Replace request failure, Details: Replace request is rejected because order is already in pending replace status |
| 11065 | TE_ORD_REPLACE_NOT_MODIFIED | Replace request failure, Details: Replace request doesn't modify any parameters of the order |
| 11066 | TE_ORD_STATUS_CANNOT_REPLACE | Replace request failure, Details: Order is not able to be replaced under current status |
| 11067 | TE_CANNOT_REPLACE_PRICE | Conditional order request failure, Details: Market conditional order cannot change price |
| 11068 | TE_CANNOT_REPLACE_QTY | Conditional order request failure, Details: Condtional order for close position cannot change order quantity, since the order quantity is determined by position size already |
| 11069 | TE_ACCOUNT_NOT_IN_RANGE | Request rejection, Details: The account ID in the request is not valid or is not in the range of current process |
| 11070 | TE_SYMBOL_NOT_IN_RANGE | Request rejection, Details: The symbol is invalid |
| 11072 | TE_TKFR_NOT_IN_RANGE | Failed to change taker fee, Details: The fee value is not valid |
| 11073 | TE_MKFR_NOT_IN_RANGE | Failed to change maker fee, Details: The fee value is not valid |
| 11074 | TE_CANNOT_ATTACH_TP_SL | Order request failure, Details: Order request cannot contain TP/SL parameters when the account already has positions |
| 11075 | TE_TP_TOO_LARGE | Order request failure, Details: TakeProfit price is too large |
| 11076 | TE_TP_TOO_SMALL | Order request failure, Details: TakeProfit price is too small |
| 11077 | TE_TP_TRIGGER_INVALID | Order request failure, Details: Invalid trigger type |
| 11078 | TE_SL_TOO_LARGE | Order request failure, Details: StopLoss price is too large |
| 11079 | TE_SL_TOO_SMALL | Order request failure, Details: StopLoss price is too small |
| 11080 | TE_SL_TRIGGER_INVALID | Order request failure, Details: Invalid trigger type |
| 11081 | TE_RISK_LIMIT_EXCEEDS | Order request failure, Details: Total potential position breaches current risk limit |
| 11082 | TE_CANNOT_COVER_ESTIMATE_ORDER_LOSS | Order request failure, Details: The remaining balance cannot cover the potential unrealized PnL for this new order |
| 11083 | TE_TAKE_PROFIT_ORDER_DUPLICATED | Order request failure, Details: TakeProfit order already exists |
| 11084 | TE_STOP_LOSS_ORDER_DUPLICATED | Order request failure, Details: StopLoss order already exists |
| 11085 | TE_CL_ORD_ID_DUPLICATE | Order request failure, clOrderID duplicated |
