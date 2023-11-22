---
title: "Create a Subnet"
---

# Create a Subnet

Before you create your first subnet, we strongly recommend that you follow the below order:
1. First create a subnet on your local, and develop and test your incentive mechanism on the local subnet. 
2. After you are satisfied with it, next create a subnet on the Bittensor testchain, and test and debug your incentive mechanism on this testchain subnet. 
3. Finally, only after you completed the above steps, create a subnet on the Bittensor mainchain. 

:::tip Minimum required TAO 
Creating a subnet on mainnet is competitive and the cost is determined by the rate at which new networks are being registered onto the chain. By default you must have at least 100 TAO in your owner wallet to create a subnetwork. However the exact amount will fluctuate based on demand.
:::

## Prerequisites

To create a subnet, whether locally or on testchain or mainchain, make sure that:
- You [installed Bittensor](../getting-started/installation.md). 
- You have already [created a wallet or know how to create one](../getting-started/wallets.md#creating-a-local-wallet). 

### Creating a subnet

 The code below shows how to get the current price of creating a subnetwork.

```bash dark
btcli subnet lock_cost
>> Subnet lock cost: τ100.000000000
```

Use the `btcli subnet create` subcommand to register a new subnet.

```bash dark
# Run the register subnetwork command on the main chain.
btcli subnet create
>> Enter wallet name (default): owner # Enter your owner wallet name
>> Enter password to unlock key: # Enter your wallet password.
>> Register subnet? [y/n]: <y/n> # Select yes (y)
>> Registering subnet...
✅ Registered subnetwork with netuid: 1 # Your subnet netuid will show here, save this for later.
```




## Register as a subnet owner

Prior to mining TAO, miners must attain a UID slot within one of Bittensor's sub-networks via a competition. This step is called registration. At the time of writing, there are 1024 UIDs available on Subnetwork 1 and 4096 on Subnetwork 3.

```bash dark
uids
    netuid 1: [ 0, 1, 2, ... 1022, 1023 ]
    netuid 3: [ 0, 1, 2, ... 4094, 4095 ]
```

TAO Recycle Registration

```bash
btcli subnet recycle_register --netuid SELECTED_NETUID --wallet.name YOUR_COLDKEY --wallet.hotkey YOUR_HOTKEY
```

Once the registration cost has been paid, the miner enters the network by replacing an older underperforming miner and can now [mine](./run-a-subnet-miner.md) themselves from that slot.


### Pow

Proof-of-Work (POW) registrations require miners to solve a SHA256 hashing problem before winning a UID. This route is recommmended for miners contributing raw compute power to Bittensor or don't have a previous token supply.
```bash dark
btcli subnet register
      --netuid SUBNETWORK_NETUID_TARGET
      --wallet.name YOUR_COLDKEY_NAME
      --wallet.hotkey YOUR_HOTKEY_NAME
```
!> Coldkeys
The POW registration does not requires only your `coldkeypub.txt` and `hotkey`. It is also possible to sign the registration extrinsic payload by a separate key.


### Recycle registration

Recycle registrations allow a miner to recycle TAO back into the inflation mechanism (to be passed through the incentive mechanism at a later date) in exchange for a UID on a subnetwork. Recycle registrations cost TAO to execute but takes less time to activate than POW registration. They recommended for miners seeking to attain slots quickly and who already have a small amount of TAO at their disposal.
```bash dark
btcli subnet recycle_register
      --netuid SELECTED_NETUID
      --wallet.name YOUR_COLDKEY
      --wallet.hotkey YOUR_HOTKEY
```


### Cubit

It is highly recommended that you use a Nvidia GPU to register for a faster hash rate. To use your GPU during the hashing problem you must [install Cubit](https://github.com/opentensor/cubit).

```bash dark
python3 -m pip install bittensor[cubit]
```

Once installed you can specify which GPU you are running your registration over or `--all` of them.
```bash dark
btcli subnet register
      --cuda
      --all
```


### Cost updates

POW and the recycle regsistration cost are mutually adaptive, updating their costs on an `adjustment interval` so that the number of registrations over that interval remain constanct, i.e. 3 registrations per 100 blocks. Below is pseudo code for the update conditions.
```python numbered dark title=subtensor/pallets/subtensor/src/block_step link=https://github.com/opentensor/subtensor/pallets/subtensor/src/block_step.rs
if regs > target and recycle_regs > pow_regs:
      burn_cost *= ( n_burn_regs + target_regs ) / (2 * target_regs)

elif regs > target and recycle_regs <= pow_regs:
      pow_difficulty *= ( n_pow_regs + target_regs ) / (2 * target_regs)

elif regs <= target and recycle_regs > pow_regs:
      burn_difficulty *= ( n_burn_regs + target_regs ) / (2 * target_regs)

elif regs <= target and recycle_regs <= pow_regs:
      pow_difficulty *= ( n_pow_regs + target_regs ) / (2 * target_regs)
```

To view the current subnetwork difficulty on each subnet run `btcli subnets list`
```bash dark
btcli subnets list
NETUID  NEURONS  MAX_N   DIFFICULTY  TEMPO  CON_REQ  EMISSION  BURN(τ)
1       691    1.02 K   198.08 T    99     None     28.44%   τ4.75710
3      4096    4.10 K   320.81 T    99     None     71.56%   τ1.00000
    DIFFICULTY: Current proof of work difficulty
    BURN: Current cost to register a key via recycle registration.
```


### Inspecting UIDs

After you obtain a slot you can view the performance of you registered wallet by running:
```bash
btcli wallet overview --netuid
```

After providing your wallet name at the prompt, you will see the following output:

| Parameter         | Value | Description |
| :---------------- | :------: | ----: |
| COLDKEY        |   my_coldkey   | The name of the coldkey associated with the won slot. |
| HOTKEY      | my_first_hotkey      |    The name of the hotkey associated with the won slot.                          |
| UID         | 5                    |    The index of the uid out of available uids.                                   |
| ACTIVE      | True                 |    Whether or not the uid is considered active.                                  |
| STAKE(τ)    | 71.296               |    The amount of stake on this miner.                                            |
| RANK        | 0.0629               |    This miner's absolute ranking according to validators on the network.         |
| TRUST       | 0.2629               |    This miner's trust as a proportion of validators on the network.              |
| CONSENSUS   | 0.89                 |    This miner's aggregate consensus score.                                       |
| INCENTIVE   | 0.029                |    This miner's incentive, TAO emission attained via mining.                     |
| DIVIDENDS   | 0.001                |    This miner's dividends, TAO emission attained via validating.                 |
| EMISSION    | 29_340_153           |    This miner's total emission in RAO ( 10^-9 TAO ) per block.                   |
| VTRUST      | 0.96936              |    This validators trust score as a validator.                                   |
| VPERMIT     | *                    |    Whether this miner is considered active for validating on this subnetwork.    |
| UPDATED     | 43                   |    Blocks since this miner set weights on the chain.                             |
| AXON        | 131.186.56.85:8091   |    The entrypoint advertised by this miner on bittensor blockchain.              |
| HOTKEY_SS58 | 5F4tQyWr...          |    The raw ss58 encoded address of the miner's hotkey.                           |



### Difficulty adjustment 

The POW and Recycle difficulties are adaptively adjusted every 100 blocks based on the following 4 cases.


    1. Registrations exceed the target and there were more recycle registrations than pow registrations?
        `burn_cost = burn_cost * ( burn_regs_this_interval + target_regs ) / 2 * target_regs`

    2. Registrations exceed the target and there were not more recycle registrations than pow registrations?
        `pow_difficulty = pow_difficulty * ( pow_regs_this_interval + target_regs ) / 2 * target_regs`

    3. Registrations do not exceed the target and there were more recycle registrations than pow registrations?
        `burn_difficulty = pow_difficulty * ( regs_this_interval + target_regs ) / 2 * target_regs`

    4. Registrations do not exceed the target and there were not more recycle registrations than pow registrations?
        `pow_difficulty = pow_difficulty * ( regs_this_interval + target_regs ) / 2 * target_regs`



