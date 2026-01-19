bin-admin
=========

`bin-admin` is a collection of Bash scripts that would be found useful by Linux system administrators. This repo is often deployed in conjuction with [my dotfiles](https://github.com/DavidVogelxyz/dotfiles).

Usage
-----

Within the root of this project (`bin-admin`) is a directory named `bin-admin`. Create a symlink at `${HOME}/.local/bin/bin-admin` that points to the *subdirectory* named `bin-admin`.

Table of contents
-----------------

- [Usage](#usage)
- [Scripts](#scripts)
    - [ssh-ca_cert-renew](#ssh-ca_cert-renew)
        - [Instructions](#ssh-ca_cert-renew-instructions)
            - [configs directory](#ssh-ca_cert-renew-configs-directory)
            - [keys directory](#ssh-ca_cert-renew-keys-directory)
        - [Directory tree](#ssh-ca_cert-renew-directory-tree)

Scripts
-------

### `ssh-ca_cert-renew`

Date created: 2025 Aug 5, Tue

`ssh-ca_cert-renew` was written in order to simplify the management of a SSH certificate authority (CA). For more information on SSH CAs, refer to the [documentation found here](https://github.com/DavidVogelxyz/library/blob/master/docs/security/ssh/ssh-ca/README.md).

`ssh-ca_cert-renew` assumes that the admin has a directory (`DIR_SSH_CA`) which contains the keys and "site configuration files" for a SSH CA. This README suggests that the admin make this directory into a Git repo, which will allow the admin to perform version control over the keys and certificates within.

#### `ssh-ca_cert-renew` instructions

In order to run `ssh-ca_cert-renew`, a configuration file must be created that sets the path to `DIR_SSH_CA`. For more information, refer to the comment block in the script that addresses the `source_conf` function.

Once the configuration file exists, and the script has a definition for `DIR_SSH_CA`, the script will attempt to find various files within `DIR_SSH_CA`. For a diagram of what the directory structure should look like, refer to [this section of the README](#ssh-ca_cert-renew-directory-tree).

The two main directories that should exist are the `configs` and the `keys` directory. The scripts assumes that the `configs` and `keys` directories will be organized into subdirectories for different sites; though, this can also be changed in the admin's configuration file that gets sourced by the `source_conf` function.

While the script will look for any "signing CA keys" at their respective paths, the script will attempt to find these keys in the admin's `ssh-agent`. It is suggested that the admin leverage this feature in order to simplify certificate generation and renewal.

If the files in `configs` have been configured correctly, and a configuration file exists to set `DIR_SSH_CA`, then the script should run in a self-explanatory way. The admin will be asked for information to assess:
- if the type of certificate to renew is a "host" or "user" certificate
- for which site the script will be renewing certificates
- the quantity of certificates to be renewed (either "single certificate" or "all certificates")
- if the quantity is "single", which specific certificate to renew

Once this information has been provided, the script will check the `ssh-agent` for the corresponding signing key, and will generate the certificate.

##### `ssh-ca_cert-renew` - `configs` directory

The `configs` directory should have, at minimum, 3 files: a `hosts` file, a `users` file, and at least 1 `userlist` file. The `hosts` file is a newline separated list of all the hosts in the SSH CA *for that site*, and the `users` file is a newline separated list of all the users in the SSH CA *for that site*. In contrast, the `userlist` file is a file with a single line, and contains a comma-separated list of the principals (user accounts on hosts) for which a given user will have access *for hosts at that site*.

It is suggested that the `hosts` file have, as its first entry, the line `SITE host CA`. Doing so will allow the admin to also renew the site's host certificate, which is important to give to users so that they can verify hosts within the SSH CA.

##### `ssh-ca_cert-renew` - `keys` directory

Each site subdirectory in the `keys` directory is expected to have 4 directories each: a `host-CA` directory, a `hosts` directory, a `user-CA` directory, and a `users` directory. The `host-CA` directory will contain the public and private keypair for the host CA for that site. The `hosts` directory will contain the public and private keypairs for all hosts at the site. Similarly, the `user-CA` directory will contain the public and private keypair for the user CA for that site, and the `users` directory will contain the public keys for all users accessing hosts at that site.

For any directories within `keys/SITE`, if it doesn't already exist, a `certs` subdirectory will be created to store any generated certificate files.

For ease of use, it is also suggested to create a `users` subdirectory within `keys` -- this way, a user that accesses multiple sites can have their public key symlinked into whichever `keys/SITE/users` directory it should exist within.

#### `ssh-ca_cert-renew` directory tree

```
DIR_SSH_CA
├── configs
│   ├── SITE_1
│   │   ├── hosts
│   │   ├── userlist_user_1
│   │   └── users
│   └── SITE_2
│       ├── hosts
│       ├── userlist_user_2
│       └── users
└── keys
    ├── SITE_1
    │   ├── host-CA
    │   │   ├── certs
    │   │   ├── SITE_1_host-CA
    │   │   └── SITE_1_host-CA.pub
    │   ├── hosts
    │   │   ├── certs
    │   │   ├── HOST_1
    │   │   ├── HOST_1.pub
    │   │   ├── HOST_2
    │   │   └── HOST_2.pub
    │   ├── user-CA
    │   │   ├── SITE_1_user-CA
    │   │   └── SITE_1_user-CA.pub
    │   └── users
    │       ├── certs
    │       └── user_1.pub -> ../../users/user_1.pub
    ├── SITE_2
    │   ├── host-CA
    │   │   ├── certs
    │   │   ├── SITE_2_host-CA
    │   │   └── SITE_2_host-CA.pub
    │   ├── hosts
    │   │   ├── certs
    │   │   ├── HOST_A
    │   │   ├── HOST_A.pub
    │   │   ├── HOST_B
    │   │   └── HOST_B.pub
    │   ├── user-CA
    │   │   ├── SITE_2_user-CA
    │   │   └── SITE_2_user-CA.pub
    │   └── users
    │       ├── certs
    │       └── user_2.pub -> ../../users/user_2.pub
    └── users
        ├── user_1.pub
        └── user_2.pub
```
