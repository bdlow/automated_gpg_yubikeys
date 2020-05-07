# README

<!-- TOC -->

- [README](#readme)
  - [Purpose](#purpose)
    - [Process Summary](#process-summary)
      - [Initial setup](#initial-setup)
  - [Usage](#usage)
    - [Usage Summary](#usage-summary)
    - [Usage Detail](#usage-detail)
      - [1. Setup](#1-setup)
      - [2. Create OpenPGP key](#2-create-openpgp-key)
      - [3. Copy to a YubiKey](#3-copy-to-a-yubikey)
        - [Change the YubiKey OpenPGP card Admin PIN and user PIN](#change-the-yubikey-openpgp-card-admin-pin-and-user-pin)
      - [4. Cleanup](#4-cleanup)
  - [Example Use of Key](#example-use-of-key)
  - [Appendix](#appendix)
    - [Verify the YubiKey copy](#verify-the-yubikey-copy)

<!-- /TOC -->

## Purpose

This script automates the creation of a GPG key and provisioning to one or more Yubikeys.

### Process Summary

#### Initial setup

The `make_gpg_keys` script assists to:

- Create OpenGPG keys
- Copy those keys to one or more YubiKey Security Keys

## Usage

### Usage Summary

1. setup
    - create and switch to a temporary GPG home directory
    - set a temporary GPG key passphrase environment variable
1. use the `make_gpg_keys -g` script to create a GPG key
    - copy the public key somewhere safe
1. repeat for each card:
    - use the `make_gpg_keys -y` script to copy the GPG key to a YubiKey
    - card holder to change the PIN to their own personal passphrase
1. cleanup: securely erase the temporary directory

### Usage Detail

#### 1. Setup

Install required software:

- [GnuPG](https://www.gnupg.org/) 2.2 or later
- [YubiKey Manager](https://www.yubico.com/products/services-software/download/yubikey-manager/) (`ykman`)

Starting from where you've checked out this repo:

```shell
# set an environment variable to point to the script
% MAKE_GPG_KEYS=$PWD/make_gpg_keys

# create and switch to a temporary GPG home directory
% export GNUPGHOME=$(mktemp -d)
% cd $GNUPGHOME
tmp.e9hcfpTs%

# check dependencies
tmp.e9hcfpTs% $MAKE_GPG_KEYS -d && echo ok
ok

# example failure:
tmp.e9hcfpTs% $MAKE_GPG_KEYS -d && echo ok
[tmp/make_gpg_keys] gpg v2 not found
[tmp/make_gpg_keys] failed: 1
tmp.e9hcfpTs%
```

#### 2. Create OpenPGP key

```shell
# create a temporary key passphrase
tmp.e9hcfpTs% export PASSPHRASE=$($MAKE_GPG_KEYS -p)

# generate a key
tmp.e9hcfpTs% $MAKE_GPG_KEYS -g 'Firstname Lastname','first.last@example.com'
[make_gpg_keys] creating primary key
[make_gpg_keys] creating subkey for key ID [69EE5DCC9C00F7A843739CC98B5A276A23558357]
[make_gpg_keys] exporting public key to 69EE5DCC9C00F7A843739CC98B5A276A23558357.pub
[make_gpg_keys] done
tmp.e9hcfpTs%

# copy the public key somewhere permanent (not sensitive)
tmp.e9hcfpTs% cp 69EE5DCC9C00F7A843739CC98B5A276A23558357.pub ~/
```

- *it is essential to keep the public key available; it is not possible to extract the public key from the secret key once it is moved to an OpenPGP card*
  - in other words, if the *public key* is lost the private key on the card will be rendered useless and encrypted data will be unrecoverable

#### 3. Copy to a YubiKey

- repeat this step for as many YubiKeys as desired

```shell
# connect the first YubiKey and check we can see it, making a note of the serial number
tmp.e9hcfpTs% $MAKE_GPG_KEYS -l
YubiKey 4 [CCID] Serial: 7306336

# copy the GPG key generated in the previous step to the YubiKey
# - this will reset the YubiKey and clear any existing data

tmp.e9hcfpTs% $MAKE_GPG_KEYS -y 69EE5DCC9C00F7A843739CC98B5A276A23558357,7306336
[make_gpg_keys] Looking for security key with serial number [7306336]
Device type: YubiKey 4
Serial number: 7306336
...
[make_gpg_keys] resetting YubiKey [7306336]
WARNING! This will delete all stored OpenPGP keys and data and restore factory settings? [y/N]: y
Resetting OpenPGP data, don't remove your YubiKey...
Success! All data has been cleared and default PINs are set.
PIN:         123456
Reset code:  NOT SET
Admin PIN:   12345678
[make_gpg_keys] setting CCID mode
[make_gpg_keys] starting gpg agent
[make_gpg_keys] looking for GPG key [69EE5DCC9C00F7A843739CC98B5A276A23558357]
[make_gpg_keys]   ID: Firstname Lastname <first.last@example.com>
[make_gpg_keys] configuring GPG card
[make_gpg_keys] copying keys to card
[make_gpg_keys] configuring touch policy
[make_gpg_keys] done
tmp.e9hcfpTs%
```

##### Change the YubiKey OpenPGP card Admin PIN and user PIN

- "Admin PIN" must be at least 8 characters long and the "PIN" must be at least 6; they may be any alphanumeric character (i.e. not just digits)
- the initial values are 12345678 and 123456
- for the purposes of this work both Admin PIN and PIN should be known only to one person; they may be the same

```shell
tmp.e9hcfpTs% $MAKE_GPG_KEYS -a
[make_gpg_keys] changing Admin PIN
# pin entry dialog
[make_gpg_keys] done

tmp.e9hcfpTs% $MAKE_GPG_KEYS -u
[make_gpg_keys] changing User PIN
# pin entry dialog
[make_gpg_keys] done

```

#### 4. Cleanup

- once this step is completed no further copies of the key can be made

```shell
tmp.e9hcfpTs% rm -rf $GNUPGHOME
```

## Example Use of Key

```shell
# create a new GPG home, and import the public key
% export GNUPGHOME=$(mktemp -d)
% cd $GNUPGHOME

tmp.SqBloihz% gpg --import /tmp/69EE5DCC9C00F7A843739CC98B5A276A23558357.pub
gpg: key 8B5A276A23558357: public key "Firstname Lastname <first.last@example.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1

# confirm the key was imported, and that the private key is not present
tmp.SqBloihz% gpg --list-keys
/var/folders/g9/h1f54jhn09s0djhl3dtg8mz40000gp/T/tmp.SqBloihz/pubring.kbx
-------------------------------------------------------------------------
pub   rsa2048 2019-10-02 [C]
      69EE5DCC9C00F7A843739CC98B5A276A23558357
uid           [ unknown] Firstname Lastname <first.last@example.com>
sub   rsa2048 2019-10-02 [E]

tmp.SqBloihz% gpg --list-secret-keys
tmp.SqBloihz%

# encrypt a message using the public key
tmp.SqBloihz% echo "hello" | gpg --armor --encrypt --recipient 69EE5DCC9C00F7A843739CC98B5A276A23558357 --trust-model always > message.gpg


# connect the YubiKey if it's not already connected

tmp.SqBloihz% gpg --card-status
Reader ...........: Yubico Yubikey 4 CCID
Application ID ...: D2760001240102010006073063360000
...
Signature key ....: 69EE 5DCC 9C00 F7A8 4373  9CC9 8B5A 276A 2355 8357
      created ....: 2019-10-02 02:05:08
Encryption key....: 1B30 FBD7 61EA 4190 E959  384E E721 0CAF 5A2B 51ED
      created ....: 2019-10-02 02:05:09
Authentication key: [none]
General key info..: pub  rsa2048/8B5A276A23558357 2019-10-02 Firstname Lastname <first.last@example.com>
sec>  rsa2048/8B5A276A23558357  created: 2019-10-02  expires: never
                                card-no: 0006 07306336
ssb>  rsa2048/E7210CAF5A2B51ED  created: 2019-10-02  expires: never
                                card-no: 0006 07306336


# confirm the secret key is now available
tmp.SqBloihz% gpg --list-secret-keys
/var/folders/g9/h1f54jhn09s0djhl3dtg8mz40000gp/T/tmp.SqBloihz/pubring.kbx
-------------------------------------------------------------------------
sec>  rsa2048 2019-10-02 [C]
      69EE5DCC9C00F7A843739CC98B5A276A23558357
      Card serial no. = 0006 07306336
uid           [ unknown] Firstname Lastname <first.last@example.com>
ssb>  rsa2048 2019-10-02 [E]

# the ">" in the `sec` and `ssb` records indicates the key is on a smart card

# decrypt the message
# - some environments require setting the GPG terminal environment variable
tmp.SqBloihz% export GPG_TTY=$(tty)

# - gpg will pop up a passphrase entry dialog, and you'll need to touch the key
tmp.SqBloihz% gpg --decrypt < message.gpg
  # pin entry dialog
  # YubiKey touch
gpg: encrypted with 2048-bit RSA key, ID E7210CAF5A2B51ED, created 2019-10-02
      "Firstname Lastname <first.last@example.com>"
hello
```

## Appendix

### Verify the YubiKey copy

Compare the gpg and card fingerprints:

- gpg pub = card Signature
- gpg sub = card Encryption

```shell
tmp.e9hcfpTs% gpg --fingerprint --fingerprint
/var/folders/g9/h1f54jhn09s0djhl3dtg8mz40000gp/T/tmp.e9hcfpTs/pubring.kbx
-------------------------------------------------------------------------
pub   rsa2048 2019-10-02 [C]
      69EE 5DCC 9C00 F7A8 4373  9CC9 8B5A 276A 2355 8357
uid           [ultimate] Firstname Lastname <first.last@example.com>
sub   rsa2048 2019-10-02 [E]
      1B30 FBD7 61EA 4190 E959  384E E721 0CAF 5A2B 51ED

tmp.e9hcfpTs% gpg --card-status
Reader ...........: Yubico Yubikey 4 CCID
Application ID ...: D2760001240102010006073063360000
Version ..........: 2.1
Manufacturer .....: Yubico
Serial number ....: 07306336
Name of cardholder: Firstname Lastname
Language prefs ...: [not set]
Sex ..............: unspecified
URL of public key : [not set]
Login data .......: first.last@example.com
Signature PIN ....: forced
Key attributes ...: rsa2048 rsa2048 rsa2048
Max. PIN lengths .: 127 127 127
PIN retry counter : 3 0 3
Signature counter : 0
Signature key ....: 69EE 5DCC 9C00 F7A8 4373  9CC9 8B5A 276A 2355 8357
      created ....: 2019-10-02 02:05:08
Encryption key....: 1B30 FBD7 61EA 4190 E959  384E E721 0CAF 5A2B 51ED
      created ....: 2019-10-02 02:05:09
Authentication key: [none]
General key info..: pub  rsa2048/8B5A276A23558357 2019-10-02 Firstname Lastname <first.last@example.com>
sec   rsa2048/8B5A276A23558357  created: 2019-10-02  expires: never
ssb   rsa2048/E7210CAF5A2B51ED  created: 2019-10-02  expires: never
```

- if the `gpg --card-status` command raises an error:
  - ensure there are no other gpg agents running on the system
  - try again
  - if problems persist re-plug the YubiKey
