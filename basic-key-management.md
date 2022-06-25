---
description: Create, import, export and delete keys using the CLI keyring.
---

# Basic Key Management

### Create a new key <a href="#create-a-new-key" id="create-a-new-key"></a>

You can create a new key with the name default using the following command&#x20;

```
keys add <wallet_name> defualt
```

Example:&#x20;

```
umeed keys add Default
- name: default
  type: local 
  address: umeeizrozwvkagp7ku6lflfzv8g5757e9drogdj74wd 
  pubkey: '{"@type":"/cosmos.crypto.secp256k1.Pubkey", "key":"AwDAQYHenwe 7HJ137QLGgrxx9ZPdPSMWKRSKEXLF23ZA"} 
  mnemonic:
**Important** write this mnemonic phrase in a safe place. It is the only way to recover your account if you ever forget your password.
arctic immense assume lock dinner scan goose magnet alpha around indoor exotic tape memory hybrid rice setup brisk betray iconmmense worth flock sponsor root@ubuntu-4gb-ash-1: #
```

The key comes with a "mnemonic phrase", which is serialized into a human-readable 24-word mnemonic. User can recover their associated addresses with the mnemonic phrase.

**WARNING**

It is important that you keep the mnemonic for address secure, as there is **no way** to recover it. You would not be able to recover and access the funds in the wallet if you forget the mnemonic phrase

### Restore existing key by seed phrase <a href="#restore-existing-key-by-seed-phrase" id="restore-existing-key-by-seed-phrase"></a>

You can restore an existing key with the mnemonic using the following command

```
keys add <key_name> --recover
```

Example:&#x20;

```
umeed keys add default_restore --recover
> Enter your bip39 mnemonic
```

### List your keys

Multiple keys can be created when needed. You can list all keys saved under the storage path using the following command.&#x20;

```
umeed keys list 
```

### Retrieve key information

You can retrieve key information by searching that keys name.

```
umeed keys show <key_name>
```

To retrieve a keys account address and its public key:

```
umeed keys show default --bech acc
```

To retrieve a keys validator address and its public key:

```
umeed keys show default --bech val
```

To retrieve a keys Consensus nodes address and its public key:

```
umeed keys show default --bech cons
```

### Delete a key

You can delete a key in your storage path.

```
umeed keys delete <key_name>
```

### Export private keys

You can export and backup your key by using the `export` subcommand:

```
umeed keys export <key_name>
```

### The keyring-backend option

Interacting with a node requires a public-private key pair. Keyring is the place holding the keys. The keys can be stored in different locations with specified backend type.

```
umeed keys [subcommands] --keyring-backend [backend type]
```

#### os backend

The default `os` backend stores the keys in operating system's credential sub-system, which are comfortable to most users, yet without compromising on security.

Here is a list of the corresponding password managers in different operating systems:

* macOS (since Mac OS 8.6): [Keychain](https://support.apple.com/en-gb/guide/keychain-access/welcome/mac)
* Windows: [Credentials Management API](https://docs.miosmosoft.com/en-us/windows/win32/secauthn/credentials-management)
* GNU/Linux:
  * [libsecret](https://gitlab.gnome.org/GNOME/libsecret)
  * [kwallet](https://api.kde.org/frameworks/kwallet/html/index.html)

#### file backend

The `file` backend stores the encrypted keys inside the app's configuration directory. A password entry is required every time a user access it, which may also occur multiple times of repeated password prompts in one single command.

#### test backend

The `test` backend is a password-less variation of the `file` backend. It stores unencrypted keys inside the app's configuration directory. It should only be used in testing environments and never be used in production.
