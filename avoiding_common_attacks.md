# Security Tools / Common Attacks

Both `MeetETH` and `solidity-treemap` CI integrate [`solium`](https://github.com/duaraghav8/Solium) linting, which performs [security static analysis](https://www.npmjs.com/package/solium-plugin-security#list-of-rules).

Automated security check [Mythril](https://github.com/ConsenSys/mythril) has been performed and found no actionable vulnerabilities. Detailed logs can be found at [mythril.log](/mytril.log).

No external contract call is performed other than the trusted `Reservation.sol` contract thus there is no **Reentrancy** vulnerability.

**Integer Overflow** have been mitigated with the use of [`SafeMath`](https://github.com/OpenZeppelin/openzeppelin-solidity/blob/master/contracts/math/SafeMath.sol) library in `Reservation.sol`. The incrementing token id is the only implemented arithmetic operation in the contract.

Only `cancelAll` function in `Calendar.sol` performs array iteration in either contract. However, the length of iteration is controlled by the arguments thus caller may fit their request under block gas limit by limiting the window of cancellation. This mitigates the **Denial of Service** attack vector.

There is no **timestamp dependency** in the contract. There is no handling of Ether thus **Force Sending Ether** does not cause any problem.
