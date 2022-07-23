# Using a YubiKey with an SSH key
By [Zidmann](mailto:emmanuel.zidel@gmail.com) :bow:

## Global presentation
SSH is the default method for system administrators to log into remote Linus systems.
The protocol can use either a password or an SSH key to authenticate.
An SSH key is composed of a public key and a private key, moreover the private key is traditionally secured with a password to cipher its content.

However, this situation can be improved nowadays by enforcing a second authentication factor with a YubiKey.
Then, if a private SSH key is stolen (with its password compromised), it will not be usable.

There are two types of key where a Yubikey can be used as a 2nd factor authentication :
* ecdsa-sk
* ed25519-sk

## Requirements
SSH version must be _**8.3 or greater**_ on the client and the server.
Yubikey firmware must be _**5.2.3 or greater**_ to use _**ed25519-sk**_ type key, but older version can use _**ecdsa-sk**_ type key.

To check the Yubikey firmware version, use the following command :
```bash
> lsusb -v 2>/dev/null | grep -A2 Yubico
```

## Security and privacy
**!! WARNING !!** In 2007, two Microsoft employees highlighted a weakness in NIST SP800-90 Dual Ec Prng which could be backdoor implemented by the NSA.
In other words, the ECDSA algorithm implementation could be concerned by a backdoor making the state organization capable of reading the communications using this algorithm.

Then, if it is possible the ED25519 should be used instead whenever possible.
In the next steps, the algorithm used will be ED25519.
Feel free to use ECDSA instead if you want : in this case, replace **ed25519** by **ecdsa**.

## Create a new SSH key
The **ssh-keygen** will simply be used with the option **-t** (type of key). You can use other option if necessary :
```bash
> ssh-keygen -t ed25519-sk
Generating public/private ed25519-sk key pair.
You may need to touch your authenticator to authorize key generation.
```

By default, ssh-keygen creates these files :
* _**id_ed25519_sk**_ as a private key
* _**id_ed25519_sk.pub**_ as a public key

If the console prints the message below, then it means that the Yubikey is not plugged on the device.
```bash
Key enrollment failed: device not found
```

## Use the SSH key
To use the SSH key you can use the same commands to a classical SSH key.
Contrary to the sources, it is not necessary on the most recent version to change the **/etc/ssh/sshd_config** configuration file.

### _Connect with a specific key_
```bash
> ssh -i ~/.ssh/id_ed25519_sk <user>@<host>
```

### _To create an SSH key using Yubikey as 2nd factor authentication_
```bash
> ssh-keygen -t ed25519-sk
```

### _To add the SSH key in the SSH agent_
```bash
> ssh-add ~/.ssh/id_ed25519_sk
```

### _To connect with a specific key_
```bash
> ssh-copy-id -i ~/.ssh/ed25519-sk <user>@<host>
```

If the key is plugged :
```bash
Confirm user presence for key ED25519-SK <key signature>
User presence confirmed
```

Otherwise, you will get :
```bash
Confirm user presence for key ED25519-SK <key signature>
sign_and_send_pubkey: signing failed for ED25519-SK "/home/ubuntu/.ssh/id_ed25519_sk": device not found
```

## Tests
The Yubikey used with SSH was tested and validated on
* Github service
* an OpenSSH server

## References
* https://developers.yubico.com/SSH/Securing_SSH_with_FIDO2.html
* https://bash-prompt.net/guides/bash-ssh-yubikey/
* https://www.youtube.com/watch?v=QGZz_xb0fCU
* https://www.wired.com/2013/09/nsa-backdoor/
* https://www.wired.com/images_blogs/threatlevel/2013/09/15-shumow.pdf
