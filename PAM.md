# Using Linux PAM to control access with a YubiKey
By [Zidmann](mailto:emmanuel.zidel@gmail.com) :bow:

## Authentication with Linux PAM

### _Global presentation_ ###
Linux Pluggable Authentication Modules (short for PAM) are a suite of libraries that allows a Linux system administrator to configure methods to authenticate users (or services) **with password** on a Linux system.
It can be used with the Yubikey to authenticate through a PAM module provided by the Yubico company after a password is required.

It offers a flexible way to update the authentications instead of changing application code.
It centralizes the behaviour of applications in the configuration files stored in the directory **/etc/pam.d**.
There is also the file **/etc/pam.conf** but it is disabled when the **/etc/pam.d** directory has files.

**!! WARNING !!** Be careful when you change the PAM configuration, otherwise your machine or its administrator access could become unreachable.
If this happens, you will have to remove your change directly in the configuration files after booting with another way (like with a USB or a Live CD).

### _Configuration_ ###

A configuration file in **/etc/pam.d** uses this format :
```bash
type control-flag module module-arguments
```

The meaning of each attribute is described below :

| Attribute | Description |
|--------|--------|
| type | The module type/context/interface. |
| control-flag | It indicates the behavior of the PAM-API should the module fail to succeed in its authentication task. |
| module | The absolute file name or relative path name of the PAM.|
| module-arguments | Space separated list of tokens for controlling module behavior. |

Linux PAM tasks are separated into four independent management groups described below :

| Group name | Description |
|--------|--------|
| account | It checks that the specified account is a valid authentication target under current conditions.<br/>This may include conditions like account expiration, time of day, and that the user has access to the requested service. |
| authentication | It authenticates a user and set up user credentials. |
| password | It is responsible for updating user passwords and works with authentication modules. |
| session | It manages actions performed at the beginning of a session and end of a session |

### _Most used authentication_ ###

| Configuration path | Description |
|--------|--------|
| /etc/pam.d/common-auth | It is a common configuration used for all authentication (use with caution !!) |
| /etc/pam.d/sudo | It refers when a user uses **sudo** command to run programs with security privileges |
| /etc/pam.d/login | It refers when a user connects to one **TTY console** with a physical access |
| /etc/pam.d/sshd | It refers when a user connects remotely with an SSH connection |
| /etc/pam.d/gdm-password | It refers when a user **logs into** a session through a graphical session |

**!! WARNING !!** /etc/pam.d/su should not be changed even if the **su** command is used by users to switch from a user to another one.
Indeed, some scripts could also use **su** command and be bothered by the PAM rules if it is required to press the Yubikey button.

**!! Note !!** In the case of SSH, it was observed that the message which requests to tap the Yubikey appears not after typing the password ... but after pressing the Yubikey button. Keep in mind that the Yubikey must be tapped on the server side, not on the client side.

## Install Linux PAM module
To prepare a device to use 2nd factor authentication, it is necessary to install the PAM module dedicated.
To do this, use the command :
```bash
# apt-get install libpam-u2f
```

Without this installation, you will get the error :
```bash
 PAM authentication error: Module is unknown
 a password is required
```

In practice, the installation will add the **pam_u2f.so** file in the directory **/usr/lib/<architecture>/security/**.
The architecture directory can be **x86_64-linux-gnu** for a classical computer or **aarch64-linux-gnu** for a Raspberry for example.

## Supported parameters

### _Most used_ ###
| Option | Description |
|--------|--------|
| nouserok | Set to enable authentication attempts to succeed even if the user trying to authenticate is not found inside __**authfile**__ or if __**authfile**__ is missing/malformed. |
| authfile=file | Set the location of the file that holds the mappings of user names to keyHandles and user keys. An individual (per user) file may be configured relative to the users' home dirs, e.g. ".ssh/u2f_keys". If not specified, the location defaults to $XDG_CONFIG_HOME/Yubico/u2f_keys. If $XDG_CONFIG_HOME is not set, $HOME/.config/Yubico/u2f_keys is used. |
| cue | Set to prompt a message to remind to touch the device. |
| [cue_prompt=your prompt here] | Set individual prompt message for the cue option. Watch the square brackets around this parameter to get spaces correctly recognized by PAM. |
| nodetect | Set to skip detecting if a suitable FIDO token is inserted before performing the full tactile authentication. This detection was created to avoid emitting the "cue" message if no suitable token exists, because doing so leaks information about the authentication stack if a token is inserted but not configured for the authenticating user. However, it was found that versions of __**libu2f-host 1.1.5**__ or less has buggy iteration/sleep behavior which causes a 1-second delay to occur for this initial detection. For this reason, as well as the possibility of hypothetical tokens that do not tolerate this double authentication, the "nodetect" option was added. |
| debug  | Enables debug output |
| debug_file | Filename to write debugging messages to. **If this file is missing, nothing will be logged.** This regular file **has to be created by the user** or **must exist and be a regular file** for anything getting logged to it. It is not created by pam-u2f on purpose (for security considerations). This filename may be alternatively set to "stderr" (default), "stdout", or "syslog". |

### _Others_ ###
| Option | Description |
|--------|--------|
| origin=origin | Set the relying party ID for the FIDO authentication procedure. If no value is specified, the identifier "pam://$HOSTNAME" is used. |
| appid=appid | Set the application ID for the FIDO authentication procedure. If no value is specified, the same value used for origin is taken ("pam://$HOSTNAME" if also origin is not specified). This setting is only applicable for U2F credentials created with pamu2fcfg versions v1.0.8 or earlier. Note that on v1.1.0 and v1.1.1 of pam-u2f, handling of this setting was temporarily broken if the value was not the same as the value of origin. |
| authpending_file=file | Set the location of the file that is used for touch request notifications. This file will be opened when pam-u2f starts waiting for a user to touch the device, and will be closed when it no longer waits for a touch. Use inotify to listen on these events, or a more high-level tool like yubikey-touch-detector. Set an empty value in order to disable this functionality, like so: authpending_file=. Default value: /var/run/user/$UID/pam-u2f-authpending. |
| openasuser | Setuid to the authenticating user when opening the authfile. Useful when the user’s home is stored on an NFS volume mounted with the __**root_squash**__ option (which maps root to nobody which will not be able to read the file). Note that after release 1.0.8 this is done by default when no global authfile or XDG_CONFIG_HOME environment variable has been set. |
| alwaysok | Set to enable all authentication attempts to succeed (aka presentation mode). |
| max_devices=n_devices | Maximum number of devices allowed per user (default is 24). Devices specified in the authentication file that exceed this value will be ignored. |
| interactive | Set to prompt a message and wait before testing the presence of a FIDO device. Recommended if your device doesn’t have a tactile trigger. |
| [prompt=your prompt here] | Set individual prompt message for interactive mode. Watch the square brackets around this parameter to get spaces correctly recognized by PAM. |
| manual | Set to drop to a manual console where challenges are printed on screen and response read from standard input. Useful for debugging and SSH sessions without U2F-support from the SSH client/server. If enabled, interactive mode becomes redundant and has no effect. |
| userpresence=int | If 1, request user presence during authentication. If 0, do not request user presence during authentication. If omitted, fallback to the authenticator’s default behaviour. |
| userverification=int | If 1, request user verification during authentication (e.g. biometrics). If 0, do not request user verification during authentication. If omitted, fallback to the authenticator’s default behaviour. If enabled, an authenticator with support for FIDO2 user verification is required. |
| pinverification=int | If 1, request PIN verification during authentication. If 0, do not request PIN verification during authentication. If omitted, fallback to the authenticator’s default behaviour. If enabled, an authenticator with support for a FIDO2 PIN is required. |
| sshformat | Use credentials produced by versions of OpenSSH that have support for FIDO devices. It is not possible to mix native credentials and SSH credentials. Once this option is enabled all credentials will be parsed as SSH. |

### _Example_ ###
Below some PAM configurations, which can be used :
```bash
auth   required   pam_u2f.so   nouserok authfile=/etc/Yubico/u2f_keys cue debug debug_file=/var/log/pam_u2f.log
auth   required   pam_u2f.so   authfile=/etc/u2f_keys
auth   required   pam_u2f.so   nouserok authfile=/etc/u2f_keys cue
```

These lines are usually written quickly after the **@include common-auth** in the configuration file.

## Reloading the PAM configuration
To take into account the changes in the configuration files, you must use the command below :
```bash
# /usr/sbin/pam-auth-update
```

## The mapping file

### _Global presentation_ ###
Using Yubikey devices requires to use a file with authorized keys (like SSH).

### _Path and content_ ###
By default, each user has a key file stored in the path **$HOME/.config/Yubico/u2f_keys**. However, it is possible to centralize all these configuration files in one single (with **authfile** option).

The content should look like this, one per line:
```bash
<User1>:<KeyHandle1>,<UserKey1>,<CoseType1>,<Options1>:<KeyHandle2>,<UserKey2>,<CoseType2>,<Options2>:...
<User2>:<KeyHandle1>,<UserKey1>,<CoseType1>,<Options1>:<KeyHandle2>,<UserKey2>,<CoseType2>,<Options2>:...
```

## Add a Yubikey in the default mapping file
The next steps concerned the case when you want each user to authenticate with the Yubikey.

### _Create the Yubico default directory_ ###
To create the directory where to store the Yubikey keys (if each user manages its file) :
```bash
> mkdir -p ~/.config/Yubico
```

### _Create a Yubikey key files_ ###
```bash
> pamu2fcfg > ~/.config/Yubico/u2f_keys
```

### _Add another Yubikey key files_ ###
```bash
> pamu2fcfg -n >> ~/.config/Yubico/u2f_keys
```

## References
* https://support.yubico.com/hc/en-us/articles/360016649099-Ubuntu-Linux-Login-Guide-U2F
* https://developers.yubico.com/pam-u2f/
* https://askubuntu.com/questions/1071027/how-to-configure-a-u2f-keysuch-as-a-yubikey-for-system-wide-2-factor-authentic
