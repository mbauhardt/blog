---
title: "A Better GPG Keychain"
date: 2017-03-03
tags: ["gpg"]
series: ["Security"]
---

## Background

My personal goal is to have a GPG keychain i use on more than one machine. I don't want to loose all my signatures in case i lost my key.
After reading the internet i figure it out how can i achieve a better GPG keychain.

The approach of a master keypair and a bunch of subkeys will minimize the damage of the worst case (loosing my keypair). So i would like to explain what the benefit of subkeys are and how to generate subkeys.

In general if you have your keypair on your laptop and this laptop is stolen by an attacker then you want to revoke your key and publish the revoked key on a keyserver. And that means that you will loose all your signatures you maybe collected in the past few years. Subkeys can help you to minimze the damage in case you will loose your key but it will not prevent the attacker to use your stolen private subkey to decrypt messages which was encrypted with the related public key. Also signatures can be done with that stolen key but can not be verified with the related revoked public key.

The concept behind subkeys can be explained as

* Generate a keypair which contains __only__ a cerification key.
* Adding three subkeys. A signing key, an encryption key and an authentication key.
* Generate a revocation certificate from the primary key and the containing subkeys.
* This is your __master keypair__. Put it on a safe place.
* Copy this keypair on your computer where you want to use it (e.g. your laptop) and use GPG to remove the _certification key_. Only your subkeys remain in this this keyring and this keyring is called __Laptop Keypair__.

The laptop keypair is there to encrypt/decrypt messages, to sign messages and todo SSH logins.

The advantage of this configuration is that in case this keypair is stolen you have only to revoke the subkeys. You will not loose the signatures because the signatures are saved onto the master keypair.



## Generate master keypair

Use `gpg --expert --gen-key`, choose option (8) RSA with own capabilities. Toggle _Sign_ and _Encrypt_ that the current allowed action is _Ceritify_ only. Save the keyair with keylength 4096 and choose that the key does not expire. Adding also your name and email to this key.



## Generate Subkeys

Use `gpg --expert --edit-key <KEYID>` to open the menu to be able to edit the key. Use `addkey`, choose option (8) RSA with own capabilities. Toggle _Encryption_ and _Authentication_ to have a key which is there for _Signing_ only. Choose a keylength of _4096_ and let the key expire. I'm using one year and i'm adding every year new subkeys.

Do the same for two more subkeys. But toggle the capabilities to get a key for _Encryption_ and another key for _Authentication_.
At the end you have a keypair which contains your ceritifation key and three subkeys, one for signing, one for encryption and one for authentication.


## Create Revocation Certificate

If you will loose your _master keypair_ the _revocation certificate_ is the only way to communicate that your keypair is stolen and must be ignored. This step is urgent and should not be ignored or skipped.

Use `gpg --output <YOUR_EMAIL>.gpg-revocation-certificate --gen-revoke <YOUR_EMAIL>` and choose a reason for the revocation. Put that revocation certificate also on a safe place.


## Export the whole keypair

First export your private key with `gpg --export-secret-keys --armor <YOUR_EMAIL> > <YOUR_EMAIL>.private.gpg-key`. After that export your public key with `gpg --export --armor <YOUR_EMAIL> > <YOUR_EMAIL>.public.gpg-key`.

Put these two key files, the private and the public, together with the revocation certificate on a safe place. These three files are called your __master keypair__.



## Transform your Master Keypair into a Laptop Keypair

There is only one step you have todo to transform your master keypair into a laptop keyair. You have to remove the private certification key from the whole keypair. In gpg version > `2.0` it is much easier or more convenience. This documentation is made for gpg version `2.0.30`.
So what you have todo is to export the secret subkeys by executing the command `gpg --export-secret-subkeys <YOUR_EMAIL> > subkeys`. Then delete all secret keys from the keypair by executing `gpg --delete-secret-key <YOUR_EMAIL>` and import the subkeys back with `gpg --import subkeys`. At the end don't forget to delete the previous exported `subkeys` via `rm subkeys` or better `shred --remove subkeys`.
You will see that your private key is marked with `sec#` when using `gpg -K`. The sign `#` is there to mark a secret key as _not available_.



## Revoke Subkeys

In case your laptop keypair is stolen you use your master keypair to revoke your subkeys. Your are doing this by import your private and public key from the master keypair. `gpg --import /path/to/<YOUR_EMAIL>.public.gpg-key /path/to/<YOUR_EMAIL>.private.gpg-key`. `gpg --edit-key <YOUR_EMAIL>`. Select all 3 subkeys via `key 1`, `key 2` and `key 3`. Use `revkey` to revoke the selected subkeys and execute `save` to save the changed keyring. After that upload the revoked keys to a keyserver.


## Smartcard

I want to go one step further. I don't want to leave any foot print on my laptop. So the _laptop key_ is not that ideal goal for me. The idea is to move the generated private subkeys to a smartcard. And copy only the public keys to the laptop. A [yubikey](https://www.yubico.com) satisfy the configuration i want to have. The next section sould give you a rough overview what you can do to put your private keys onto the yubikey.

At the beginning configure your yubikey into super combo mode by using the command `ykpersonalize -m 86`. After that use `gpg --card-edit` to change the __admin pin__ from `12345678` to your own PIN. Do the same for the __user pin__ and change the default PIN `123456` to your own PIN. The user pin is the pin you have to use every day while using the smartcard. You can also personalize your yubikey by providing your name, the url to your public key.

Using `gpg --edit-key <YOUR_EMAIL>` and `toggle` to show your private subkeys. Select step by step all of your private subkeys e.g. select your first subkey with `key 1` and use `keytocard` to move your subkey. When you move your key you have also to select the "slot" where you want to move your key to. So a signature key has its own slot, as well as the encryption and authentcation key (see Sources). Do the same steps for the rest of your keys via `key 2` and `key 3`.

If you moved your key you can use `gpg -K` to list your private keys. A moved key is marked with `>` or more accurate `ssb>`.
E.g. the output of my private keys are
```
[Tue 28, 21:34] 0 % gpg -K
/Users/marko/.gnupg/secring.gpg
-------------------------------
sec#  4096R/53192101 2015-08-16
uid                  Marko Bauhardt <marko.bauhardt@mailbox.org>
uid                  Marko Bauhardt (Datameer GmbH) <mb@datameer.com>
ssb>  4096R/8E4E2613 2017-01-28
ssb>  4096R/55E95007 2017-01-28
ssb>  4096R/D8F097DF 2017-01-28
```

As you can see all my private keys are not stored on disk. The certification key does not exists in my keyring `sec#`. The private subkeys are also not stored in my keyring but they are moved to a smartcard `ssb>`.


## Where the smartcard can be use

So there are some areas where the GPG smartcard can be use. Here are several applications

+ Protect documents
+ Email conversation
+ Ssh logins
+ Pull/Push (which can be done via ssh) to [Github](https://github.com)
+ Sign git commits
+ Password manager like [pass](https://www.passwordstore.org)


#### Sources

+ [https://nightsi.de/das-perfekte-gpg-schlusselpaar-generieren](https://nightsi.de/das-perfekte-gpg-schlusselpaar-generieren)
+ [https://alexcabal.com/creating-the-perfect-gpg-keypair/](https://alexcabal.com/creating-the-perfect-gpg-keypair/)
+ [https://www.void.gr/kargig/blog/2013/12/02/creating-a-new-gpg-key-with-subkeys/](https://www.void.gr/kargig/blog/2013/12/02/creating-a-new-gpg-key-with-subkeys/)
+ [https://wiki.debian.org/Subkeys](https://wiki.debian.org/Subkeys)
+ [https://malcolmsparks.com/posts/yubikey-gpg.html](https://malcolmsparks.com/posts/yubikey-gpg.html)
+ [https://developers.yubico.com/PGP/Importing_keys.html](https://developers.yubico.com/PGP/Importing_keys.html)
