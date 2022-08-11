---
description: Receive an inbound message from any Abacus supported network.
---

# Receive

Contracts must implement the \`handle()\` ABI defined below in order to receive messages from Abacus.

### Interface

```solidity
interface IMessageRecipient {
    function handle(
        uint32 _origin,
        bytes32 _sender,
        bytes calldata _messageBody
    ) external;
}
```

### Access Control

{% hint style="warning" %}
In `handle` implementations, it is important to restrict the `msg.sender` to the corresponding `origin` domain's `Inbox` address to prevent any contract from spoofing an inbound Abacus message.&#x20;
{% endhint %}

Developers can reference deployed [`Inbox` addresses](../addresses/) and corresponding [domain identifiers](../domains.md) in the docs. An example of `Inbox` access control implemented on `avalanche` for restricting inbound messages from `ethereum` is provided below.

```solidity
uint32 constant ethereumDomain = 0x657468;
address constant ethereumInbox = 0x0E3239277501d215e17a4d31c487F86a425E110B;
// for access control on handle implementations
modifier onlyEthereumInbox(uint32 origin) {
    require(origin == ethereumDomain && msg.sender == ethereumInbox);
    _;    
}
```

A generalized message access control layer is provided in the [`AbacusConnectionManager`](../building-applications/writing-contracts/abacusconnectionmanager.md) library, and Abacus Works has deployed and maintained these registries on all supported networks.&#x20;

### Encoding

{% hint style="info" %}
Abacus message senders are left-padded to `bytes32` for compatibility with virtual machines that are addressed differently.&#x20;
{% endhint %}

The following utility is provided in the [`TypeCasts` library](https://github.com/abacus-network/abacus-monorepo/blob/main/solidity/core/contracts/libs/TypeCasts.sol) for convenience.

```solidity
// alignment preserving cast
function bytes32ToAddress(bytes32 _buf) internal pure returns (address) {
    return address(uint160(uint256(_buf)));
}
```

### Example Usage

An example `handle` implementation for receiving messages from `ethereum` on `avalanche` is provided below.

```solidity
event Received(address sender, bytes message);
function handle(
    uint32 _origin,
    bytes32 _sender,
    bytes memory _message
) external onlyEthereumInbox(_origin) {
    address sender = bytes32ToAddress(_sender);
    emit Received(sender, _message);
}
```