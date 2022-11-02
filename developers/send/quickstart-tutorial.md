# Quickstart Tutorial

This tutorial demonstrates how to make a simple call via Interchain Accounts to a pre-deployed [`TestRecipient`](https://github.com/hyperlane-xyz/hyperlane-monorepo/blob/main/solidity/core/contracts/test/TestRecipient.sol) contract on a remote destination chain. You can also check out the [`hyperlane-quickstart`](https://github.com/hyperlane-xyz/hyperlane-quickstart) repo for running this out of the box.

### Inputs

* `$DESTINATION_DOMAIN`: The domain ID of the destination chain. Domain IDs can be found [here](../domains.md).
* `$RECIPIENT`: The address of the `TestRecipient` contract on the destination chain,`` `0xBC3cFeca7Df5A45d61BC60E7898E63670e1654aE` ``

### Make a call

Sending a message is a simple matter of calling `Outbox.dispatch()`. This function can be called easily using Etherscan+[Metamask](https://metamask.io/) or [cast](https://book.getfoundry.sh/cast/).

{% tabs %}
{% tab title="Using Metamask" %}
1. Navigate to the `InterchainAccountRouter 0x28DB114018576cF6c9A523C17903455A161d18C4` contract page on [Etherscan](https://etherscan.io/address/0x28DB114018576cF6c9A523C17903455A161d18C4) (or whatever chain you want to send from)
2. Under the `Contract` tab, find the `Write Contract` button.
3. Click on the `Connect to Web3` button to connect your Wallet (i.e. Metamask). Make sure that you are on the correct network.
4. Expand the `dispatch` box.
5. For destination domain, enter `$DESTINATION_DOMAIN`. You can find some [here](../domains.md), or you could use `0x706f6c79` to send to Polygon.
6. For the `calls` argument, we have to pass an array of `Call` (which itself is a tuple of `(to: address, calldata: bytes)` For this demo on Etherscan that looks like `[["$RECIPIENT", "$CALLDATA"]]`
   1. `$RECIPIENT` is just our `TestRecipient` at `0xBC3cFeca7Df5A45d61BC60E7898E63670e1654aE`
   2.  Our `$CALLDATA` is the abi-encoded function call `fooBar(uint256,string)` (or whatever function call you want to make). In this case this would come to:

       1. ➜ cast calldata "fooBar(uint256,string)" 1 "HelloWorld from an ICA"&#x20;

       which generates `0xf07c1f4700000000000000000000000000000000000000000000000000000000000000010000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000001648656c6c6f576f726c642066726f6d20616e2049434100000000000000000000`

       You can use your preferred abi encoding tool (like [https://abi.hashex.org/](https://abi.hashex.org/)) as well
   3. Using our `TestRecipient` and the function call shown above, the tuple looks like:\
      `[["0xBC3cFeca7Df5A45d61BC60E7898E63670e1654aE", "0xf07c1f4700000000000000000000000000000000000000000000000000000000000000010000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000001648656c6c6f576f726c642066726f6d20616e2049434100000000000000000000"]]`
7. Submit the transaction via your wallet/Metamask



<figure><img src="../../.gitbook/assets/Screen Shot 2022-10-04 at 4.24.21 PM.png" alt=""><figcaption></figcaption></figure>
{% endtab %}

{% tab title="Using cast" %}
You can call the `InterchainAccountRouter` directly using `cast`. Make sure that you have a valid RPC URL for the origin chain and a private key with which you can pay for gas.

{% code overflow="wrap" %}
```shell
cast send 0x28DB114018576cF6c9A523C17903455A161d18C4 'dispatch(uint32, (address,bytes)[])' $DESTINATION_DOMAIN "[($RECIPIENT,$(cast calldata "fooBar(uint256,string)" 1 "HelloWorld from an ICA via cast"))]" --rpc-url $RPC_URL --private-key $PRIVATE_KEY
```
{% endcode %}
{% endtab %}
{% endtabs %}

If you view the transaction on a block explorer, you should be able to see the `Dispatch` event. You can see an example message sending transaction [here](https://goerli.etherscan.io/tx/0xbb076b17dca5e436f574a4728dd59d25da4fd9d05c48c6ec304ea5a354849edf).



### Confirm delivery

After the transaction that sent your call is [finalized](../latencies.md), you should be able to see a corresponding transaction delivering your message to the `TestRecipient` contract on the destination chain. You can watch for this transaction on the destination chain's block explorer by querying for the recipient's address (in this guide `0xBC3cFeca7Df5A45d61BC60E7898E63670e1654aE)`.&#x20;

If your message body was a human readable string, you can view it in tab with the list of events by selecting "Text" in the dropdown for the fourth parameter.\
\
You can see an example message delivery transaction [here](https://alfajores.celoscan.io/address/0xBC3cFeca7Df5A45d61BC60E7898E63670e1654aE#events).

<figure><img src="../../.gitbook/assets/ICA Quickstart Polyscan.png" alt=""><figcaption><p>This transaction sent a "Hello World" message from an Interchain Account</p></figcaption></figure>

Read more under the [`Where is my message?` section](../observability.md) to use tools like the[ Hyperlane Message Debugger.](https://explorer.hyperlane.xyz/debugger)