---
description: >-
  Brief overview of all the imperative functionalities of the Push Communicator
  smart contract
---

# Methods - Comm

## Only Admin Setter Functions

1. **setEPNSCoreAddress(address)**

```
function setEPNSCoreAddress(address _coreAddress) external onlyPushChannelAdmin() {};
```

|       Argument      |    Type   |              Description             |
| :-----------------: | :-------: | :----------------------------------: |
| _**\_commAddress**_ | _address_ | Address of the communicator protocol |

\*_Description:_

* Allows only the Push Channel Admin to set the EPNS Communicator smart contract's address

1. **setGovernanceAddress(address)**

```
function setGovernanceAddress(address _governanceAddress) external onlyPushChannelAdmin() {};
```

|          Argument         |    Type   |             Description            |
| :-----------------------: | :-------: | :--------------------------------: |
| _**\_governanceAddress**_ | _address_ | Address of the Governance protocol |

**Description:**

* Allows only the Push Channel Admin to set the Governance address

1. **transferPushChannelAdminControl(address)**

```
function transferPushChannelAdminControl(address _newAdmin) public onlyPushChannelAdmin() {};
```

|     Argument     |    Type   |        Description       |
| :--------------: | :-------: | :----------------------: |
| _**\_newAdmin**_ | _address_ | address of the new Admin |

**CheckPoints:**

* Can only be called by the current Push Channel Admin.
* _**\_newAdmin**_ address being passed as an argument must be a valid address.
* _**\_newAdmin**_ address being passed as an argument must not be the already existing Push Channel Admin's address

**Description:**

* Changes the Push Channel admin's address to a new Address.

## Core Functionalities

### SendNotification Functions

1. **sendNotification(address, address, bytes)**

```
function transferPushChannelAdminControl(address _newAdmin) public onlyPushChannelAdmin() {};
```

|      Argument     |    Type   |                             Description                             |
| :---------------: | :-------: | :-----------------------------------------------------------------: |
|  _**\_channel**_  | _address_ |    address of the channel through which the notification is sent    |
| _**\_recipient**_ | _address_ |             address of the recipient of the notification            |
|  _**\_identity**_ |  _bytes_  | identity bytes that contain the crucial data about the notification |

**CheckPoints:** The **sendNotification()** is executed if any one of the below-mentioned conditions are satisfied:

* If channel is a **zero address**, then the caller of the function must be **Push Channel Admin** itself.
* If Channel addressed passed in the argument is any channel in an **Active state** in the protocol, then the _**sendNotification()**_ must be called by the channel owner itself.
* If caller of the function is a delegate notification sender, then the caller must be approved by the channel to send notification on behalf of the channel.
* If the caller of a the function is any random user, then the recipient of the notification should be the caller(_msg.sender_) itself. _This ensures that any user, who is not a channel or a valid delegate notification sender, can send notifications as well but only to themselves_

All of these checks are handled by the _**\_checkNotifReq()**_ function in the communicator protocol, as shown below:

```solidity
function _checkNotifReq
    (
      address _channel,
      address _recipient
    ) private view
    {
      require(
          (_channel == 0x0000000000000000000000000000000000000000 &&
              msg.sender == pushChannelAdmin) ||
              (_channel == msg.sender) ||
              (delegatedNotificationSenders[_channel][msg.sender]) ||
              (_recipient == msg.sender),
          "EPNSCommV1::_checkNotifReq: Invalid Channel, Delegate or Subscriber"
      );
    }
```

**Description:**

* If any one of the above-mentioned conditions is met, the function is executed and it emits out a **SendNotification** event.

1. **addDelegate(address)**

```
function addDelegate(address _delegate) external {};
```

|     Argument     |    Type   |       Description       |
| :--------------: | :-------: | :---------------------: |
| _**\_delegate**_ | _address_ | address of the delegate |

**Description:**

* Allows a particular channel owner to approve any specific wallet address as the delegate notification sender for the channel.
* Once approved, the approved address shall be able to send a notification on behalf of the channel.

1. **removeDelegate(address)**

```
function removeDelegate(address _delegate) external {};
```

|     Argument     |    Type   |       Description       |
| :--------------: | :-------: | :---------------------: |
| _**\_delegate**_ | _address_ | address of the delegate |

**Description:**

* Allows a particular channel owner to revoke the approval of its delegate notification sender.
* Once the approval is revoked, the delegate notification sender can no longer send notification on behalf of that channel.

### Subscribe & Unsubscribe Functions

1. **subscribe(address)**

```
function subscribe(address _channel) external returns (bool) {};
```

|     Argument    |    Type   |       Description      |
| :-------------: | :-------: | :--------------------: |
| _**\_channel**_ | _address_ | address of the channel |

**Checkpoints:**

* The channel argument passed in the function must be a valid channel in an **ACTIVE** state.
* The caller of the function(_subscriber_) must not already be subscribed to the channel.

> _**Note**_ -> The validation of whether or not a channel address is a valid channel, is performed at the backedn by the **Push Nodes** and not by the smart contract.

\*_Description:_

* If the caller of the function is a new user and not yet added to the protocol, the user is added as a valid user of the communicator protocol.
* However, if the user was already added the function proceeds with its execution and stores imperative information about the **subscribe action** in the smart contract.
* The caller of the function is marked as a valid subscriber of the channel and his/her subscribe count is increased.
* Emits out the **Subscribe** event with the _Channel and Subscriber's address_.

1. **batchSubscribe(address\[])**

```
function batchSubscribe(address[] calldata _channelList) external returns (bool) {};
```

|       Argument      |   Type  |        Description        |
| :-----------------: | :-----: | :-----------------------: |
| _**\_channelList**_ | _array_ | list of channel addresses |

**Checkpoints:**

* The channel arguments passed in the function must be a valid channel in an **ACTIVE** state.
* The caller of the function(_subscriber_) must not already be subscribed to the channel.

> _**Note**_ -> The validation of whether or not a channel address is a valid channel, is performed at the backedn by the **Push Nodes** and not by the smart contract.

\*_Description:_

* Allows a user to subscribe to a list of different channels at once.

1. **subscribeViaCore(address)**

```
function subscribeViaCore(address _channel, address _user) external onlyEPNSCore returns
( bool) {}};
```

|     Argument    |    Type   |        Description        |
| :-------------: | :-------: | :-----------------------: |
| _**\_channel**_ | _address_ |   address of the channel  |
|   _**\_user**_  | _address_ | address of the subscriber |

**Checkpoints:**

* It can only be called by the EPNSCore smart contract

\*_Description:_

* Allows the EPNS Core smart contract to perform the necessary susbcribe actions whenever a user creates his own channel.
* For instance, whenever users create their own channel, they must be subscribed to the following channels: a. EPNS Alerter Channel b. Push Channel Admin c. Channel owner must subscribe to its own channel as well.
* The **subscribeViaCore()** helps the EPNS Core to perform the above-mentioned subscribe actions as such actions can only happen in the EPNS Communnicator protocol.

1. **unsubscribe(address)**

```
function unsubscribe(address _channel) external returns (bool){};
```

|     Argument    |    Type   |       Description      |
| :-------------: | :-------: | :--------------------: |
| _**\_channel**_ | _address_ | address of the channel |

**Checkpoints:**

* The caller of the **unsubscribe()** function should already be a subscriber of the _**\_channel**_.

\*_Description:_

* The caller of the function is removed as a subscriber of the channel
* The user's on-chain data is updated and his/her **subscribe count**, i.e., _total number channels the user is subscribed to_, is decreased by 1.
* Emits out the **Unsubscribe** event with the _Channel and Subscriber's address_.

1. **batchUnsubscribe(address\[])**

```
function batchUnsubscribe(address[] calldata _channelList) external returns (bool) {};
```

|                     |          |                           |
| ------------------- | -------- | ------------------------- |
| **Argument**        | **Type** | **Description**           |
| _**\_channelList**_ | _array_  | list of channel addresses |

**Checkpoints:**

* The caller of the function should already be a subscriber of the list of channels provided as argument.

\*_Description:_

* Allows the user to unsubscribe from a list of channels at once.

1. **\_addUser(address)**

```
function _addUser(address _user) private returns (bool userAlreadyAdded) {};
```

|              |           |                                                 |
| ------------ | --------- | ----------------------------------------------- |
| **Argument** | **Type**  | **Description**                                 |
| _**\_user**_ | _address_ | address of the user to be added in the protocol |

**Description:**

* Checks whether or not the user address passed as an argument is activated in the protocol.
* Simply returns true if the user is already added to the protocol.
* However, if the user address is not yet added, this function activates the user's address, stores the block number as the user start block in the protocol.
* It also increases the total user count of the communicator protocol.

1. **broadcastUserPublicKey(bytes)**

```
function broadcastUserPublicKey(bytes calldata _publicKey) external {};
```

|              |               |                            |
| ------------ | ------------- | -------------------------- |
| **Argument** | **Type**      | **Description**            |
| _**bytes**_  | _\_publicKey_ | public key to be broadcast |

**Description:**

* The function returns if the given _\_publicKey_ is already broadcasted in the protocol.
* However, if the public key is not yet broadcasted, the function performs its checks and marks the **publicKeyRegistered** member of the User struct as **true.**.
* Emits out the **PublicKeyRegistered()** event with the _User's wallet address & the Public key._

1. **verifyChannelAlias(string)**

```
function verifyChannelAlias(string memory _channelAddress) external{};
```

|                        |          |                                               |
| ---------------------- | -------- | --------------------------------------------- |
| **Argument**           | **Type** | **Description**                               |
| _**\_channelAddress**_ | _string_ | address of the channel on Ethereum blockchain |

**Description:**

* An important and compulsory functions for channel owners to verify their channel alias.
* The function emits out 4 crucial details:
  * Chain name: _This represents the blockchain name on which the EPNS Communicator is deployed_
  * Chain ID: _Denotes the Chain ID of the blochain where the EPNSCommV1 is deployed_
  * Caller's Address(**msg.sender**): _The address of the channel owner on the Communicator protocol._
  * Channel Address: _This represents the address of the main channel that exists on Ethereum blockchain_
* The entire verification procedure is handled by the **PUSH Nodes**.

### Meta Transaction Functions

EPNS Communicator protocol supports meta transactions or, in other words, gasless transactions. In simpler terms, the protocol allows you to interact with its functions without actually paying any gas fee.

All you really need to do is simply sign the transaction that you want to put on the blockchain with your keys. This action doesn't really require any gas fees as you aren't actually interacting with the protocol on-chain but just signing a transaction. The signed transaction will then be submitted on the blockchain by the operator by paying some gas fees. Once the transaction is submitted on the blockchain, the communicator protocol verifies your signature to ensure it's a valid signed transaction and executes the required function on your behalf. The protocol currently allows meta transactions for the following functions:

1. **subscribeBySig(address, uint256, uint256, uint8, bytes32, bytes32)**

```
function subscribeBySig
    (
        address channel,
        uint256 nonce,
        uint256 expiry,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) public {};
```

**Checkpoints:**

* Signer's address must not be invalid.
* Nonce passed as an argument should be a valid one.
* Transaction must not be expired.

\*_Description:_

* Allows user to subscribe to any specific channel thorugh gasless transaction.

1. **unsubscribeBySig(address, uint256, uint256, uint8, bytes32, bytes32)**

```solidity
function unsubscribeBySig
    (
        address channel,
        uint256 nonce,
        uint256 expiry,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) public {};
```

**Checkpoints:**

* Signer's address must not be invalid.
* Nonce passed as an argument should be a valid one.
* Transaction must not be expired.

\*_Description:_

* Allows user to unsubscribe from any channel thorugh meta transaction.

1. **sendNotifBySig(address, address, bytes, uint256, uint256, uint8, bytes32, bytes32)**

```
function sendNotifBySig
    (
        address _channel,
        address _recipient,
        bytes calldata _identity,
        uint256 nonce,
        uint256 expiry,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) external {};
```

**Checkpoints:**

* Signer's address must not be invalid.
* Nonce passed as an argument should be a valid one.
* Transaction must not be expired.

\*_Description:_

* The EPNS Communicator smart contract allows users to send notification in a gasless fashion via meta transaction.
* The notification, however, shall only be triggered from the smart contract if all the requirements for sending notification is met.
