# Response Codes


Every response contains <font face="Courier New">responseCode</font> and <font face="Courier New">responseDesc</font> fields. These indicate whether the request was successful or failed. The only successful </font> is denoted by a value of <em>-1</em>. All other response codes indicate failed requests. The table belows denotes all possible response codes.


Response Code | Response Description
---------- | -------
-1  | Successful
0001  | General System Error
0023  | Required field missing or invalid type used.
2001  | Invalid transaction type supplied.
2002  | Invalid token type supplied.
2107  | Transaction to VSP timed out.
2108  | VSP not found.
2110  | Failed to log transaction products.
2111  | API POS not found.
2112  | Store not found.
2114  | Store does not support Deposits.
2115  | Store does not support Payments.
2116  | Store does not support Withdrawals.
2117  | VSP does not support Deposits.
2118  | VSP does not support Payments.
2119  | VSP does not support Withdrawals.
2120  | Invalid token.
2122  | The totalAmount does not match basketAmount + cashbackAmount.
2124  | POS provider not active.
2125  | Store is not active.
2121  | Cashback can only be processed on a Payment.
2126  | Store is not linked to POS provider.
2127  | Store does not support Cashbacks.
2128  | VSP does not support Cashbacks.
2129  | Store does not accept VSP.
2130  | Store does not allow Deposits for VSP.
2131  | Store does not allow Payments for VSP.
2132  | Store does not allow Withdrawals for VSP.
2133  | Store does not allow Cashback on Payments for VSP.
