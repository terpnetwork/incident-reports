# Incident Report: Improperly Derived Pubkeys For Headstash Allocation

This report provides context of how what the issue is, how the issue happened, actions needed to resolve the issue, as well as future possible, proactive measures that can be taken to minimize the chance of the issue occuring again.

## The Issue
Public addresses generated for the [headstash allocation](https://github.com/terpnetwork/terp-core/blob/main/app/upgrades/v3/headstash_data.go) were improperly derived from the indended pubkey of the [headstash recipients](https://github.com/terpnetwork/airdrop#airdrop-cycle-2-cannabis-culture-communities).

## The Cause
In order to correctly derive a pubkey with the new [Metamask Snaps](https://metamask.io/snaps/) plugin, in short, the private key must be included in the process that generates & provides the pubkey deterministically. Since there cannot be access to the private keys of the included pubkeys, you can never pre-determine the pubkey without the private key. 

## Resolving Actions 
- A software upgrade will be proposed that will handle reverting the previous allocations
- As well as uploading & instantiating the smart contracts that will handle the pubkey verification for proper headstash allocation & distribution. 

## Proactive Measures
We can create a vesting bounty to incentivize the development of a ui-widget that is configured to the smart contracts that handle verification, so that users can create their own headstashes.