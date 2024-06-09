---
alias: SSH Keycutter
---
# [Preview] Keycutter - FIDO SSH Keys made easy

-  8 Jun 2024: Preparing for alpha release (big changes being made)
- 18 Mar 2024: Preview release for review only

Ever wondered how to contribute to an open-source project on GitHub from an employer managed (i.e. untrusted) laptop, without compromising the security of your personal GitHub account?

Keycutter came out of an attempt to solve this problem but evolved into a tool to improve security by simplifying FIDO SSH Key management.

Keycutter is a config cookie cutter that creates FIDO SSH Keys on Hardware Security Keys, along with Git and SSH config so they're ready to use immediately.

*While initially created for use with YubiKeys and GitHub, Keycutter supports other devices and services.*

## What Keycutter gives you

**Security:**

- **Unstealable* FIDO SSH keys:** No way to extract them from the device.
- **Physical presence verification:** touch to approve each use.
- **PIN retry lockout:** defend against stolen hardware security token.

**Convenience:**

- **Git commit and tag signing with your SSH keys**
- **Automatic SSH key selection by given service/identity based on host aliases**
- **SSH and Git config in one place**
- **AWS SSH-over-SSM**

**Privacy:**

- **Security Boundary Separation:** Different keys for different personal, work, etc.
- **Selective key forwarding with ssh-agent:** Map keys to services and devices.

**All config is stored in a single directory (`~/.keycutter`) which:**

- Can be kept in version control.
- Can be used on multiple devices / hosts.

## SSH Keytags

Managing multiple FIDO SSH keys across multiple devices and services can be an effort.

Keycutter introduces **SSH Keytags**, labels to help you organise and keep track of your
FIDO SSH Keys across multiple devices and services.

**SSH Keytags are used:**

- In the SSH Key filename
- In the public key comment
- In the key name on services like GitHub

**SSH Keytag format:**  `<service>_<identity>@<device>`

- **Service:** FQDN of remote service (e.g. gitlab.com)
- **Identity:** The **user account** on remote service (e.g. alexdoe-work)
- **Device**: The **hardware security key** or **computer** where the private key resides.

*Read more about [SSH Keytags](docs/design/ssh-keytags.md)*

## Getting Started

- **Prerequisites:**
  
  - **Bash >= 4.0**
  - **Git >= 2.34.0**
  - **GitHub CLI >= 2.0** (Greater than 2.4.0+dfsg1-2 on Ubuntu)
  - **OpenSSH >= 8.2p1** (WSL users need `ssh-sk-helper`([OpenSSH for Windows >= 8.9p1-1](https://github.com/PowerShell/Win32-OpenSSH/releases)))
  - **YubiKey Manager (`ykman`)**

- **Recommended:**

	- [YubiKey Touch Detector](docs/yubikeys/yubikey-touch-detector.md)

### 1. Install Keycutter

**Clone the Git repo:**

```shell
git clone https://github.com/bash-my-aws/keycutter
```

**Add the following to your shell profile (e.g. bashrc or zshrc):**

```shell
# keycutter/ssh/config uses these to determine which SSH key to use
export KEYCUTTER_HOSTNAME="$(hostname -s)" # or a preferred alias for this device
[[ -z "${SSH_CONNECTION}" ]] && export KEYCUTTER_ORIGIN="${KEYCUTTER_HOSTNAME}"

# WSL (Windows Subsystem for Linux) users need to set the path to ssh-sk-helper.exe
if [[ -f "/mnt/c/Program Files/OpenSSH/ssh-sk-helper.exe" ]]; then
	export SSH_SK_HELPER="/mnt/c/Program Files/OpenSSH/ssh-sk-helper.exe"
fi
```

**Optionally add the bin directory to your path:**

*As a config cookie cutter, keycutter is not required to be in the path but it is useful for generating new configs.*

```shell
# Used for generating config with keycutter
export PATH="$PATH:${PWD}/keycutter/bin"
```

### 2. Create a FIDO SSH Key

**Provide an SSH Keytag (`<service>_<identity>@<device>`) to the create command:**

```shell
keycutter create github.com_alexdoe@personal
```

For a particular service and identity on a device:

- Separate the domain name and user name with an underscore.
- Append the device name after the `@` symbol.

### 3. Review your keycutter config

The directory contains all the Git / SSH config files for your FIDO SSH keys.

```shell
tree ~/.keycutter
```

**Example output:**

    ```shell
    $ tree ~/.keycutter
    /home/alexdoe/.keycutter
    ├── git
    │   ├── allowed_signers
    │   ├── config.d
    │   │   └── github.com_alexdoe-work
    │   └── config
    └── ssh
        ├──config 
        └── keys
            ├── github.com_alexdoe-work@yk01
            └── github.com_alexdoe-work@yk01.pub
    ```

## Usage

### 1. Clone a GitHub repo using your new key

```shell
git clone git@github.com_alexdoe:bash-my-aws/keycutter
```

### 2. Commit a change signed with your new SSH Key

```shell
cd keycutter
date >> README.md 
git commit -m "I signed this commit with my new SSH Key"
```

### 3. Explore your new config

You're ready for FIDO SSH access to anything you were using file based SSH keys for.


## See also

- [Keycutter Documentation](docs/README.md)
- [Tips and tricks](docs/tips-and-tricks.md): Undocumented features and cool tricks.
