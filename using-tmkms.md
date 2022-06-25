---
description: >-
  The Tendermint Key Management System (or TMKMS) should be used by any
  validator currently or intending to be in the active validator set. This
  application mitigates the risk of double-signing and prov
---

# Using TMKMS

### Prepare TMKMS Dependencies <a href="#prepare-tmkms-dependencies" id="prepare-tmkms-dependencies"></a>

Start by opening the node you intend to run TMKMS (not the node you validate on) and install the following dependencies:\


**Rust**

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
```

#### GCC - Ubuntu

```
sudo apt update
sudo apt install git build-essential ufw curl jq snapd --yes
```

#### GCC - MAC&#x20;

```
brew install gcc
```

**Libusb - Ubuntu**

```
apt install libusb-1.0-0-dev
```

**Libusb - Mac**

```
brew install libusb
```

If on x86\_64 architecture:

```
export RUSTFLAGS=-Ctarget-feature=+aes,+ssse3
```

### Setup TMKMS <a href="#setup-tmkms" id="setup-tmkms"></a>

In this example, we will be compiling from the github source code using the `--features=softsign` flag, however you may use `--features=yubihsm` if you want to use a yubikey (ledger support is not working properly at the moment, and this guide will not go into using yubihsm).

```
cd $HOME
git clone https://github.com/iqlusioninc/tmkms.git
cd $HOME/tmkms
cargo install tmkms --features=softsign
tmkms init config
tmkms softsign keygen ./config/secrets/secret_connection_key

```

Now we will transfer your validator private key from your validator to your VM running TMKMS. You can do this manually or though scp. I will use scp in this example (the validator has the IP of 123.456.32.123):

```
scp user@123.456.32.123:~/.umeed/config/priv_validator_key.json ~/tmkms/config/secrets

```

Then, import the private validator key into tmkms:

```
tmkms softsign import $HOME/tmkms/config/secrets/priv_validator_key.json $HOME/tmkms/config/secrets/priv_validator_key
```

Please note at this point, you could delete the `priv_validator_key.json` from both your validator node and tmkms node and store it safely offline in case of an emergency. This newly created `priv_validator_key` will be what TMKMS will use to sign for your validator.

Now, modify the `tmkms.toml` file

```
nano $HOME/tmkms/config/tmkms.toml

```

In this example, my validator has the IP address of 123.456.32.123 and we will be using port 26659 to feed the validator key to the validator. We will also be using chain\_id `umeed-1`, but if you are doing this on the testnet be sure to use `umeemania-1` instead:

```
# Tendermint KMS configuration file

## Chain Configuration

### Cosmos Hub Network

[[chain]]
id = "umee-1"
key_format = { type = "bech32", account_key_prefix = "umeepub", consensus_key_prefix = "umeevalconspub" }
state_file = "/root/tmkms/config/state/priv_validator_state.json"

## Signing Provider Configuration

### Software-based Signer Configuration

[[providers.softsign]]
chain_ids = ["umee-1"]
key_type = "consensus"
path = "/root/tmkms/config/secrets/priv_validator_key"

## Validator Configuration

[[validator]]
chain_id = "umee-1"
addr = "tcp://123.456.32.123:26659"
secret_key = "/root/tmkms/config/secrets/secret_connection_key"
protocol_version = "v0.34"
reconnect = true
```

Now, modify your validators `config.toml` to use the port you selected in the `tmkms.toml` file:

```
nano $HOME/.umeed/config/config.toml

```

```
priv_validator_laddr = "tcp://0.0.0.0:26659"
```

It is also recommended to comment out the `priv_validator_key_file` line and the `priv_validator_state_file` line:

```
# Path to the JSON file containing the private key to use as a validator in the consensus protocol
# priv_validator_key_file = "config/priv_validator_key.json"

# Path to the JSON file containing the last sign state of a validator
# priv_validator_state_file = "data/priv_validator_state.json
```

Next, stop the validator. Move back to your VM running TMKMS and start it:

```
tmkms start -c $HOME/tmkms/config/tmkms.toml

```

You will see error logs like the following:

```
2022-03-08T23:42:37.926816Z  INFO tmkms::commands::start: tmkms 0.11.0 starting up...
2022-03-08T23:42:37.926968Z  INFO tmkms::keyring: [keyring:softsign] added consensus Ed25519 key: umeevalconspub1zcjduepq2qfkp3ahrhaafzuqglme9mares0eluj58xr6cy7c37cdmzq0eecqk0yehr
2022-03-08T23:42:37.927216Z  INFO tmkms::connection::tcp: KMS node ID: 948f8fee83f7715f99b8b8a53d746ef00e7b0d9e
2022-03-08T23:42:37.929454Z ERROR tmkms::client: [umee-1@tcp://123.456.32.123:26659] I/O error: Connection refused (os error 111)
2022-03-08T23:42:38.929746Z  INFO tmkms::connection::tcp: KMS node ID: 948f8fee83f7715f99b8b8a53d746ef00e7b0d9e
2022-03-08T23:42:38.931428Z ERROR tmkms::client: [umee-1@tcp://123.456.32.123:26659] I/O error: Connection refused (os error 111)
2022-03-08T23:42:39.931729Z  INFO tmkms::connection::tcp: KMS node ID: 948f8fee83f7715f99b8b8a53d746ef00e7b0d9e
2022-03-08T23:42:39.932417Z ERROR tmkms::client: [umee-1@tcp://123.456.32.123:26659] I/O error: Connection refused (os error 111)
2022-03-08T23:42:40.932732Z  INFO tmkms::connection::tcp: KMS node ID: 948f8fee83f7715f99b8b8a53d746ef00e7b0d9e
2022-03-08T23:42:40.933425Z ERROR tmkms::client: [umee-1@tcp://123.456.32.123:26659] I/O error: Connection refused (os error 111)

```

Now, start your umee validator on the validator node:

```
umeed start
```

Your TMKMS node will now show logs like the following:

```
2022-03-08T23:46:06.208451Z  INFO tmkms::connection::tcp: KMS node ID: 948f8fee83f7715f99b8b8a53d746ef00e7b0d9e
2022-03-08T23:46:06.210568Z  INFO tmkms::session: [umee-1@tcp://164.92.136.160:26659] connected to validator successfully
2022-03-08T23:46:06.210604Z  WARN tmkms::session: [umee-1@tcp://164.92.136.160:26659]: unverified validator peer ID! (ba44dd36899602e255b04e3608e5ef0fe4bc5f5b)
2022-03-08T23:46:15.929787Z  INFO tmkms::session: [umee-1@tcp://164.92.136.160:26659] signed PreCommit:<nil> at h/r/s 3399910/0/2 (0 ms)
2022-03-08T23:46:17.344579Z  INFO tmkms::session: [umee-1@tcp://164.92.136.160:26659] signed PreCommit:<nil> at h/r/s 3399911/0/2 (0 ms)
2022-03-08T23:46:22.367627Z  INFO tmkms::session: [umee-1@tcp://164.92.136.160:26659] signed PreCommit:<nil> at h/r/s 3399912/0/2 (0 ms)
2022-03-08T23:46:27.409777Z  INFO tmkms::session: [umee-1@tcp://164.92.136.160:26659] signed PreCommit:<nil> at h/r/s 3399913/0/2 (0 ms)
2022-03-08T23:46:32.442300Z  INFO tmkms::session: [umee-1@tcp://164.92.136.160:26659] signed PreCommit:<nil> at h/r/s 3399914/0/2 (0 ms)
2022-03-08T23:46:37.452162Z  INFO tmkms::session: [umee-1@tcp://164.92.136.160:26659] signed PreCommit:<nil> at h/r/s 3399915/0/2 (0 ms)

```

You should now be signing blocks! If you cancel the TMKMS process, you will no longer sign blocks and will stop syncing. If you restart the TMKMS process, your validator node will continue to sync from where it left off.

### Final Notes <a href="#final-notes" id="final-notes"></a>

Please note that this is a bare minimum setup. More robust settings such as setting up a firewall to only allow your TMKMS node to get through the priv\_validator\_laddr port would make your validator even more secure.
