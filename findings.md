### [M-1] Looping through players array to check for duplicates in `PuppyRaffle::enterRaffle` is a potential DOS(DENIAL OF SERVICE) attack, incrementing gas costs for future entrants.


**Description:** The `PuppyRaffle::enterRaffle` function loops through the `players` array to check for duplicates.However the longer `PuppyRaffle::players` array is the more checks new players have to  make making the gas costs for new players expensive vs the players who participated earlier

**Impact:** An attacker might intentionally increas the `players` array to increase his chances for that particulart NFT.

**Proof of Concept:**
If we have 2 sets of 100 player gas cost will be as such-
1 set-6252039
2 set-18067741

this is 3X times more expensive for the second players.

<details>
<summary>POC</summary>

Place the following code in `PuppyRaffleTest.t.sol`

```javascript
function testReadDuplicateGasCosts() public {
        vm.txGasPrice(1);

        // We will enter 100 players into the raffle
        uint256 playersNum = 100;
        address[] memory players = new address[](playersNum);
        for (uint256 i = 0; i < playersNum; i++) {
            players[i] = address(i);
        }
        // And see how much gas it cost to enter
        uint256 gasStart = gasleft();
        puppyRaffle.enterRaffle{value: entranceFee * playersNum}(players);
        uint256 gasEnd = gasleft();
        uint256 gasUsedFirst = (gasStart - gasEnd) * tx.gasprice;
        console.log("Gas cost of the 1st 100 players:", gasUsedFirst);

        // We will enter 100 more players into the raffle
        for (uint256 i = 0; i < playersNum; i++) {
            players[i] = address(i + playersNum);
        }
        // And see how much more expensive it is
        gasStart = gasleft();
        puppyRaffle.enterRaffle{value: entranceFee * playersNum}(players);
        gasEnd = gasleft();
        uint256 gasUsedSecond = (gasStart - gasEnd) * tx.gasprice;
        console.log("Gas cost of the 2nd 100 players:", gasUsedSecond);

        assert(gasUsedFirst < gasUsedSecond);
        // Logs:
        //     Gas cost of the 1st 100 players: 6252039
        //     Gas cost of the 2nd 100 players: 18067741
    }
```
</details>

**Recommended Mitigation:** They are a few recommendations.
1. Consider allowing duplicates. Users can make duplicate addresses anyways so functionality doesnt matter.

2. Consider using mapping if you want the same functionality.

3. Or use `Openzepplin's EnumerableSet library` that might help.