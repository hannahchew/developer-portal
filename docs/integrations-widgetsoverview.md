---
id: Integrations-WidgetsPluginsOverview
title: Overview
---
## Introduction
The Widgets and Plugins are 2 different types of integration that can be easily implemented on any websites or applications. There are currently 2 widgets, the [KyberWidget](integrations-kyberwidgetguide.md) and the [Sign In Widget](integrations-signinwidgetguide.md), and one plugin, the [WooCommerce Plugin](integrations-woocommercepluginguide.md) that are available for users to implement.

## KyberWidget
The KyberWidget allows users to pay for goods using any token supported by Kyber while allowing merchants / vendors to accept the payment in their preferred token. It provides a friendly stock user interface for users to use ERC20 tokens to pay to an ETH address. Users can use different wallets of choice (e.g. keystore, Trezor, Ledger, private key, and MetaMask) to sign the transaction and make the payment. The widget will broadcast the transaction to the Ethereum network automatically and notify the app (vendors) about the transaction.

### How it works
![How the widget works](/uploads/widgetflow.png "Widgetflow")

The diagram above shows how components like the widget, Kyber Network smart contract, vendor wallet, vendor servers interact with each other.

### Demo
A simple demo is available at https://widget.knstats.com/shop/. If you specify https://kyberpay-sample.knstats.com/callback as callback URL, you could go to https://kyberpay-sample.knstats.com/list to see postback transaction list.

### Payment Status Verification
With KyberNetwork, the payment is made on the blockchain. Thus, vendors don't have to trust or rely on any data that comes from KyberNetwork but from the Ethereum network. However, vendors have to implement and run their own monitoring logic to get the payment status from the Ethereum network. Alternatively, in the near future, they may use and run the libraries that KyberNetwork and the community will provide.

#### Monitoring The Status
Before broadcasting the transaction, the widget will send all information about the payment (depending on the parameters), including the *transaction hash*, to the callback passed to the widget. At this point, the vendor's server is responsible to store that *transaction hash* and necessary payment information (e.g. userID, orderID, etc.) to monitor that transaction's status on the Ethereum network. The payment status will be determined based on that *transaction status* and the payment information.

##### Pending Payment
The payment is pending if the transaction is pending.
![Pending](/uploads/widget-guide-payment-status/pending.jpg "Pending")

##### Failed Payment
The payment failed if the transaction is reverted/failed.
![Fail](/uploads/widget-guide-payment-status/fail.jpg "Fail")

##### Successful Payment
![Success](/uploads/widget-guide-payment-status/success.jpg "Success")
If the transaction is successful, the vendor will still have to check if the payment amount (in ETH or in ERC20 tokens) that the user sent is equal to the expected amount (price of a specific order).

#### Verifying Payment Amount
In order to get the amount of tokens the user has sent, the vendor needs to see which type the transaction falls under, because each type has a different way to obtain the token amount sent by the user.
There are 3 different transaction types that the widget can broadcast.

##### Case 1: ETH -> ETH
Customers pay in ETH, vendors receive payment in ETH. This is a simple ETH transfer transaction, so the payment amount is the transaction value.

##### Case 2: Token A -> Token A
Customers pay in token A, vendors receive payment in token A. This is a simple ERC20 transfer transaction. The payment amount is logged in `event Transfer(address indexed _from, address indexed _to, uint _value)`, in the `_value` parameter.

##### Case 3: Token A -> Token B
Customers pay in token A, vendors receive payment in token B where B != A, and **either token could be in ETH.** This involves a `trade()` transaction. The payment amount is logged in `TradeExecute (index_topic_1 address origin, address src, uint256 srcAmount, address destToken, uint256 destAmount, address destAddress)`, in the `destAmount` parameter.<br>

**Note:** For **cases 2 and 3**, the token amount recorded is **in wei**, and so would have to be divided by `10^TOKEN_DECIMALS`.

**Pseudo code:**
```js
// DISCLAIMER: Code snippets in this guide are just examples and you
// should always do your own testing. If you have questions, visit our
// https://t.me/KyberDeveloper.

function paymentStatus(txhash, expectedPayment) -> return status {
  startTime = time.Now()
  txStatus = not_found
  loop {
    txStatus = getTxStatusFromEthereum(txhash) // possible returned value: not_found, pending, failed, success
    switch txStatus {
    case pending:
      // wait more, do nothing
      continue
    case failed:
      return "failed"
    case success:
      // TODO
    case not_found:
      if time.Now() - startTime > 15 mins {
        // if the txhash is not found for more than 15 mins
        return "failed"
      }
    }

  }
}
```

## Sign In Widget
The Sign In Widget allows your users who have been registered with our platform to log into your platform using the same log in credentials. Note that the widget is still under development and is only provided to select partners for early testing. You can learn how to use the Sign In Widget [here](integrations-signinwidgetguide.md).

## WooCommerce Plugin
E-commerce websites that are created with WordPress are traditionally restricted to fiat payment gateway. Our [extension](https://woocommerce.com/product-category/woocommerce-extensions/) of the WooCommerce plugin enables your users with the ability to pay in any tokens of their choice while you are receive the payment in your preferred token. Click [here](integrations-woocommercepluginguide.md) to learn how to integrate your WordPress powered e-commerce site with our extended WooCommerce plugin.
