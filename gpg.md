---
title: GPG & Nitrokey
description: 
published: true
date: 2024-12-08T10:43:35.671Z
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
Github is a piece of shit. It neither allows you to import multiple subkeys for one primary key nor the primary key by itself (and verifying that the commit key is in the chain-of-trust).

Therefore, we have to merge ALL our keys in ONE CENTRAL PLACE. Oh boy.

Anyway, do a quick `gpg --armor --export <SUBKEY_ID>! > machine.public.gpg` and copy all files to one machine.

Now, run `gpg --homedir . --import machine.public.gpg` for every `machine` you have.
`gpg --homedir . --list-keys` should now show multiple `[S]` subkeys.
Finally, export the primary key: `gpg --homedir . --armor --export <PRIMARY_ID>` and [upload the output to GitHub](https://github.com/settings/keys).

This is a HORRIBLE way to do this. Should one machine get compromised, you must delete the one single "monokey" and RECREATE the ENTIRE KEYCHAIN (or just `key n` & `delkey` in the existing gpg homedir) and UPLOAD IT AGAIN. Please let GitHub know that some of their users may have multiple computers and would love to maintain their integrity.