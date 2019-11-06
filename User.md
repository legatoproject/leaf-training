- [Introduction](#introduction)
  - [Installation](#installation)
    - [Debian package](#debian-package)
    - [From the sources](#from-the-sources)
    - [Using PyPI](#using-pypi)
    - [Updating Leaf](#updating-leaf)
  - [Common CLI arguments](#common-cli-arguments)
- [Remotes](#remotes)
  - [Adding/removing remotes](#addingremoving-remotes)
  - [Enabling/disabling remotes](#enablingdisabling-remotes)
  - [Security](#security)
  - [Priorities](#priorities)
- [Packages](#packages)
  - [Manifest](#manifest)
  - [Data](#data)
  - [Environment variables](#environment-variables)
  - [Binaries](#binaries)
  - [Settings](#settings)
  - [Help](#help)
  - [Plugins](#plugins)
- [Workspace](#workspace)
  - [Structure](#structure)
  - [Environment](#environment)
  - [Sharing a workspace using Git](#sharing-a-workspace-using-git)
- [Profiles](#profiles)
  - [Basic operations](#basic-operations)
  - [Synchronizing a profile](#synchronizing-a-profile)
  - [Environment](#environment-1)
  - [Leaf shell](#leaf-shell)
  - [Package override](#package-override)
- [Extra](#extra)
  - [Autocompletion](#autocompletion)
  - [Settings](#settings-1)
  - [Configuration override](#configuration-override)
  - [All-In-One commands](#all-in-one-commands)
    - [*leaf setup*](#leaf-setup)
    - [*leaf update*](#leaf-update)
  - [Keyword *latest*](#keyword-latest)




# Introduction

- Generic
- Userland
- Multiple versions of packages
- Dependencies with conditions


## Installation

### Debian package

Using the SierraWireless *debian* package
```sh
$ wget https://downloads.sierrawireless.com/tools/leaf/leaf_latest.deb -O /tmp/leaf.deb
$ sudo apt install /tmp/leaf.deb
```

### From the sources

```sh
$ sudo apt install python3-all python3-pip python3-setuptools 
$ git clone ssh://$USER@gerrit.legato:29418/leaf
$ cd leaf
$ python3 setup.py install --user
$ python3 setup.py install_data -d $HOME/.local/

# In cas of ~/.local/bin is not in the PATH
$ export PATH=$PATH:~/.local/bin

$ leaf --version
```

### Using PyPI

In the future, we could imagine publishing *leaf* on PyPI so that you could install *Leaf* with:

```sh
$ sudo apt install python3-pip

# Install for user only
$ pip3 install swi-leaf --user

# Install for all users
$ sudo pip3 install swi-leaf
```

> Leaf is not published on PyPI for now, the `swi-leaf` package does not exist yet!

### Updating Leaf

In installed from the deb package, Swi apt repository is automatically added in `/etc/apt/sources.list.d/`

```sh
$ sudo apt update
$ sudo apt full-upgrade
# Or update only leaf ... not recommended
$ sudo apt install --only-upgrade leaf
```

## Common CLI arguments

Every leaf command has `--verbose` and `--quiet` to set the verbosity.

There is also a `--non-interactive` to disable interactive behavior for installation for example.


Every command is also documented with `--help` or `-h`

```
$ leaf -h
usage: leaf [-h] [-v | -q] [-V] [--non-interactive] [-w LEAF_WORKSPACE]
            SUBCOMMAND ...

Copyright Sierra Wireless. All rights reserved.

  Licensed under the Mozilla Public License Version 2.0
  https://www.mozilla.org/en-US/MPL/2.0/

  Leaf is part of the Legato Project, https://legato.io/

optional arguments:
  -h, --help            show this help message and exit
  -v, --verbose         increase output verbosity
  -q, --quiet           decrease output verbosity
  -V, --version         show program's version number and exit
  --non-interactive     assume yes if a print_with_confirmation is asked
  -w LEAF_WORKSPACE, --workspace LEAF_WORKSPACE
                        use given workspace

subcommands:
  supported subcommands

  SUBCOMMAND            actions to execute
    status              print leaf status
    search              search for available packages
    select              change current profile and install missing packages
    env                 display environement variables
    run                 execute binary provided by installed packages
    config              manage leaf configuration
    init                initialize workspace
    profile             command to manage profiles
    remote              display and manage remote repositories
    package             core package manager commands
    build               commands to build leaf artifacts (manifest, package or
                        index)
    help                read leaf documentation
    setup               all in one command to create a profile in a workspace
    update              update current profile packages
    shell               run a sub-shell that tracks the environment for your
                        Leaf workspace
    completion          helper to configure auto completion in your shell
                        (bash or zsh)
    version             display leaf version
```


# Remotes

Remotes are *URI* to json files referencing leaf packages.
Many protocoles are supported:
- *HTTP(s)*: `https://server.tld/index.json` or `http://server.tld/index.json`
- Local filesystem: `file:///tmp/index.json` or `/tmp/index.json`
- Any protocole supported by `python3-request` library


## Adding/removing remotes

```sh
$ leaf remote add --insecure my-remote https://server.tld/index.json
$ leaf remote remote my-remote
```

> In the next version (v2.2), `--insecure` won't be mandatory anymore

## Enabling/disabling remotes 

```sh
$ leaf remote enable my-remote
$ leaf remote disable my-remote
```


## Security

Remotes can be secured using *GnuPG*, if so, leaf verify the content of `index.json` with the signature `index.json.asc`
```sh
$ leaf remote add --gpg 8C20018BE986D5300A346323FAE026860F1F8AEE my-remote https://server.tld/index.json
```
> Note: By default the signature is imported from `subset.pool.sks-keyservers.net`, the GPG server can be changed with the setting `leaf.gpg.server`


## Priorities

When an artifact is available in more than 2 remotes, priorities will allow to use one specifically
- Local filesystem repository will have an higher priority
- When adding a new remote, you will be able to set a custom priority

> Note: This feature will be available with leaf version 2.2



# Packages

A *leaf* is a *TAR* archive which contains at least a file: `manifest.json`


## Manifest

This file provides the metadatas of the package like
- the package name, matching `[a-zA-Z0-9][-a-zA-Z0-9]*`
- the package version, matching `[a-zA-Z0-9][-._a-zA-Z0-9]*`
- a short description
- the documentation URL
- the dependencies
- the commands to be executed during install/uninstall
- ...

> The *Package Identifier* is the concatenation of the name and the version using `_` as separator, example: `myPackage_1.0-beta1`


## Data

All files contained in the *tar* leaf package are extracted in the package folder which is located by default, in `~/.leaf/<PACKAGE_IDENTIFIER>/`

> Note: the *root* folder where packages are installed be changed using the setting: `leaf.user.root`


## Environment variables

A package can export some *environment variables*. Use `leaf env package <PACKAGE_IDENTIFIER>` to print all variables exported by a given package.

```sh
$ leaf env package wp76-legato_18.07.1-201809191119
# Exported by package wp76-legato_18.07.1-201809191119
export LEGATO_ROOT="/home/seb/.leaf/wp76-legato_18.07.1-201809191119";
export LEGATO_TARGET="${LEGATO_TARGET:-wp76xx}";
export PATH="$LEGATO_ROOT/bin:$PATH";
export DEST_IP="${DEST_IP:-192.168.2.2}";
```

> Note: Do not forget the quotes when you use `eval`


## Binaries

Leaf packages can bundle *tools* which you can run using `leaf run`

> Note: Feature is not used for the moment, but could be used for `swicwe`for example: `leaf run swicwe`


## Settings

Leaf comes with some settings but packages can declare their own settings.

```sh
$ leaf config list -all mypackage
```

## Help

Leaf packages can also provide some documentation.

```sh
$ leaf help
┌───────────────────────────────────────────────────┐
│     List of help topics in installed packages     │
├──────────────────────────┬─────────┬──────────────┤
│          Topic           │ Version │   Formats    │
╞══════════════════════════╪═════════╪══════════════╡
│ swi-help/leaf            │ 2.1     │ man|html|pdf │
│ swi-help/leaf-build      │ 2.1     │ man|html|pdf │
│ swi-help/leaf-colors     │ 2.1     │ man|html|pdf │
│ swi-help/leaf-config     │ 2.1     │ man|html|pdf │
│ swi-help/leaf-env        │ 2.1     │ man|html|pdf │
│ swi-help/leaf-feature    │ 2.1     │ man|html|pdf │
│ swi-help/leaf-getsrc     │ 2.1     │ man|html|pdf │
│ swi-help/leaf-init       │ 2.1     │ man|html|pdf │
│ swi-help/leaf-manifest   │ 2.1     │ man|html|pdf │
│ swi-help/leaf-package    │ 2.1     │ man|html|pdf │
│ swi-help/leaf-profile    │ 2.1     │ man|html|pdf │
│ swi-help/leaf-remote     │ 2.1     │ man|html|pdf │
│ swi-help/leaf-run        │ 2.1     │ man|html|pdf │
│ swi-help/leaf-search     │ 2.1     │ man|html|pdf │
│ swi-help/leaf-select     │ 2.1     │ man|html|pdf │
│ swi-help/leaf-setup      │ 2.1     │ man|html|pdf │
│ swi-help/leaf-shell      │ 2.1     │ man|html|pdf │
│ swi-help/leaf-status     │ 2.1     │ man|html|pdf │
│ swi-help/leaf-update     │ 2.1     │ man|html|pdf │
│ swi-help/leaf-versioning │ 2.1     │ man|html|pdf │
│ swi-help/legato-build    │ 2.1     │ man|html|pdf │
│ swi-help/legato-dev      │ 2.1     │ man|html|pdf │
│ swi-help/legato-source   │ 2.1     │ man|html|pdf │
│ swi-help/legato-start    │ 2.1     │ man|html|pdf │
│ swi-help/legato-workflow │ 2.1     │ man|html|pdf │
└──────────────────────────┴─────────┴──────────────┘
```

Documentation can have multiple formats:
- `man` will be displayed using `/usr/bin/man`
- other formats (`html` or `pdf` in the previous snippet) will be opened using `xdg-open` (you chan change this with setting `leaf.help.default.open`)

Default format is `man` if available, you can change this with the setting `leaf.help.default.format`


## Plugins

Some leaf subcommands, like `leaf setup` are not part of leaf itself, but are *plugins*.

Note: Leaf plugins must be implemented in Python3 


# Workspace


## Structure

A *leaf workspace* is a folder that contains:
- `leaf-workspace.json`: the configuration of the workspace and its profiles
- `leaf-data/`: the folder where symbolic links are created by leaf to link the packages used by the profiles


```sh
$ leaf status   
┌───────────────────────────────────────────────────────────────────────┐
│                     Workspace: /home/seb/workspace                    │
╞═══════════════════════════════════════════════════════════════════════╡
│                  Profile: SWI-WP76 [current] (sync)                   │
├──────────┬────────────────┬───────────────────────────────────────────┤
│ Packages │   Identifier   │                Description                │
├──────────┼────────────────┼───────────────────────────────────────────┤
│ Included │ swi-wp76_1.4.2 │ SDK for WP76 (Release 9 + Legato 18.07.1) │
└──────────┴────────────────┴───────────────────────────────────────────┘
Other profile: PROFILE1

$ tree       
.
├── leaf-data
│   ├── current -> SWI-WP76
│   ├── PROFILE1
│   │   └── mypackage -> /home/seb/.leaf/mypackage_1.0
│   └── SWI-WP76
│       ├── swi-wp76 -> /home/seb/.leaf/swi-wp76_1.4.2
│       ├── wp76-image -> /home/seb/.leaf/wp76-image_1.4.2
│       ├── wp76-legato -> /home/seb/.leaf/wp76-legato_18.07.1-201809191119
│       ├── wp76-legato-image -> /home/seb/.leaf/wp76-legato-image_18.07.1-201809191119
│       ├── wp76-linux-image -> /home/seb/.leaf/wp76-linux-image_SWI9X07Y_02.16.02.00
│       ├── wp76-modem-image -> /home/seb/.leaf/wp76-modem-image_9
│       └── wp76-toolchain -> /home/seb/.leaf/wp76-toolchain_SWI9X07Y_02.16.02.00-linux64
└── leaf-workspace.json

12 directories, 1 file
4 directories, 1 file
```


## Environment

The workspace can contain environment variables that will be shared between all profiles in the workspace.

```sh
$ leaf env workspace 
# Exported by workspace
export LEAF_WORKSPACE="/home/seb/workspace";

$ leaf env workspace --set FOO=BAR
# Exported by workspace
export LEAF_WORKSPACE="/home/seb/workspace";
export FOO="BAR";

$ leaf env workspace              
# Exported by workspace
export LEAF_WORKSPACE="/home/seb/workspace";
export FOO="BAR";
```


## Sharing a workspace using Git

A leaf workspace can be shared in a git repository, to do that just:
- commit the ̀`leaf-workspace.json`: if you update a profile or create a new one, this file will have to be committed again to share the new configuration
- add `leaf-data/` to `.gitignore`: this folder is maintained by leaf itself and does not contain any important data to be shared. It will be recreated when needed.


# Profiles

A workspace can contains many profiles, each profile is identifier by its name.

> Note: Profile are usually named with CAPITAL LETTERS like `SWI-WP77`

A profile contains:
- a list pof packages
- a list of environment variables

Every profile has a folder in the `leaf-data/` folder where leaf maintains symbolic links to the packages used by the profile.

There is a spacial profile: the *current* profile. The *current* profile is just the *symbolic link* `leaf-data/current` pointing to a specific profile.
You can use `leaf profile switch` to switch from a profile to another.


## Basic operations

You can see the list of profiles with
```sh
$ leaf profile
$ leaf profile list -v
```

There is also `leaf status` which gives information about the *current profile*
```sh
$ leaf status
┌───────────────────────────────────────────────────────────────────────┐
│                     Workspace: /home/seb/workspace                    │
╞═══════════════════════════════════════════════════════════════════════╡
│                  Profile: SWI-WP76 [current] (sync)                   │
├──────────┬────────────────┬───────────────────────────────────────────┤
│ Packages │   Identifier   │                Description                │
├──────────┼────────────────┼───────────────────────────────────────────┤
│ Included │ swi-wp76_1.4.2 │ SDK for WP76 (Release 9 + Legato 18.07.1) │
└──────────┴────────────────┴───────────────────────────────────────────┘
Other profile: PROFILE1
```

Basic operation on profiles are availble with commands:
- `leaf profile create PROFILENAME` to create a new profile
- `leaf profile delete PROFILENAME` to remove a profile 
- `leaf profile rename NEWNAME` to rename the *current* profile
- `leaf profile switch PROFILENAME` to switch to a profile which means `PROFILENAME` will become the *current profile*
- `leaf profile config` to add/remove packages to a profile

Note: Adding a package to a profile does not install the package if it is missing, you need to *sync* the profile (see next chapter)
Note: Removing a profile does not uninstall packages it uses. Packages are profile/workspace independant.


## Synchronizing a profile

When you configure a profile to use a package `mypackage_1.0`, it this package is not installed when you configure the profile, it is not installed automatically.
You need to run `leaf profile sync` to synchronize the profile and install the missing packages.


## Environment

The profile can contain environment variables.

```sh
$ leaf env profile
# Exported by profile SWI-WP76
export LEAF_PROFILE="SWI-WP76";

$ leaf env profile --set FOO=BARBAR
# Exported by profile SWI-WP76
export LEAF_PROFILE="SWI-WP76";
export FOO="BARBAR";

$ leaf env profile                 
# Exported by profile SWI-WP76
export LEAF_PROFILE="SWI-WP76";
export FOO="BARBAR";
```

When you have a profile, you can use its *full environment* which is the combination of 
- the builtin environment
- the user environment (*user scope*)
- the workspace environment (*workspace scope*)
- the profile environment (*profile scope*)
- the environment of every package contained by the profile
  
```sh
$ leaf env   
# Leaf built-in variables
export LEAF_VERSION="2.1";
export LEAF_PLATFORM_SYSTEM="Linux";
export LEAF_PLATFORM_MACHINE="x86_64";
export LEAF_PLATFORM_RELEASE="4.19.0-6-amd64";
# Exported by workspace
export LEAF_WORKSPACE="/home/seb/workspace";
export FOO="BAR";
# Exported by profile SWI-WP76
export LEAF_PROFILE="SWI-WP76";
export FOO="BARBAR";
# Exported by package wp76-legato_18.07.1-201809191119
export LEGATO_ROOT="/home/seb/workspace/leaf-data/SWI-WP76/wp76-legato";
export LEGATO_TARGET="${LEGATO_TARGET:-wp76xx}";
export PATH="$LEGATO_ROOT/bin:$PATH";
export DEST_IP="${DEST_IP:-192.168.2.2}";
# Exported by package wp76-toolchain_SWI9X07Y_02.16.02.00-linux64
export WP76XX_TOOLCHAIN_DIR="/home/seb/workspace/leaf-data/SWI-WP76/wp76-toolchain/sysroots/x86_64-pokysdk-linux/usr/bin/arm-poky-linux-gnueabi";
export WP76XX_TOOLCHAIN_PREFIX="arm-poky-linux-gnueabi-";
export WP76XX_SYSROOT="/home/seb/workspace/leaf-data/SWI-WP76/wp76-toolchain/sysroots/armv7a-neon-poky-linux-gnueabi";
# Exported by package wp76-modem-image_9
export WP76XX_MODEM_IMAGES="/home/seb/workspace/leaf-data/SWI-WP76/wp76-modem-image";
# Exported by package wp76-legato-image_18.07.1-201809191119
export WP76XX_LEGATO_IMAGES="/home/seb/workspace/leaf-data/SWI-WP76/wp76-legato-image";
# Exported by package wp76-linux-image_SWI9X07Y_02.16.02.00
export WP76XX_LINUX_IMAGES="/home/seb/workspace/leaf-data/SWI-WP76/wp76-linux-image";
# Exported by package wp76-image_1.4.2
export WP76XX_DEVICE_IMAGES="/home/seb/workspace/leaf-data/SWI-WP76/wp76-image";
```

> Note: You can also set environment variables in the *user scope* with the command `leaf env user --set XXX=YYY`


## Leaf shell

To use all the environment variables of your profile you can export them to your current shell

```sh
$ echo $LEGATO_ROOT

$ eval "$(leaf env)"
$ echo $LEGATO_ROOT 
/home/seb/workspace/leaf-data/SWI-WP76/wp76-legato
```

The `eval` way of loading the environment variables is not dynamic, if you change a variable or add/remove a package to your profile, your shell environment is not updated. 

To have your shell automatically updated, use `leaf shell` instead:

```sh
$ leaf shell
Leaf Shell /usr/bin/zsh, (instead of /bin/zsh,) started.

(lsh:SWI-WP76) $ echo $LEGATO_ROOT
/home/seb/workspace/leaf-data/SWI-WP76/wp76-legato

(lsh:SWI-WP76) $ echo $MYVAR


(lsh:SWI-WP76) $ leaf env profile --set MYVAR=Hello
# Exported by profile SWI-WP76
export LEAF_PROFILE="SWI-WP76";
export FOO="BARBAR";
export MYVAR="Hello";

(lsh:SWI-WP76) $ echo $MYVAR                       
Hello
```


## Package override

In leaf you can multiple versions of a packages, but a profile can only contain one version of a package.

For example, if you create a profile to use a SDK and that SDK has a dependency on `toolchain_1.0`, you can override the used toolchain by adding `toolchain_1.1` to the profile package list.

```sh
$ leaf status -v                
┌─────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                          Workspace: /home/seb/workspace                                         │
╞═════════════════════════════════════════════════════════════════════════════════════════════════════════════════╡
│                                       Profile: SWI-WP76 [current] (sync)                                        │
├──────────────┬──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Environment  │ LEAF_WORKSPACE=/home/seb/workspace                                                               │
│              │ LEAF_PROFILE=SWI-WP76                                                                            │
├──────────────┼─────────────────────────────────────────────┬────────────────────────────────────────────────────┤
│   Packages   │                  Identifier                 │                    Description                     │
├──────────────┼─────────────────────────────────────────────┼────────────────────────────────────────────────────┤
│   Included   │ swi-wp76_1.4.2                              │ SDK for WP76 (Release 9 + Legato 18.07.1)          │
├──────────────┼─────────────────────────────────────────────┼────────────────────────────────────────────────────┤
│ Dependencies │ wp76-legato_18.07.1-201809191119            │ Legato Framework built for WP76                    │
│              │ wp76-toolchain_SWI9X07Y_02.16.02.00-linux64 │ GCC cross compiler Toolchain for WP76              │
│              │ wp76-modem-image_9                          │ Modem Firmware Image for WP76                      │
│              │ wp76-legato-image_18.07.1-201809191119      │ Legato Image for WP76                              │
│              │ wp76-linux-image_SWI9X07Y_02.16.02.00       │ Linux Image for WP76                               │
│              │ wp76-image_1.4.2                            │ Device Image for WP76 (Release 9 + Legato 18.07.1) │
└──────────────┴─────────────────────────────────────────────┴────────────────────────────────────────────────────┘
Other profile: PROFILE1

$ leaf package list -a toolchain                                              
┌──────────────────────────────────────────────────────────────────────────────────────────────┐
│                               2 packages - Filter: "toolchain"                               │
├─────────────────────────────────────────────┬───────────────────────────────────────┬────────┤
│                  Identifier                 │              Description              │  Tags  │
╞═════════════════════════════════════════════╪═══════════════════════════════════════╪════════╡
│ wp76-toolchain_SWI9X07Y_02.16.02.00-linux64 │ GCC cross compiler Toolchain for WP76 │ wp76xx │
│ wp76-toolchain_SWI9X07Y_02.28.03.05-linux64 │ GCC cross compiler Toolchain for WP76 │ wp76xx │
└─────────────────────────────────────────────┴───────────────────────────────────────┴────────┘

$ leaf profile config -p wp76-toolchain_SWI9X07Y_02.28.03.05-linux64

$ leaf profile sync                                                           

$ leaf status -v                                                    
┌─────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                          Workspace: /home/seb/workspace                                         │
╞═════════════════════════════════════════════════════════════════════════════════════════════════════════════════╡
│                                       Profile: SWI-WP76 [current] (sync)                                        │
├──────────────┬──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Environment  │ LEAF_WORKSPACE=/home/seb/workspace                                                               │
│              │ LEAF_PROFILE=SWI-WP76                                                                            │
├──────────────┼─────────────────────────────────────────────┬────────────────────────────────────────────────────┤
│   Packages   │                  Identifier                 │                    Description                     │
├──────────────┼─────────────────────────────────────────────┼────────────────────────────────────────────────────┤
│   Included   │ swi-wp76_1.4.2                              │ SDK for WP76 (Release 9 + Legato 18.07.1)          │
│              │ wp76-toolchain_SWI9X07Y_02.28.03.05-linux64 │ GCC cross compiler Toolchain for WP76              │
├──────────────┼─────────────────────────────────────────────┼────────────────────────────────────────────────────┤
│ Dependencies │ wp76-legato_18.07.1-201809191119            │ Legato Framework built for WP76                    │
│              │ wp76-modem-image_9                          │ Modem Firmware Image for WP76                      │
│              │ wp76-legato-image_18.07.1-201809191119      │ Legato Image for WP76                              │
│              │ wp76-linux-image_SWI9X07Y_02.16.02.00       │ Linux Image for WP76                               │
│              │ wp76-image_1.4.2                            │ Device Image for WP76 (Release 9 + Legato 18.07.1) │
└──────────────┴─────────────────────────────────────────────┴────────────────────────────────────────────────────┘
Other profile: PROFILE1
```

Note: The strategy leaf uses to resolve package dependencies allow package overrides only with higher versions.


# Extra

## Autocompletion

If installed using the `leaf.deb` package, autocompletion should be automatically enabled for `bash`and `zsh`.

To enable autocompletion
- for *bash*, use `eval "$(leaf completion -s bash)"`
- for *zsh*, use `eval "$(leaf completion -s zsh)"`
- for *tcsh*, use `eval "$(leaf completion -s tcsh)"`

Example using bash:
```sh
$ leaf p<TAB><TAB> # No completion available
$ eval "$(leaf completion)"
$ leaf p<TAB><TAB>
package  profile  
```


## Settings

Settings allow to customize some features of *Leaf* itself but can also be used to tweak package behavior (package can declare their own settings)

```sh
$ leaf config list -a 
┌──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                                 Configuration folder: /home/seb/.config/leaf                                                 │
├───────────────────────────────┬──────────────────────────────────────────────────────────────────────────┬───────────────────────────────────┤
│           Identifier          │                               Description                                │               Value               │
╞═══════════════════════════════╪══════════════════════════════════════════════════════════════════════════╪═══════════════════════════════════╡
│ leaf.build.tar                │ Use custom tar binary instead of *tar* command when generating artifacts │                                   │
│ leaf.cache                    │ Leaf cache                                                               │ "~/.cache/leaf"                   │
│ leaf.debug                    │ Enable traces                                                            │                                   │
│ leaf.download.resume.disable  │ Disable resume when a download fails                                     │                                   │
│ leaf.download.retry           │ Retry count for download operations                                      │ "5"                               │
│ leaf.download.timeout         │ Timeout (in sec) for download operations                                 │ "20"                              │
│ leaf.gpg.server               │ Server where GPG keys will be fetched                                    │ "subset.pool.sks-keyservers.net"  │
│ leaf.help.default.format      │ Default format for help topics                                           │ "man"                             │
│ leaf.help.default.open        │ Default command to open help topics if not 'manpage' format              │ "xdg-open"                        │
│ leaf.locks.disable            │ Disable lock files for install operations                                │                                   │
│ leaf.noninteractive           │ Do not ask for confirmations, assume yes                                 │                                   │
│ leaf.pager                    │ Force a pager when a pager is needed                                     │                                   │
│ leaf.plugins.disable          │ Disable plugins                                                          │                                   │
│ leaf.profile.relative.disable │ Disable relative path for installed package of a profile                 │                                   │
│ leaf.remote.smartrefresh      │ Delta time (in days) before remotes are automatically fetched            │ "1"                               │
│ leaf.shell.default            │ Shell used when leaf needs an internal shell to run commands             │ "bash"                            │
│ leaf.system.roots             │ Folders where system leaf packages are installed                         │ "/usr/share/leaf/packages:\       │
│                               │                                                                          │  /usr/local/share/leaf/packages:\ │
│                               │                                                                          │  ~/.local/share/leaf/packages"    │
│ leaf.theme                    │ Custom color theme                                                       │                                   │
│ leaf.user.root                │ Folder where leaf packages are installed                                 │ "~/.leaf"                         │
└───────────────────────────────┴──────────────────────────────────────────────────────────────────────────┴───────────────────────────────────┘
```

> Note: Some settings may have a value validator, use `--verbose` to display it


## Configuration override

There is no settings to set the *configuration folder* of leaf (because this folder contains the settings ...)

To override the configuration folder, use
```sh
$ export LEAF_CONFIG=~/cutom_leaf_config_folder/
$ leaf_config/ leaf config
┌──────────────────────────────────────────────────────────┐
│ Configuration folder: /home/seb/cutom_leaf_config_folder │
└──────────────────────────────────────────────────────────┘
```


## All-In-One commands

### *leaf setup*

The command `leaf setup` is a all-in-one command which chains some basic operations:
- if you are not in a workspace, create one with `leaf init`
- create a new profile with `leaf profile create XXX`, the name of the new profile is computed using the name of the package(s) you want to use in your profile
- add the packages to the new profile with `leaf profile config -p XXX`
- optionally add the env variables with `leaf env profile --set XXX=YYY`
- sync the newly create profile with `leaf profile sync`

One feature of `leaf setup` if to resolve the version automatically, you can simply use `leaf setup -p swi-wp76` instead of `leaf setup -p swi-wp76_4.3.1` (`4.3.1̀` was the highest version available at the time I ran the example)


### *leaf update*

This is an interactive command which propose to update the current profile to use the latest version available of every packages included in the profile.


## Keyword *latest*

The *latest* version is a reserved keyword, it is used to reference the latest version of a package.

> Note: to compare versions, leaf uses its own algo, see `leaf help leaf-versioning` for more details

There is a binary to use leaf versioning in *shell* scripts for example:

```sh
$ leaf-version-compare -h         
usage: leaf-version-compare [-h] (-gt | -ge | -lt | -le | -eq | -ne) A B

Leaf version comparator

positional arguments:
  A           version A
  B           version B

optional arguments:
  -h, --help  show this help message and exit
  -gt         Compare A > B
  -ge         Compare A >= B
  -lt         Compare A < B
  -le         Compare A <= B
  -eq         Compare A == B
  -ne         Compare A != B

$ leaf-version-compare 1.0 -le 2.0
True: 1.0 <= 2.0
```
