# Chain Consensus Error

The Terp Network testnet halted at block `1497396`, for the network upgrade proposed. Ideally, the upgrade should halt the production of new blocks momentarily, while independent contributors confirm the exported state to be used for block-height 0, upgrade their terp-core daemon to the latest version v0.2.0, & proceed to resume block consensus with the networks validator peers.  

## Known Manual Changes needed for v0.1.2 to v0.2.0 migration
- removal of `"feeibc":[]` from genesis.json
- addition of 
```json
"feeibc": {
            "fee_enabled_channels": [],
            "forward_relayers": [],
            "identified_fees": [],
            "registered_counterparty_payees": [],
            "registered_payees": []
        },
```

This process was expected as the result from initial testing with a private chain mimicking these steps, & the migration process verified our predictions during testing. \
Unfortunately, this testing did not take into account the db state having contract instantiations, which ended up being a critical mistake in the testing workflow  
prior to proposing the upgrade.


# Halt

- Upon node operators attempting to resume consensus with  the first attempted exported state of `athena-2`, P2P connections were successful, however the network consensus only allowed pre-commits & not pre-votes from the first block, which is when it it was realized that new blocks were not going to be made. *see [commit_data_height_1497396.json](./logs/commit_data_height_1497396.json)

-  all node operators were asked to shut down their nodes and standby until we could discover the root of the issue. 

# Root of Issue
In order to verify the root of the issue was solved, contributors were using the `validate-genesis` function to verify the new genesis that had the exported state. This is what was found to be the root of our issues:
- App State params were not alphabetized upon migration
- Contract state param `created` was not included in any of the contract states on our test network
- Contract Codes 1-4 did not exist in the ~/.terp/wasm folders 

# Solutions 
- Alphabetize the app state params
- reformat genesis for wasmd v0.30.0
- Reset wasm state, reupload contracts.


