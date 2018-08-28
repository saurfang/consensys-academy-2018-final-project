# Design Pattern and Decisions

## Design Patterns

The following patterns are implemented.

### Fail early and fail loud

All public functions that interacts with Calendar checks whether the calendar exists in `Calendar.sol` right away. For example:

```solidity
function cancel(uint256 _tokenId, uint256 _reservationId)
  public
  whenNotPaused()
  {
    require(exists(_tokenId), "Calendar does not exist");

    ...
  }
```

All public functions that requires ownership checks whether the caller is the owner or has been approved in `Reservation.sol` right away. For example:

```solidity
function cancel(address _owner, uint256 _tokenId)
  external
  onlyOwner()
  {
    super._burn(_owner, _tokenId);

    ...
  }
```

`super._burn` throws if `_owner` doesn't own `_tokenId` internally.

### Restricting Access

All functions are marked as public as they are meant to be used for uer interactions. However, all functions and constants in [`TreeMap.sol`](https://github.com/saurfang/solidity-treemap/blob/master/contracts/TreeMap.sol) have protected `internal` and `private` modifiers to prevent accidental mis-use from users and contracts.

### Circuit Breaker

`Calendar.sol` contract [extends](https://github.com/saurfang/meeteth/blob/master/contracts/Calendar.sol#L10) `Pausable` lifecycle contract from [openzeppelin-solidity](https://github.com/OpenZeppelin/openzeppelin-solidity/blob/master/contracts/lifecycle/Pausable.sol).Public functions are marked with `whenNotPaused` modifier to ensure the contract can be paused in the event of bugs.

Because `transfer` and other standard ERC721 are implemented without modifications, they are intentionally not overriden with the `whenNotPaused` modifiers. That means transfer will continue to be possible when the `Calendar` contract is paused.

`Reservation.sol` contract does not need emergency stop because `reserve` and `cancel` are the only non-inherited public functions and they are guarded with `onlyOwner`. Hence, only `Calendar.sol` may operate `Reservation.sol` and a paused `Calendar.sol` would free `Reservation.sol` correspondingly.

### Not implemented patterns

**Auto Deprecation** and **Mortal** does not apply because the contract serves as a ERC721 token tracker and should be kept forever for record keeping.

**Pull over Push Payments** does not apply because payments have been intentionally left out of this proof of concept because reservation payment (deposit, refund, and etc) can be very complex.

**State Machine** does not apply because there is no obvious state change in the current design. If the calendar requires calendar owner to accept a reservation, we would implement a state machine at the token level that marks a reservation as `REQUESTED, CONFIRMED, CANCELLED` etc.

**Speed Bump** has not been implemented as we envision the contract can be used in high frequency scenario. For example, if the contract is deployed in a plasma chain that has very short block time, it could support real time reserving and bettering activity such as bidding and trading for right of the road between vehicles. **However**, we impose a limit on the duration of a reservation to 8 hours because we don't have other spam protection mechanism in place. This serves as a soft speed bump that user will need to send multiple transactions to squat a calendar.

## Design Decisions

It is worth noting that we broke the problem into two separate ERC721 token contracts: `Calendar.sol` and `Reservation.sol`. This forms a two-tier token approach where Calendar and Reservations form a one-to-many relationship. Furthermore, we make `Reservation.sol` a `Ownable` contract owned by `Calendar.sol` such that we may implement Reservation specific functions in `Reservation.sol` while keeping the user interaction functions with `Calendar.sol`. This creates a good separate of concerns where the attach surface area is restricted in `Calendar.sol`.
