---
title: "Upgrade Nas4Free from 9.3.0.2 to 10.2.0.2"
date: 2016-03-11
tags: ["nas4free"]
---

I upgraded my nas machine which is based on [nas4free](http://www.nas4free.org) this wednesday.
Everything went smooth. I made a backup of my current configuration. I uploaded the new `10.2.0.2.2332` image and after some minutes my server reboots automatically.
After this I was able to login. The [upgrade procedure](https://sourceforge.net/projects/nas4free/files/NAS4Free-10.2.0.2/10.2.0.2.2332/) described to restore my previous configuration.
But this failed. The browser showed me the error `XML error: Not well-formed (invalid token) at line 1`. I assume the encrypted version of my configuration can not be decrypted.
After reading some blog posts i found [this one](https://johncireland.wordpress.com/2016/02/13/decrypt-or-encrypt-nas4free-config/).
So i followed the steps described there to decrypt my configuration. With success. I decrypted my configuration with 

    openssl enc -aes-256-cbc -d -a -in <my-encrypted-config-file> -out decrypted-version.xml
    
I created an issue on the [sourceforge bugtracker]((https://sourceforge.net/p/nas4free/bugs/276/)).

Restoring the decrypted configuration was no issue for nas4free. I was able to upgrade my nas4free machine to version 10.2.0.2 without any big issues.

Nice!


