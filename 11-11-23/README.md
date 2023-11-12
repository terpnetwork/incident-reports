# Incident Report: v4 upgrade postmortem

This report provides context for the issues we encountered during the v4 upgrade, which led to node operators downgrading to the previous binary version, and skipping the upgrade. 

This also tracks the errors we have in the headstash contract. 

## v4 upgrade rollback 
Upon reaching the upgrade height `3341663`, freshly upgrade nodes were greeted with the fatal panic: `panic: can not create code: unauthorized`. 

### The Cause
Terp Network is currently a permissioned wasm chain, requiring a governance approval to store wasm binaries on chain. The logic that handled storing & instantiating the wasm binary on chain during the upgrade had the incorrect permissions configured, [see reference](https://github.com/terpnetwork/terp-core/blob/main/app/upgrades/v4/headstash-contract.go#L29C37-L30C1).

Our contributors assumed that since the govModule address was defined as the account in the instantiation message, that the network would proceed to store the wasm binary on chain. 

### Resolving actions 
- validators rolled back their states to the previous block 
- validators downgraded their binary versions to v3.1.0
- validators were asked to resume consensus, skipping over the upgrade height 
- a traditional store wasm proposal to store the headstash patch contract
- a software upgrade proposal for reverting the distribution event in v3  

## headstash patch bugs 
There are some small bugs prominent in our [headstash contract patch](https://github.com/terpnetwork/headstash-patch) that can be also highlighted.

- [token denoms are hard-coded](https://github.com/terpnetwork/headstash-patch/blob/main/contracts/headstash-contract/src/state.rs#L15) to (`uterp` & `uthiol`), meaning contract deployed on test-net cannot distribute testnet tokens. 
- [clawback function](https://github.com/terpnetwork/headstash-patch/blob/main/contracts/headstash-contract/src/contract.rs#L159) is incomplete.

### Resolving actions 
- prepare contract for test-net, hard coded with native test-net denoms.
- deploy new contract for ui testing
- migrate old contract


### Proactive measures
adamant peer review, unit & integration tests, as well as manual test within simulated environment as production is the best possible proactive method to minimize situations like these from happening. 