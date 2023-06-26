# Stride ICA with Osmosis as host chain

## Start docker

### Checkout repo
```
git clone https://github.com/Stride-Labs/stride
cd stride && git checkout v10.0.0
```
### Download dep chains and relayers
`git submodule update --init --recursive`

### Note: Init chain and config
ICA related accounts are set up. 

1. Set hostzone [here](https://github.com/Stride-Labs/stride/blob/v10.0.0/dockernet/src/init_chain.sh#L48)
```
    # Add interchain accounts to the genesis set
    jq "del(.app_state.interchain_accounts)" $genesis_config > json.tmp && mv json.tmp $genesis_config
    interchain_accts=$(cat $DOCKERNET_HOME/config/ica_host.json)
    jq ".app_state += $interchain_accts" $genesis_config > json.tmp && mv json.tmp $genesis_config
```

### Start docker for stride and osmo
- Add OSMO to list of HOST_CHAINS [here](https://github.com/Stride-Labs/stride/blob/v10.0.0/dockernet/config.sh#L19)
`HOST_CHAINS=(OSMO) `
- Build and start docker for Stride, Osmosis and Relayer
`make start-docker build=sor`

Successful run would output:
```
Initializing STRIDE chain...
Node #1 ID: 5e7585adf48ac4102ab6d94295552e959baa7312@stride1:26656
Node #2 ID: 07a1c0196f842f1f7940bab365481a4ec24c4e2d@stride2:26656
Node #3 ID: 62ba8df1c38e887bb00c838f3bc25fc9089c400d@stride3:26656
Initializing OSMO chain...
Node #1 ID: 61a819c6a8ed1905e096260a5eb08016d653e3e1@osmo1:26656
Starting STRIDE chain
[+] Building 0.0s (0/0)                                                                                                                                       
[+] Running 4/4
 ✔ Network dockernet_default      Created                                                                                                                0.0s 
 ✔ Container dockernet-stride2-1  Started                                                                                                                0.7s 
 ✔ Container dockernet-stride3-1  Started                                                                                                                0.7s 
 ✔ Container dockernet-stride1-1  Started                                                                                                                0.7s 
Starting OSMO chain
[+] Building 0.0s (0/0)                                                                                                                                       
[+] Running 1/1
 ✔ Container dockernet-osmo1-1  Started                                                                                                                  0.5s 
Waiting for STRIDE to start...Done
Waiting for OSMO to start...Done
STRIDE <> OSMO - Adding relayer keys...Done
STRIDE <> OSMO - Creating client, connection, and transfer channel...Done
[+] Building 0.0s (0/0)                                                                                                                                       
[+] Running 1/1
 ✔ Container dockernet-relayer-osmo-1  Started                                                                                                           0.4s 
OSMO - Registering host zone...
  code: 0
  txhash: 6ED36B147851B555CE9C4A4D36D8C181D3951E2B3F08181475206291F8A79E33
OSMO - Registering validators...
  code: 0
  txhash: D8BD8392AEBEF26173EA30C50E43C72F04AA9466EB9E831E5FCE24A4544C7122
Done
```
### Check relayes paths
In `relayer` docker container shell, run
`rly paths list`
Output:
```
 0: stride-evmos         -> chns(✘) clnts(✘) conn(✘) (STRIDE<>evmos_9001-2)
 1: stride-gaia          -> chns(✘) clnts(✘) conn(✘) (STRIDE<>GAIA)
 2: stride-host          -> chns(✘) clnts(✘) conn(✘) (STRIDE<>HOST)
 3: stride-juno          -> chns(✘) clnts(✘) conn(✘) (STRIDE<>JUNO)
 4: stride-osmo          -> chns(✔) clnts(✔) conn(✔) (STRIDE<>OSMO)
 5: stride-stars         -> chns(✘) clnts(✘) conn(✘) (STRIDE<>STARS)
```
### Logs generated
Call: `dockernet/start_network.sh` -> `dockernet/src/create_logs.sh`

Out put in `dockernet/logs/state.log`:
```
LIST-HOST-ZONES STRIDE
host_zone:
- address: stride1hrzg27xh9zc25uufl8lm84z595zv8efsxl6cfr76kvg5xafzd8asx6lnz2
  bech32prefix: osmo
  blacklisted_validators: []
  chain_id: OSMO
  connection_id: connection-0
  delegation_account:
    address: osmo1cx04p5974f8hzh2lqev48kjrjugdxsxy7mzrd0eyweycpr90vk8q8d6f3h
    target: DELEGATION
  fee_account:
    address: osmo1n4r77qsmu9chvchtmuqy9cv3s539q87r398l6ugf7dd2q5wgyg9su3wd4g
    target: FEE
  halted: false
  host_denom: uosmo
  ibc_denom: ibc/ED07A3391A112B175915CD8FAF43A2DA8E4790EDE12566649D0C2F97716B8518
  last_redemption_rate: "1.000000000000000000"
  max_redemption_rate: "1.500000000000000000"
  min_redemption_rate: "0.900000000000000000"
  redemption_account:
    address: osmo1uy9p9g609676rflkjnnelaxatv8e4sd245snze7qsxzlk7dk7s8qrcjaez
    target: REDEMPTION
  redemption_rate: "1.000000000000000000"
  staked_bal: "0"
  transfer_channel_id: channel-0
  unbonding_frequency: "1"
  validators:
  - address: osmovaloper1uk4ze0x4nvh4fk0xm4jdud58eqn4yxhr6n3re8
    delegation_amt: "0"
    internal_exchange_rate: null
    name: oval1
    weight: "5"
  withdrawal_account:
    address: osmo10arcf5r89cdmppntzkvulc7gfmw5lr66y2m25c937t6ccfzk0cqqz2l6xv
    target: WITHDRAWAL
pagination:
  next_key: null
  total: "0"
```

### Run integration tests
Comment out all other chains and change to Osmosis [here](https://github.com/Stride-Labs/stride/blob/v10.0.0/dockernet/tests/run_all_tests.sh#L8) as
`CHAIN_NAME=OSMO TRANSFER_CHANNEL_NUMBER=0 $BATS $INTEGRATION_TEST_FILE`
And run
`make test-integration-docker`
Output:
```
bash ./dockernet/tests/run_all_tests.sh
integration_tests.bats
 ✓ [INTEGRATION-BASIC-OSMO] host zones successfully registered
 ✓ [INTEGRATION-BASIC-OSMO] ibc transfer updates all balances
 ✓ [INTEGRATION-BASIC-OSMO] liquid stake mint and transfer
 - [INTEGRATION-BASIC-OSMO] packet forwarding automatically liquid stakes (skipped: Packet forward liquid stake test is only run on GAIA and HOST)
 ✓ [INTEGRATION-BASIC-OSMO] tokens on OSMO were staked
 ✓ [INTEGRATION-BASIC-OSMO] redemption works
 ✓ [INTEGRATION-BASIC-OSMO] claimed tokens are properly distributed
 ✓ [INTEGRATION-BASIC-OSMO] rewards are being reinvested, exchange rate updating
 ✓ [INTEGRATION-BASIC-OSMO] rewards are being distributed to stakers

9 tests, 0 failures, 1 skippedOSMO
```

### Check host zones
`strided q stakeibc show-host-zone OSMO`
Output:
```
host_zone:
  address: stride1hrzg27xh9zc25uufl8lm84z595zv8efsxl6cfr76kvg5xafzd8asx6lnz2
  bech32prefix: osmo
  blacklisted_validators: []
  chain_id: OSMO
  connection_id: connection-0
  delegation_account:
    address: osmo1cx04p5974f8hzh2lqev48kjrjugdxsxy7mzrd0eyweycpr90vk8q8d6f3h
    target: DELEGATION
  fee_account:
    address: osmo1n4r77qsmu9chvchtmuqy9cv3s539q87r398l6ugf7dd2q5wgyg9su3wd4g
    target: FEE
  halted: false
  host_denom: uosmo
  ibc_denom: ibc/ED07A3391A112B175915CD8FAF43A2DA8E4790EDE12566649D0C2F97716B8518
  last_redemption_rate: "1.001933942971404911"
  max_redemption_rate: "1.500000000000000000"
  min_redemption_rate: "0.900000000000000000"
  redemption_account:
    address: osmo1uy9p9g609676rflkjnnelaxatv8e4sd245snze7qsxzlk7dk7s8qrcjaez
    target: REDEMPTION
  redemption_rate: "1.001936972647078130"
  staked_bal: "992123"
  transfer_channel_id: channel-0
  unbonding_frequency: "1"
  validators:
  - address: osmovaloper1uk4ze0x4nvh4fk0xm4jdud58eqn4yxhr6n3re8
    delegation_amt: "992123"
    internal_exchange_rate: null
    name: oval1
    weight: "5"
  withdrawal_account:
    address: osmo10arcf5r89cdmppntzkvulc7gfmw5lr66y2m25c937t6ccfzk0cqqz2l6xv
    target: WITHDRAWAL
```
### Check balance in redemption account on host chain
```
REDEMPTION_ICA_ADDR=$(strided q stakeibc show-host-zone OSMO | grep redemption_account -A 1 | grep address | awk '{print $2}')
osmosisd q bank balances $REDEMPTION_ICA_ADDR --denom uosmo --home dockernet/state/osmo1
```
Output:
```
balances:
- amount: "39"
  denom: uosmo
pagination:
  next_key: null
  total: "0"
```

# Source code 
- [RestoreInterchainAccount](https://github.com/Stride-Labs/stride/blob/v10.0.0/x/stakeibc/keeper/msg_server_restore_interchain_account.go#L16)
    1. GetHostZone
    2. GetConnection
    3. owner
    4. portID from owner
    5. [RegisterInterchainAccount](https://github.com/cosmos/ibc-go/blob/main/modules/apps/27-interchain-accounts/controller/keeper/account.go#L32)

# Test ICA CLI
- Demo ICA from Cosmos modified to run with latest version of relayer [here](https://github.com/nabaruns/interchain-accounts-demo/tree/nabaruns/update-scripts)
- Checkout script [here](https://github.com/nabaruns/stride/blob/nabaruns/stride-osmo-ica-test/dockernet/tests/run_ica_tests.sh)

Run:
`./dockernet/tests/run_ica_tests.sh`

Output:
```
-> Validator key 1: stride1uk4ze0x4nvh4fk0xm4jdud58eqn4yxhrt52vv7
-> Validator key 2: osmo1uk4ze0x4nvh4fk0xm4jdud58eqn4yxhrqyeqwq
-> Wallet key 1: stride1m9l358xunhhwds0568za49mzhvuxx9uxqj5hgp

-> Wallet key 2: osmo1m9l358xunhhwds0568za49mzhvuxx9uxtz8m2l

-> Wallet key 3: osmo10h9stc5v6ntgeygf5xf945njqq5h32r5e8nv6u
-> Check balance
balances:
- amount: "4994999900000"
  denom: ustrd
pagination:
  next_key: null
  total: "0"
balances:
- amount: "4994999900000"
  denom: uosmo
pagination:
  next_key: null
  total: "0"
balances:
- amount: "100000"
  denom: ustrd
pagination:
  next_key: null
  total: "0"
balances:
- amount: "100000"
  denom: uosmo
pagination:
  next_key: null
  total: "0"
balances: []
pagination:
  next_key: null
  total: "0"
-> Send tokens
  code: 0
  txhash: 4B4BAE55D24B50ED651782E595336BDF96DB4A12F88E6085240F045922A98A69
  code: 0
  txhash: A452A759C5CD37723DF44AB0AA1CFF5DDF81AD09B31370C1660F4DA4E4676BD1
  code: 32
  txhash: E7E3E27463E7E2E0401652B714AE70F229E4ED8C5D0157F1D6B8E27C815A94DB
-> Check balance again
balances:
- amount: "4994999900000"
  denom: ustrd
pagination:
  next_key: null
  total: "0"
balances:
- amount: "4994999900000"
  denom: uosmo
pagination:
  next_key: null
  total: "0"
balances:
- amount: "100000"
  denom: ustrd
pagination:
  next_key: null
  total: "0"
balances:
- amount: "100000"
  denom: uosmo
pagination:
  next_key: null
  total: "0"
balances: []
pagination:
  next_key: null
  total: "0"
-> Register an interchain account on behalf of stride1m9l358xunhhwds0568za49mzhvuxx9uxqj5hgp
  code: 0
  txhash: 490AD7D54AC92B2814DC00CA240EA7F223D643A6407193FBECF4F876DB515A48
-> Wait until the relayer has relayed the packet
-> Store the interchain account address by parsing the query result
osmo1erl4adm3d5klfn8sdfwqg3jf3tyzf7epsu6d5h3px0z8ng72245svsljr4
-> Query the interchain account balance on the host chain.
balances: []
pagination:
  next_key: null
  total: "0"
-> Send funds to the interchain account.
  code: 0
  txhash: 3B88FCD8A129F860500D19870F63FEDC23199CBA837FD6565EEA107EA88BC814
-> Wait until the relayer has relayed the packet
-> Query the balance once again and observe the changes
balances:
- amount: "10000"
  denom: uosmo
pagination:
  next_key: null
  total: "0"
-> Submit a staking delegation tx using the interchain account via ibc
  code: 0
  txhash: 1597EFA47C377F0C64240E06FFB3193A765422281566796F8F1FDB3F4D5F6CBA
-> Wait until the relayer has relayed the packet
-> Inspect the staking delegations on the host chain
delegation_responses:
- balance:
    amount: "5000000000"
    denom: uosmo
  delegation:
    delegator_address: osmo1uk4ze0x4nvh4fk0xm4jdud58eqn4yxhrqyeqwq
    shares: "5000000000.000000000000000000"
    validator_address: osmovaloper1uk4ze0x4nvh4fk0xm4jdud58eqn4yxhr6n3re8
- balance:
    amount: "1000"
    denom: uosmo
  delegation:
    delegator_address: osmo1erl4adm3d5klfn8sdfwqg3jf3tyzf7epsu6d5h3px0z8ng72245svsljr4
    shares: "1000.000000000000000000"
    validator_address: osmovaloper1uk4ze0x4nvh4fk0xm4jdud58eqn4yxhr6n3re8
pagination:
  next_key: null
  total: "0"
-> Submit a bank send tx using the interchain account via ibc
  code: 0
  txhash: 8BF51826D7B1BB15177DDB005F7E6432040B89EC99EDF00B2C4A669F6BA105B8
-> Wait until the relayer has relayed the packet
-> Query the interchain account balance on the host chain
balances:
- amount: "8000"
  denom: uosmo
pagination:
  next_key: null
  total: "0"
balances:
- amount: "1000"
  denom: uosmo
pagination:
  next_key: null
  total: "0"
```
