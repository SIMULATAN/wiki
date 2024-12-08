---
title: GPG & Nitrokey
description: 
published: true
date: 2024-12-08T10:36:11.377Z
tags: 
editor: markdown
dateCreated: 2024-12-08T10:36:11.377Z
---

# Creating and moving
https://docs.nitrokey.com/nitrokeys/features/openpgp-card/openpgp-keygen-backup
https://mikeross.xyz/create-gpg-key-pair-with-subkeys/

Export the public key on your primary machine:
*this is needed in order to use the secret key from the Nitrokey on another machine*
```bash
gpg --output public.gpg --export <PRIMARY_ID>
```

Now, copy `public.gpg` to your second machine and `gpg --import public.gpg` it.
`gpg --list-secret-keys` should now show something like this:

```
sec>  ed25519 2024-12-08 [SC]
      <PRIMARY_ID>
      Card serial no. = 000F 74630250
uid           [ultimate] Your Name <your@mail.com>
ssb>  cv25519 2024-12-08 [E]
ssb>  ed25519 2024-12-08 [A]
```
The top one (`[SC]`) is your primary key. All of them are stored on your Nitrokey and will be unavailable if you eject the hardware (stubs will remain).

Now, create a new signing subkey for just this machine:

```bash
gpg --edit-key --expert <PRIMARY_ID>

gpg> addkey
Your selection? 10
# complete the rest as desired
# remember the new `ssb` ID with the `usage: S` (SUBKEY_ID)
gpg> save

# export the new subkey
gpg --output machine.secsub.gpg --export-secret-subkeys <SUBKEY_ID>

# delete key from smart card / cleanup
gpg --edit-key --expert <PRIMARY_ID>
gpg> key <SUBKEY_ID_INDEX>
gpg> delkey
gpg> save
```
With this, the key shouldn't know of any subkey, but the `machine.secsub.gpg` contains exactly one.
Now, eject your Nitrokey and run `gpg --import machine.secsub.gpg`.
`gpg --list-secret-keys` should now print an entry for your `PRIMARY_ID` *with* the signing key.

## Setup git
```bash
git config --global user.signingkey <PRIMARY_ID> # yes, the primary id. it'll automatically pick the right key.
git config --global user.email "your@mail.com"
```

## Setup GitHub
Github is a piece of shit. This will take a little while to do.