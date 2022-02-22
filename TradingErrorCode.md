## BizError Field
  |  code   |   Message   |   Description   |  
  |---------|-------------|-----------------|  
  |  19999  |  REQUEST_IS_DUPLICATED  |  Details: Duplicated request ID  |  
  |  10001  |  OM_DUPLICATE_ORDERID  |  Details: Duplicated order ID  |  
  |  10002  |  OM_ORDER_NOT_FOUND  |  Details: Cannot find order ID  |  
  |  10003  |  OM_ORDER_PENDING_CANCEL  |  Details: Cannot cancel while order is already in pending cancel status  |  
  |  10004  |  OM_ORDER_PENDING_REPLACE  |  Details: Cannot cancel while order is already in pending cancel status  |  
  |  10005  |  OM_ORDER_PENDING  |  Details: Cannot cancel while order is already in pending cancel status  |  
  |  11001  |  TE_NO_ENOUGH_AVAILABLE_BALANCE  |  Details: Insufficient available balance  |  
  |  11002  |  TE_INVALID_RISK_LIMIT  |  Details: Invalid risk limit value  |  
  |  11003  |  TE_NO_ENOUGH_BALANCE_FOR_NEW_RISK_LIMIT  |  Details: Insufficient available balance  |  
  |  11004  |  TE_INVALID_LEVERAGE  |  Details: invalid input or new leverage is over maximum allowed leverage  |  
  |  11005  |  TE_NO_ENOUGH_BALANCE_FOR_NEW_LEVERAGE  |  Details: Insufficient available balance  |  
  |  11006  |  TE_CANNOT_CHANGE_POSITION_MARGIN_WITHOUT_POSITION  |  Details: Position size is zero. Cannot change margin  |  
  |  11007  |  TE_CANNOT_CHANGE_POSITION_MARGIN_FOR_CROSS_MARGIN  |  Details: Cannot change margin under CrossMargin  |  
  |  11008  |  TE_CANNOT_REMOVE_POSITION_MARGIN_MORE_THAN_ADDED  |  Details: exceeds the maximum removable Margin  |  
  |  11009  |  TE_CANNOT_REMOVE_POSITION_MARGIN_DUE_TO_UNREALIZED_PNL  |  Details: exceeds the maximum removable Margin  |  
  |  11010  |  TE_CANNOT_ADD_POSITION_MARGIN_DUE_TO_NO_ENOUGH_AVAILABLE_BALANCE  |  Details: Insufficient available balance  |  
  |  11011  |  TE_REDUCE_ONLY_ABORT  |  Details: Cannot accept reduce only order  |  
  |  11012  |  TE_REPLACE_TO_INVALID_QTY  |  Details: Order quantity Error  |  
  |  11013  |  TE_CONDITIONAL_NO_POSITION  |  Details: Position size is zero. Cannot determine conditional order's quantity  |  
  |  11014  |  TE_CONDITIONAL_CLOSE_POSITION_WRONG_SIDE  |  Details: Close position conditional order has the same side  |  
  |  11015  |  TE_CONDITIONAL_TRIGGERED_OR_CANCELED  |    |  
  |  11016  |  TE_ADL_NOT_TRADING_REQUESTED_ACCOUNT  |  Details: Request is routed to the wrong trading engine  |  
  |  11017  |  TE_ADL_CANNOT_FIND_POSITION  |  Details: Cannot find requested position on current account  |  
  |  11018  |  TE_NO_NEED_TO_SETTLE_FUNDING  |  Details: The current account does not need to pay a funding fee  |  
  |  11019  |  TE_FUNDING_ALREADY_SETTLED  |  Details: The current account already pays the funding fee  |  
  |  11020  |  TE_CANNOT_TRANSFER_OUT_DUE_TO_BONUS = 11020;  |  Details: Withdraw to wallet needs to remove all remaining bonus. However if bonus is used by position or order cost, withdraw fails.  |  
  |  11021  |  TE_INVALID_BONOUS_AMOUNT = 11021; // Grpc command cannot be negative number  |  Details: Invalid bonus amount  |  
  |  11022  |  TE_REJECT_DUE_TO_BANNED = 11022;  |  Details: Account is banned  |  
  |  11023  |  TE_REJECT_DUE_TO_IN_PROCESS_OF_LIQ = 11023;  |  Details: Account is in the process of liquidation  |  
  |  11024  |  TE_REJECT_DUE_TO_IN_PROCESS_OF_ADL = 11024;  |  Details: Account is in the process of auto-deleverage  |  
  |  11025  |  TE_ROUTE_ERROR = 11025;  |  Details: Request is routed to the wrong trading engine  |  
  |  11026  |  TE_UID_ACCOUNT_MISMATCH = 11026;  |    | 
  |  11027  |  TE_SYMBOL_INVALID = 11027;  |  Details: Invalid number ID or name  |  
  |  11028  |  TE_CURRENCY_INVALID = 11028;  |  Details: Invalid currency ID or name  |  
  |  11029  |  TE_ACTION_INVALID = 11029;  |  Details: Unrecognized request type  |  
  |  11030  |  TE_ACTION_BY_INVALID = 11030;  |    |  
  |  11031  |  TE_SO_NUM_EXCEEDS = 11031;  |  Details: Number of total conditional orders exceeds the max limit  |  
  |  11032  |  TE_AO_NUM_EXCEEDS = 11032;  |  Details: Number of total active orders exceeds the max limit  |  
  |  11033  |  TE_ORDER_ID_DUPLICATE = 11033;  |  Details:Duplicated order ID  |  
  |  11034  |  TE_SIDE_INVALID = 11034;  |  Details:Invalid side  |  
  |  11035  |  TE_ORD_TYPE_INVALID = 11035;  |  Details:Invalid OrderType  |  
  |  11036  |  TE_TIME_IN_FORCE_INVALID = 11036;  |  Details:Invalid TimeInForce  |  
  |  11037  |  TE_EXEC_INST_INVALID = 11037;  |  Details:Invalid ExecType  |  
  |  11038  |  TE_TRIGGER_INVALID = 11038;  |  Details:Invalid trigger type  |  
  |  11039  |  TE_STOP_DIRECTION_INVALID = 11039;  |  Details:Invalid stop direction type  |  
  |  11040  |  TE_NO_MARK_PRICE = 11040;  |  Details: Cannot get valid mark price to create conditional order  |  
  |  11041  |  TE_NO_INDEX_PRICE = 11041;  |  Details: Cannot get valid index price to create conditional order  |  
  |  11042  |  TE_NO_LAST_PRICE = 11042;  |  Details: Cannot get valid last market price to create conditional order  |  
  |  11043  |  TE_RISING_TRIGGER_DIRECTLY = 11043;  |  Details: Conditional order would be triggered immediately  |  
  |  11044  |  TE_FALLING_TRIGGER_DIRECTLY = 11044;  |  Details: Conditional order would be triggered immediately  |  
  |  11045  |  TE_TRIGGER_PRICE_TOO_LARGE = 11045;  |  Details: Conditional order trigger price is too high  |  
  |  11046  |  TE_TRIGGER_PRICE_TOO_SMALL = 11046;  |  Details: Conditional order trigger price is too low  |  
  |  11047  |  TE_BUY_TP_SHOULD_GT_BASE = 11047;  |  Details: TakeProfit BUY conditional order trigger price needs to be greater than reference price  |  
  |  11048  |  TE_BUY_SL_SHOULD_LT_BASE = 11048;  |  Details: StopLoss BUY condition order price needs to be less than the reference price  |  
  |  11049  |  TE_BUY_SL_SHOULD_GT_LIQ = 11049;  |  Details: StopLoss BUY condition order price needs to be greater than liquidation price or it will not trigger  |  
  |  11050  |  TE_SELL_TP_SHOULD_LT_BASE = 11050;  |  Details: TakeProfit SELL conditional order trigger price needs to be less than reference price  |  
  |  11051  |  TE_SELL_SL_SHOULD_LT_LIQ = 11051;  |  Details: StopLoss SELL condition order price needs to be less than liquidation price or it will not trigger  |  
  |  11052  |  TE_SELL_SL_SHOULD_GT_BASE = 11052;  |  Details: StopLoss SELL condition order price needs to be greater than the reference price  |  
  |  11053  |  TE_PRICE_TOO_LARGE = 11053;  |  Details: Order price is too large  |  
  |  11054  |  TE_PRICE_WORSE_THAN_BANKRUPT = 11054;  |  Details: Order price cannot be more aggressive than bankrupt price if this order has instruction to close a position  |  
  |  11055  |  TE_PRICE_TOO_SMALL = 11055;  |  Details: Order price is too low  |  
  |  11056  |  TE_QTY_TOO_LARGE = 11056;  |  Details: Order quantity is too large  |  
  |  11057  |  TE_QTY_NOT_MATCH_REDUCE_ONLY = 11057;  |  Details: Does not allow ReduceOnly order without position  |  
  |  11058  |  TE_QTY_TOO_SMALL = 11058;  |  Details: Order quantity is too small  |  
  |  11059  |  TE_TP_SL_QTY_NOT_MATCH_POS = 11059;  |  Details: Position size is zero. Cannot accept any TakeProfit or StopLoss order  |  
  |  11060  |  TE_SIDE_NOT_CLOSE_POS = 11060;  |  Details: TakeProfit or StopLoss order has wrong side. Cannot close position  |  
  |  11061  |  TE_ORD_ALREADY_PENDING_CANCEL = 11061;  |  Details: Repeated cancel request  |  
  |  11062  |  TE_ORD_ALREADY_CANCELED = 11062;  |  Details: Order is already canceled  |  
  |  11063  |  TE_ORD_STATUS_CANNOT_CANCEL = 11063;  |  Details: Order is not able to be canceled under current status  |  
  |  11064  |  TE_ORD_ALREADY_PENDING_REPLACE = 11064;  |  Details: Replace request is rejected because order is already in pending replace status  |  
  |  11065  |  TE_ORD_REPLACE_NOT_MODIFIED = 11065;  |  Details: Replace request does not modify any parameters of the order  |  
  |  11066  |  TE_ORD_STATUS_CANNOT_REPLACE = 11066;  |  Details: Order is not able to be replaced under current status  |  
  |  11067  |  TE_CANNOT_REPLACE_PRICE = 11067;  |  Details: Market conditional order cannot change price  |  
  |  11068  |  TE_CANNOT_REPLACE_QTY = 11068;  |  Details: Condtional order for closing position cannot change order quantity, since the order quantity is determined by position size already  |  
  |  11069  |  TE_ACCOUNT_NOT_IN_RANGE = 11069;  |  Details: The account ID in the request is not valid or is not in the range of the current process  |  
  |  11070  |  TE_SYMBOL_NOT_IN_RANGE = 11070;  |  Details: The symbol is invalid  |  
  |  11071  |  TE_ORD_STATUS_CANNOT_TRIGGER = 11071;  |    |  
  |  11072  |  TE_TKFR_NOT_IN_RANGE = 11072;  |  Details: The fee value is not valid  |  
  |  11073  |  TE_MKFR_NOT_IN_RANGE = 11073;  |  Details: The fee value is not valid  |  
  |  11074  |  TE_CANNOT_ATTACH_TP_SL = 11074;  |  Details: Order request cannot contain TP/SL parameters when the account already has positions  |  
  |  11075  |  TE_TP_TOO_LARGE = 11075;  |  Details: TakeProfit price is too large  |  
  |  11076  |  TE_TP_TOO_SMALL = 11076;  |  Details: TakeProfit price is too small  |  
  |  11077  |  TE_TP_TRIGGER_INVALID = 11077;  |  Details: Invalid trigger type  |  
  |  11078  |  TE_SL_TOO_LARGE = 11078;  |  Details: StopLoss price is too large  |  
  |  11079  |  TE_SL_TOO_SMALL = 11079;  |  Details: StopLoss price is too small  |  
  |  11080  |  TE_SL_TRIGGER_INVALID = 11080;  |  Details: Invalid trigger type  |  
  |  11081  |  TE_RISK_LIMIT_EXCEEDS = 11081;  |  Details: Total potential position breaches current risk limit  |  
  |  11082  |  TE_CANNOT_COVER_ESTIMATE_ORDER_LOSS = 11082;  |  Details: The remaining balance cannot cover the potential unrealized PnL for this new order  |  
  |  11083  |  TE_TAKE_PROFIT_ORDER_DUPLICATED = 11083;  |  Details: TakeProfit order already exists  |  
  |  11084  |  TE_STOP_LOSS_ORDER_DUPLICATED = 11084;  |  Details: StopLoss order already exists  |  
  |  11085  |  TE_CL_ORD_ID_DUPLICATE  |  Details: ClOrdId is duplicated  |  
  |  11086  |  TE_PEG_PRICE_TYPE_INVALID  |  Details: PegPriceType is invalid  |  
  |  11087  |  TE_BUY_TS_SHOULD_LT_BASE  |  Details: The trailing order's StopPrice should be less than the current last price  |  
  |  11088  |  TE_BUY_TS_SHOULD_GT_LIQ  |  Details: The traling order's StopPrice should be greater than the current liquidation price  |  
  |  11089  |  TE_SELL_TS_SHOULD_LT_LIQ  |  Details: The traling order's StopPrice should be greater than the current last price  |  
  |  11090  |  TE_SELL_TS_SHOULD_GT_BASE  |  Details: The traling order's StopPrice should be less than the current liquidation price  |  
  |  11091  |  TE_BUY_REVERT_VALUE_SHOULD_LT_ZERO  |  Details: The PegOffset should be less than zero  |  
  |  11092  |  TE_SELL_REVERT_VALUE_SHOULD_GT_ZERO  |  Details: The PegOffset should be greater than zero  |  
  |  11093  |  TE_BUY_TTP_SHOULD_ACTIVATE_ABOVE_BASE  |  Details: The activation price should be greater than the current last price  |  
  |  11094  |  TE_SELL_TTP_SHOULD_ACTIVATE_BELOW_BASE  |  Details: The activation price should be less than the current last price  |  
  |  11095  |  TE_TRAILING_ORDER_DUPLICATED  |  Details: A trailing order exists already  |  
  |  11096  |  TE_CLOSE_ORDER_CANNOT_ATTACH_TP_SL  |  Details: An order to close position cannot have trailing instruction  |  
  |  11097  |  TE_CANNOT_FIND_WALLET_OF_THIS_CURRENCY  |  Details: This crypto is not supported  |  
  |  11098  |  TE_WALLET_INVALID_ACTION  |  Details: Invalid action on wallet  |  
  |  11099  |  TE_WALLET_VID_UNMATCHED  |  Details: Wallet operation request has a wrong wallet vid  |  
  |  11100  |  TE_WALLET_INSUFFICIENT_BALANCE  |  Details: Wallet has insufficient balance  |  
  |  11101  |  TE_WALLET_INSUFFICIENT_LOCKED_BALANCE  |  Details: Locked balance in wallet is not enough for unlock/withdraw request  |  
  |  11102  |  TE_WALLET_INVALID_DEPOSIT_AMOUNT  |  Details: Deposit amount must be greater than zero  |  
  |  11103  |  TE_WALLET_INVALID_WITHDRAW_AMOUNT  |  Details: Withdraw amount must be less than zero  |  
  |  11104  |  TE_WALLET_REACHED_MAX_AMOUNT  |  Details: Deposit makes wallet exceed max amount allowed  |  
  |  11105  |  TE_PLACE_ORDER_INSUFFICIENT_BASE_BALANCE  |  Details: Insufficient funds in base wallet  |  
  |  11106  |  TE_PLACE_ORDER_INSUFFICIENT_QUOTE_BALANCE  |  Details: Insufficient funds in quote wallet  |  
  |  11107  |  TE_CANNOT_CONNECT_TO_REQUEST_SEQ  |  Details: TradingEngine failed to connect with CrossEngine  |  
  |  11108  |  TE_CANNOT_REPLACE_OR_CANCEL_MARKET_ORDER  |  Details: Cannot replace/amend market order  |  
  |  11109  |  TE_CANNOT_REPLACE_OR_CANCEL_IOC_ORDER  |  Details: Cannot replace/amend ImmediateOrCancel order  |  
  |  11110  |  TE_CANNOT_REPLACE_OR_CANCEL_FOK_ORDER  |  Details: Cannot replace/amend FillOrKill order  |  
  |  11111  |  TE_MISSING_ORDER_ID  |  Details: OrderId is missing  |  
  |  11112  |  TE_QTY_TYPE_INVALID  |  Details: QtyType is invalid  |  
  |  11113  |  TE_USER_ID_INVALID  |  Details: UserId is invalid  |  
  |  11114  |  TE_ORDER_VALUE_TOO_LARGE  |  Details: Order value is too large  |  
  |  11115  |  TE_ORDER_VALUE_TOO_SMALL  |  Details: Order value is too small  |  
  |  11116  |  TE_BO_NUM_EXCEEDS| Details: the total count of brakcet orders should equal or less than 5 |
  |  11117  |  TE_BO_CANNOT_HAVE_BO_WITH_DIFF_SIDE| Details: all bracket orders should have the same Side.|
  |  11118  |  TE_BO_TP_PRICE_INVALID| Details: bracker order take profit price is invalid|
  |  11119  |  TE_BO_SL_PRICE_INVALID| Details: bracker order stop loss price is invalid|
  |  11120  |  TE_BO_SL_TRIGGER_PRICE_INVALID| Details: bracker order stop loss trigger price is invalid|
  |  11121  |  TE_BO_CANNOT_REPLACE| Details: cannot replace bracket order.|
  |  11122  |  TE_BO_BOTP_STATUS_INVALID| Details: bracket take profit order status is invalid |
  |  11123  |  TE_BO_CANNOT_PLACE_BOTP_OR_BOSL_ORDER| Details: cannot place bracket take profit order |
  |  11124  |  TE_BO_CANNOT_REPLACE_BOTP_OR_BOSL_ORDER| Details: cannot place bracket stop loss order|
  |  11125  |  TE_BO_CANNOT_CANCEL_BOTP_OR_BOSL_ORDER| Details: cannot cancel bracket sl/tp order|
  |  11126  |  TE_BO_DONOT_SUPPORT_API| Details: doesn't support bracket order via API|
  |  11128  |  TE_BO_INVALID_EXECINST| Details: ExecInst value is invalid|
  |  11129  |  TE_BO_MUST_BE_SAME_SIDE_AS_POS| Details: bracket order should have the same side as position's side|
  |  11130  |  TE_BO_WRONG_SL_TRIGGER_TYPE| Details: bracket stop loss order trigger type is invalid|
  |  11131  |  TE_BO_WRONG_TP_TRIGGER_TYPE| Details: bracket take profit order trigger type is invalid|
  |  11132  |  TE_BO_ABORT_BOSL_DUE_BOTP_CREATE_FAILED| Details: cancel bracket stop loss order due failed to create take profit order.|
  |  11133  |  TE_BO_ABORT_BOSL_DUE_BOPO_CANCELED| Details: cancel bracket stop loss order due main order canceled.|
  |  11134  |  TE_BO_ABORT_BOTP_DUE_BOPO_CANCELED| Details: cancel bracket take profit order due main order canceled.|


## CxlRejReason Field

| code | Reason | Description |
|-----|---------|-------------|
| 100 | CE_NO_ENOUGH_QTY | Details: qty is not enough |
| 101 | CE_WILLCROSS | Details: passive order rejected due to price may cross  |
| 116 | CE_NO_ENOUGH_BASE_QTY | Details: base-qty is not enough in spot-trading |
| 117 | CE_NO_ENOUGH_QUOTE_QTY | Details: quote-qty is not enough in spot-trading |

