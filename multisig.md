# Multisig

A **multisig account** is an Umee account with a special key that can require more than one signature to sign transactions. This can be useful for increasing the security of the account or for requiring the consent of multiple parties to make transactions. Multisig accounts can be created by specifying:

* threshold number of signatures required
* the public keys involved in signing

To sign with a multisig account, the transaction must be signed individually by the different keys specified for the account. Then, the signatures will be combined into a multisignature which can be used to sign the transaction. If fewer than the threshold number of signatures needed are present, the resultant multisignature is considered invalid.

### Generate a Multisig key <a href="#generate-a-multisig-key" id="generate-a-multisig-key"></a>

```
umeed keys add --multisig=name1,name2,name3[...] --multisig-threshold=K new_key_name
```

`K` is the minimum number of private keys that must have signed the transactions that carry the public key's address as signer.

The `--multisig` flag must contain the name of public keys that will be combined into a public key that will be generated and stored as `new_key_name` in the local database. All names supplied through `--multisig` must already exist in the local database.

Unless the flag `--nosort` is set, the order in which the keys are supplied on the command line does not matter, i.e. the following commands generate two identical keys:

```
umeed keys add --multisig=p1,p2,p3 --multisig-threshold=2 multisig_address
umeed keys add --multisig=p2,p3,p1 --multisig-threshold=2 multisig_address
```

### Signing a transaction

#### Step 1: Create the multisig key

Let's assume that you have `test1` and `test2` and want to make a multisig account with `test3`.

First import the public keys of `test3` into your keyring

```
umeed keys add \
    test3 \
    --pubkey=osmopub1addwnpepqgcxazmq6wgt2j4rdfumsfwla0zfk8e5sws3p3zg5dkm9007hmfysxas0u2
```

Generate the multisig key with 2/3 threshold.

```
umeed keys add \
    multi \
    --multisig=test1,test2,test3 \
    --multisig-threshold=2
```

You can see its address and details using the following command

```
umeed keys show multi
```

Let's add 10 UMEE to the multisig wallet:

```
umeed tx bank send \
    test1 \
    umee1e0fx0q9meawrcq7fmma9x60gk35lpr4xk3884m \
    10000000uumee \
    --chain-id=umee-1 \
    --gas=auto \
    --fees=1000000uumee \
    --broadcast-mode=block
```

#### Step 2: Create the multisig transaction

We want to send 5 UMEE from our multisig account to `umee1rgjxswhuxhcrhmyxlval0qa70vxwvqn2e0srft`.

```
umeed tx bank send \
    umee1rgjxswhuxhcrhmyxlval0qa70vxwvqn2e0srft \
    umee157g6rn6t6k5rl0dl57zha2wx72t633axqyvvwq \
    5000000uumee \
    --gas=200000 \
    --fees=1000000uumee \
    --chain-id=umee-1 \
    --generate-only > unsignedTx.json
```

The file `unsignedTx.json` contains the unsigned transaction encoded in JSON.

#### Step 3: Sign individually

Sign with `test1` and `test2` and create individual signatures.

```
umeed tx sign \
    unsignedTx.json \
    --multisig=umee1e0fx0q9meawrcq7fmma9x60gk35lpr4xk3884m \
    --from=test1 \
    --output-document=test1sig.json \
    --chain-id=umee-1
```

```
umeed tx sign \
    unsignedTx.json \
    --multisig=umee1e0fx0q9meawrcq7fmma9x60gk35lpr4xk3884m \
    --from=test2 \
    --output-document=test2sig.json \
    --chain-id=umee-1
```

#### Step 4: Create multisignature

Combine signatures to sign transaction.

```
umeed tx multisign \
    unsignedTx.json \
    multi \
    test1sig.json test2sig.json \
    --output-document=signedTx.json \
    --chain-id=umee-1
```

The TX is now signed

#### Step 5: Broadcast transaction

```
umeed tx broadcast signedTx.json \
    --chain-id=umee-1 \
    --broadcast-mode=block
```

\
\
\
\


\
\


\
\
\
\
\
