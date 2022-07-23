# Documentation Yubikey
By [Zidmann](mailto:emmanuel.zidel@gmail.com) :bow:

## Global presentation
![image info](./img/yubikey.png)

The **YubiKey** is a hardware authentication key manufactured by the Yubico company to protect accesses with a 2nd factor authentication.
It is a small USB security token which when it is plugged as a peripheral device provides an access to an electronically restricted resource in addition to a password.

The goal is to prevent an attacker from authenticating if the password was compromised since he/she does not have the required physical key.
Most of the distant hacks, like some phishing attacks, no longer works when a Yubikey is used.

In practical the Yubikey can be used with the devices to :
* access to an online service
* connect to a TTY terminal or log in a session for graphical operating system
* get a privilege escalation (case of sudo command)
* cipher/decipher an encrypted hardware (case of LUKS)
* protect an SSH key

## Studied device type
In this documentation, the type of Yubikey device studied is : **YubiKey 5 NFC**.

## Hardware design
The YubiKey can be used on a device as a USB stick or a RFID tag.
It does not require a battery, the power is provided by the USB port.

When the user wants to access a service or a device, all he/she needs to do, after typing the password, is to press the button on the YubiKey (if it is used as a USB stick) or to bring the YubiKey close to the phone (if it is used as a RFID tag).

When the YubiKey is used as a USB stick for an authentication, the device starts to flash its LED and to wait for you to tap.
After tapping the key, and then the user presence is confirmed, some characters are broadcast to the device.
Easy and efficient!

## Inputs/Outpus
### _Inputs_ ###
The YubiKey takes inputs in the form of API calls over USB and button presses.

The button is very sensitive. Depending on the context, touching it does one of these things :
* trigger a static password or one-time password (OTP) (short press for slot 1, long press for slot 2); this is the default behavior, and easy to trigger inadvertently.
* confirm/allow a function or access; the LED will illuminate to prompt the user.

### _Outputs_ ###
The YubiKey transforms these inputs into outputs :
* Keystrokes (emulating a USB keyboard), used to type static passwords and OTPs. (Note that static passwords are vulnerable to keyloggers.)
* The built-in LED :
  * Blinks once when plugged in, useful for troubleshooting.
  * Blinks steadily when a button press is required to permit an API response.
* API responses over USB. This is used for :
  * Challenge-Response requests (calculated using either Yubico OTP mode or HMAC-SHA1 mode)
  * U2F Challenge-Response requests

## Yubikey features, slots and interfaces

### _Features_ ###
The Yubikey can :
* handle Universal 2nd factor (U2F) requests
* handle challenge-response requests with OTP or HMAC-SHA1 mode
* generate one-time password (OTP)
* type a static password

### _Slots_ ###
Yubikey has two slots to store configuration for :
* Yubico OTP
* OATH-HOTP
* static password
* challenge response (with Yubico OTP or HMAC-SHA1)

### _Interfaces_ ###
Yubikey can interact with USB or NFC interfaces. These interfaces can be configured for the protocol below :
* OTP
* FIDO2
* FIDO U2F
* OpenPGP
* PIV
* OATH 

## Configure Yubikey hardware
Yubico provides tools to manage the Yubikey hardware.

The procedure is described in **[MANAGER.md](./MANAGER.md)** page.

## Authentication with Linux PAM
Linux Pluggable Authentication Modules (short for PAM) are a suite of libraries that allows a Linux system administrator to configure methods to authenticate users (or services) in a Linux system.
It is used to authenticate with the Yubikey: Yubico company provided a PAM module to install to have this feature.

Linux PAM can be used to connect to a TTY terminal, log in a session or get a privilege escalation.

To configure a device to control access with a Yubikey, go to **[PAM.md](./PAM.md)** page.

## Use Yubikey to protect an SSH key
To install the Yubikey authentication for SSH connections on the client side, the procedure is described in **[SSH-KEY.md](./SSH-KEY.md)** page.

## Use Yubikey with LUKS
The procedure to secure hard drive partitions with a Yubikey authentication is described in **[TWO-FACTOR.md](https://github.com/Zidmann/Documentation-LUKS/blob/main/TWO-FACTOR.md)** page inside **[Documentation-LUKS](https://github.com/Zidmann/Documentation-LUKS)** project.

## Device unreachable
**!! WARNING !!** The Yubikey device can be unusable for different situations. Two main cases were identified during the tests :
* the permission does not allow the user to access the hardware
* the Yubikey listened and waited for the user action previously, however the request was cancelled (like with Ctrl + C) and the Yubikey still waits for the user action

### _Hardware unreachable_ ###
Some operating systems can prevent some Linux users from using a Yubikey and return this message :
```bash
Key enrollment failed: device not found
```

During the Yubikey tests, for creating/using SSH keys, **'root'** can access freely to the Yubikey with **ssh-keygen** and sometimes the first user created (during the installation).
According to the operating system, the **udev** rules can affect the permissions of the **/dev/hidrawXX** associated with Yubikey device.

For Ubuntu 20.04.4 LTS, it seems that the Yubikey device has **plugdev** defined as group with read/write permissions when it is connected.
```bash
> ls -rtl /dev/hidraw*
[...]
crw-rw----+ 1 root plugdev 237, 4 juin  20 11:58 /dev/hidraw<num>
```

Then adding the user to the group **plugdev** solves the problem for Ubuntu 20.04.4 LTS. You can do this command :
```bash
# usermod -aG plugdev <user>
```

According to the situation and the case, it will be possible to change :
* the udev configuration
* the users defined in some groups
* AppArmor or SELinux

**Be careful with the operating system you use. If you want to use a new one, testing the Yubikey device is required to see how will be its behaviour.**

### _Yubikey request was previously killed_ ###
When a process (like **ssh-keygen**) waits for a 2nd factor authentication, it will send a message to the Yubikey hardware which starts flashing its LED and waiting for somebody to tap.
However, if this process is stopped, the Yubikey keeps to wait for the user action and the next process which will request a 2nd factor authentication will fail and generate this message :
```bash
Key enrollment failed: invalid format
```

To solve this problem, there are two solutions :
* remove and replug the Yubikey
* press the flashing button, which will generate a string like a keyboard

## Projects
The **Yubico** company provides several sources codes on [GitHub](https://github.com/Yubico), included :
* [yubico-pam](https://github.com/Yubico/yubico-pam) - the Yubico PAM module for the user authentication
* [pam-u2f](https://github.com/Yubico/pam-u2f) - the PAM over U2F and FIDO2 implemented module
* [yubikey-manager](https://github.com/Yubico/yubikey-manager) - the command line tool for configuring a YubiKey over a USB interface
* [yubikey-manager-qt](https://github.com/Yubico/yubikey-manager) - the graphical application for configuring a YubiKey over a USB interface

## References
* https://en.wikipedia.org/wiki/Linux_Unified_Key_Setup
* https://en.wikipedia.org/wiki/Linux_PAM
* https://www.tecmint.com/configure-pam-in-centos-ubuntu-linux/
* https://wiki.archlinux.org/title/YubiKey
