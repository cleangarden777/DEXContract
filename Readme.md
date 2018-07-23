# DEX Exchange Contract

DEXContract is a Etereum smart contract for decentralized exchange which is working on ethereum chain. 

## License
This contract is based on IDEX smart contract, and inserted custom logic.
You can find the original [IDEX contract here.](https://etherscan.io/address/0x2a0c0dbecc7e4d658f48e01e3fa353f44050c208#code)

## Main APIs
All APIs except balanceOf can be called by admin privillige. That means only admin account can call main apis of DEXContract such as trade, withdraw or cancel.

### Trade

Do trade action between two orders which are already selected as matched pair.

When a user makes or takes an order, this is the function that is called. 
```
trade(uint256[8] tradeValues, address[4] tradeAddresses, uint8[2] v, bytes32[4] rs)
``` 

**Params:**
* tradeValues: 
```
	0:amountGet: entire amount of tokens to get in order
	1:amountGive: entire amount of tokens to give in order.
	2:expires: timestamp until when transaction expires.
	3:nonce: nonce of order.
	4:amount: traded amount.
	5:tradeNonce: trade nonce of taker
	6:feeMake: maker fee of transaction
	7:feeTake: taker fee of transaction
```
* addressses:
```
     	0:tokenGet: token address to get in order
	1:tokenGive: token address to give in order
	2:maker: user account address who make order
	3:taker: user account address who take order
```
* signature binary
```
     v[0] rs[0] rs[1] : signature for order
     v[1] rs[2] rs[3] : signature for trade
```

While everything is summarized here, you can find an [in-depth explanation in this gist.](https://gist.github.com/raypulver/2f318db5dc497cab8019d3ae391af1d2#file-trade-fields-md)

Although the signature values are important to the contract, you shouldn't be needing them for any trade specific calculations.

**Keep in mind:** all trade values and amounts are in pure `uint256` values, meaning there is no decimal math calculated. 0.0001 ETH = 100000000000000.

Total fees can also be calculated using that info above, specifically using the additions to the fee account.

Only trades that are matched are broadcasted to the network, so only matched trades are counted as transactions which means there's no need to check if orders are taken.

## Deposit tokens to DEX contract
For the Deposits there are a few functions to monitor for:

```
function depositToken(address token, uint256 amount)
```

For `depositToken(address token, uint256 amount)`: It is specifically for ERC20 compliant token deposits. The `token` address and `amount` being deposited are passed in, added to the user balance for the token and then deposit event is emitted. Tracking the calls to this function will give you the tokens deposited.

## Deposit ETHs to DEX contract
```
function deposit() payable
```

Now for `deposit()`, it is for specifically ETH deposits (In the smart contract ETH is marked as a 0x0 address). It is a payable function where the value of the TX is added to the exchange balance, and the deposit event it emitted. Tracking the calls to this function will give you the ETH deposited.

## Withdrawals by Admin

For the Withdrawals there are a few functions to watch for:

**Notice:** These functions both handle _ETH_ and _ERC20_ at once. If the _token_ value is set to the 0x0 address, it will withdraw ETH, and if it is a actual token address it will withdraw that specific token. Remember to maintain decimals after retrieving the data or when you display it so numbers are accurate.

```
function adminWithdraw(address token, uint amount, address user, uint nonce, uint8 v, bytes32 r, bytes32 s, uint feeWithdrawal)
```
With `adminWithdraw()`, it has a lot of parameters but the main ones you want to watch for are `token` and `amount` for withdraw calculations. if you would like to track the fees you can also track `feeWithdrawal` which is taken out of the token being transferred.

**Note:** Some contract transactions go back to pre-byzantium fork, so if you would like remove transactions that failed, note that `transactionStatus` in a tx receipt is considered undefined if it is in a block from pre-byzantium.

## Balance

Balance of users on the exchange can be retrieved using `balanceOf(address token, address user)`:

```
function balanceOf(address token, address user)
```

Through which you pass in the token address and the user you would like to see the balance of. The contract balance in its entirety can be retrieved through web3 or calling the `balanceOf()` function from the respective tokens you want to check.

**Note:** To get the balance of ETH that an account has in the exchange, pass in _0x0000000000000000000000000000000000000000_ as the token address.

## Cancel order
```
function cancelOrder(address tokenGet, uint amountGet, address tokenGive, uint amountGive, uint expires, uint nonce, uint8 v, bytes32 r, bytes32 s, address user)
```

Cancel order specified by tokenGet, amountGet, tokenGive, amountGive, expires and user.
This order information is checked with signature values of v, r and s, and this leads order hash, which will be used to indicate the order which will be canceled.
