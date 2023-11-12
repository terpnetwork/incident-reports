# Terp Network Incident Response

(credit for this guide goes to CosmosContracts)

A repo for tracking resources related to resolving incidents on the Terp Core.

## Security Disclosures

For security disclosures, follow [these instructions](https://github.com/terpnetwork/terp-core/blob/main/SECURITY.md).

## How to use this repo

1. Create a folder with the date of the incident \
2. Collect resources related to root cause: \
    **P2P or consensus error:** \
            > Blocks leading up prior error, & post error if applicable \
            > Commit logs leading up prior error, & post error if applicable \
            > Any resourceful consensus state 

3. If you need to recover follow an `upgrade.md` with the steps involved.
4. To restore IBC clients, follow the `ibc_client_recovery.md` guide
5. Provide remidation solution, if applicable.