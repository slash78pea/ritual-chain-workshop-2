# Deep Technical Notes – Bootcamp Level 2

## 1. Scheduler booking strategy
A single schedule() call reserves three future executions spaced 200 blocks apart.  
If any attempt succeeds, the contract immediately cancels the remaining ones.  
This gives resilience without leaving dangling scheduled calls.

## 2. Precompile execution path
- TEEServiceRegistry.pickServiceByCapability(HTTP_CALL, true, seed, 8)
- HTTP precompile (0x0801) performs the GET inside TEE
- Response bytes are handed to jq precompile (0x0803) with outputType = uint256
- Result is compared against the immutable target using the stored comparator

## 3. Failure isolation
Any of the following is treated as a failed attempt:
- Non-200 status
- Undecodable response envelope
- Executor error message
- jq parse failure
- Precompile revert

After three failures the market is permanently marked Invalid and every participant can withdraw their original stake.

## 4. Payout design
claimWinnings uses the classic pari-mutuel formula:
userShare = userStake * totalPool / winningPool

No loops over participants → gas stays predictable even with many bettors.

## 5. Personal observations
The decision to treat failed oracle reads as Invalid rather than NO is the most important safety feature in the whole design.  
Also appreciate that block.timestamp is completely avoided (Ritual’s timestamp is in milliseconds anyway).

Overall this is one of the cleanest workshop codebases I have seen.
