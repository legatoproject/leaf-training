- [Introduction](#introduction)
- [Setup a dev env](#setup-a-dev-env)
  - [Clone the repository](#clone-the-repository)
  - [Virtualenv](#virtualenv)
  - [Multiple python](#multiple-python)
  - [Code formatting](#code-formatting)
  - [Flake8](#flake8)
- [Passing the tests](#passing-the-tests)
  - [Pytest](#pytest)
  - [Tox](#tox)
  - [Docker image](#docker-image)
- [Developing](#developing)
  - [API levels](#api-levels)
  - [Core](#core)
  - [Model](#model)
  - [API](#api)
  - [Rendering](#rendering)
  - [CLI](#cli)
  - [Tests](#tests)
- [Design and Architecture](#design-and-architecture)
  - [System packages vs user packages](#system-packages-vs-user-packages)
  - [leaf vs. leaf-legato](#leaf-vs-leaf-legato)
  - [Develop plugins](#develop-plugins)
  - [Keep leaf core as small as possible](#keep-leaf-core-as-small-as-possible)
  - [Main sequence](#main-sequence)
  - [Manifest validation](#manifest-validation)
  - [Handle model auto update](#handle-model-auto-update)
  - [Expected output](#expected-output)

# Introduction

Leaf is a command line tool and a library developed in python3.

> Note: You need python 3.7+ because on some dev dependencies like black.

# Setup a dev env


To install python3 and some dependencies:
```sh
$ sudo apt install ×\
    python3 python3-pip python3-virtualenv \
    git make gnupg
```

## Clone the repository


```sh
$ git clone https://github.com/legatoproject/leaf
# Or use internal mirror
$ git clone ssh://$USER@gerrit.legato:29418/leaf
$ cd leaf
```

## Virtualenv

To create the *virtualenv* with all needed libraries and tools use:
```sh
$ make venv
$ source venv/bin/activate
(venv) $ pip install -e .
```

> Note: Installing leaf using `pip install -e` installs in development mode, code modifications are automatically taken into account

## Multiple python

Leaf has to be compatible with multiple versions of python currently
- Python 3.4: may/should disappear in a near future, only used by Debian oldstable
- Python 3.5/3.6/3.7/3.8

> Note: take care when using `pathlib` API which has some changes between 3.4 and 3.5+


## Code formatting

To avoid code formatting issue, we use `black` as code formatter. 
Black needs python 3.7+, we use the default options but we use 160 line length with arguments `-l 160`

## Flake8

For static code analysis we use `Flake8` with a set of plugins. 
The plugin list is in `requirements-dev.txt`
```
## Flake8 & plugins
flake8
flake8-html
pep8-naming
flake8-builtins
flake8-blind-except
flake8-comprehensions
flake8-string-format
flake8-pep3101
flake8-bugbear
```

The flake8 configuration is in the `tox.ini` configuration file
```
[flake8]
application-import-names = leaf
max-line-length = 160
enable-extensions = G
# Best way to select all installed plugins so far ...
select = E,F,W,A,B,C,D,G,H,I,J,K,L,M,N,O,P,Q,R,S,T,U,V,X,Y,Z
ignore = E501,E203,W503
format = html
htmldir = flake-report
show-source = true
jobs = 4
exclude =
    __pycache__
```

> Hint: As jenkins job ensure there is no Flake8 warning, before submitting a change request, use `tox -e flake8 || firefox flake-report/index.html`

# Passing the tests

## Pytest

To pass the tests use:
```sh
(venv) $ pytest src/tests/
# or to select a test scope
(venv) $ pytest src/tests/api
# or to select a particular method
(venv) $ pytest -k test_my_method src/tests/
```

## Tox

We use *Tox* to pass the tests on every supported python version.
You can use tox to pass the tests, but it requires that you install python 3.4, 3.5, 3.6, 3.7,3.8 on your system:

```sh
(venv) $ tox
# You can also select a particular environment
(venv) $ tox -e py37
```

## Docker image

It is not necessary to install all supported python versions directly on your system, you can use a Docker image to test that:
```sh
# build the docker image
$ make docker-image
# run tests
$ make docker-tests
```

An alternative based on [Multipy](https://github.com/essembeh/multipy):
```sh
# Build the image
$ docker build \
    --build-arg APT_EXTRA_PACKAGES="gnupg" \
    -t multipy:leaf \
    https://github.com/essembeh/multipy.git 
# Run the tests
$ docker run \
    --rm \
    --volume $PWD:/src:ro \
    -e GIT_CLEAN=1 \
    -e TOXENV -e TOX_SKIP_ENV \
    multipy:leaf -- -n 4
```

> Note: PR to be accepted to move the Docker image from leaf repo to farm repository


# Developing

## API levels

The source code is separated in different folders:

```sh
$ ll src/leaf
total 36K
drwxr-xr-x 3 seb seb  4,0K nov.  18 15:34 api
drwxr-xr-x 4 seb seb  4,0K oct.  23 11:00 cli
drwxr-xr-x 3 seb seb  4,0K nov.  18 15:34 core
-rw-r--r-- 1 seb seb   868 avril 12  2019 __init__.py
-rw-r--r-- 1 seb seb  1,9K oct.  28 17:47 __main__.py
drwxr-xr-x 3 seb seb  4,0K nov.  18 15:34 model
drwxr-xr-x 4 seb seb  4,0K oct.  23 11:00 rendering
-rw-r--r-- 1 seb seb  1,1K juil.  3 13:49 tools.py
```

- `core/`: conains the *leaf core* classes, like exceptions, utils, constants and settings
- `model/`: contains all the model files, like `PackageIdentifier`, `AvailablePackage` ... but also the model of the configuration files
- `api/`: contains *public* APIs we need to maintain because they are used by external applications like plugins and scripts
- `rendering/`: contains all the class dedicated to the table rendering like `leaf search` 
- `cli/`: contains all the code related to the leaf CLI, all commands and utils the the commands
- `__main__.py`: the entry point of `leaf`
- `tools.py`: the entry point of `leaf-version-compare`

> Note: avoid circular dependencies

## Core

- `constants.py` contains all the constants but also all the keys of Json properties used by the model
- `download.py` functions to download files with progress/retry/timeout ...
- `error.py` all exceptions raised by leaf
- `jsonutils.py` some functions to use Json model
- `lock.py` leaf have *lock* files when installing packages for example
- `logger.py` 
- `settings.py` all the internal leaf settings (modifiable with `export XXX=YYY` or with the CLI `leaf config set XXX YYY`)
- `utils.py` 

## Model

Important classes reprensenting packages:
- `PackageIdentifier`: to manipulate NAME_VERSION identifiers
  - can be sorted
  - has regular expressions to validate fields
- `ConditionalPackageIdentifier`: same but with optional conditions
- `Manifest`:  represents a manifest.json and contains methods to access its fields
  - `Manifest.parse` static method to parse a file
- `LeafArtifact`: represents a *.leaf* file
  - the *manifest.json* is parsed from the *TAR* archive
- `AvailablePackage`: represents a package which is available in a *remote*
  - uses the *URL* api to resolve the artifact *URL* from a remote
  - dupplicate mecanism to detects same artifacts with different sha384
  - candidate mecanism to allow remote priorities
- `InstalledPackage`: package wich is extracted in a *folder*
  - can be *readonly* for *system packages*

Other important model classes:
- `Remote` to reprensent a remote repository
- `Profile` taht contains and manipulate package inside a workspace

The `DependencyUtils` class contains all the dependency logic
- `installed(pilist: list, ipmap: dict, env: Environment = None, only_keep_latest: bool = False, ignore_unknown: bool = False):`
- `install(pilist: list, apmap: dict, ipmap: dict, env: Environment = None):`
- `uninstall(pilist: list, ipmap: dict, env: Environment = None, logger: TextLogger = None):`
- `prereq(pilist: list, apmap: dict, ipmap: dict, env: Environment = None):`
- `upgrade(namelist: list, apmap: dict, ipmap: dict, env: Environment = None):`
- `rdepends(pilist: list, mfmap: dict, env: Environment = None):`


The class `Environment` is a *KEY/VALUE* container with represent an a variable environment
- can contain sub environment
- can override variables
- handle in/out scripts
- have `activate` and `deactivate` visitors
- can generate in/out scripts with `generate_scripts`
- is meant to be used with interface `IEnvProvider`

Steps execution is managed with `StepExecutor`
- uses a ̀`VariableResolver` to resolve `@{DIR:foo_1.0}`
- relies on `execute_command(*args, cwd=None, env=None, print_stdout=False):`

## API

The leaf API has some layers:
- `ConfigurationManager` contains all methods to work with the leaf configuration and folders. It also expose the API to list installed packages
- `LoggerManager` contains methods to print messages and renderers
- `GPGManager` contains all methods to fetch and verify signatures
- `RemoteManager` contains methods to add/remove/list/fetch remotes and list available packages
- `PackageManager` contains API to install/uninstall/sync packages and get packages environments
- `WorkspaceManager` is the API to work with a workspace and profiles
- `RelengManager` is the API to build manifest/packages/index.json 

## Rendering

Automatically handle formatted tables with UTF8 characters when possible.

Can be themed using the `themes.ini`

## CLI

Important classes are
- `LeafCommand` which defines a CLI command
- `LeafMetaCommand` to develop command container (subcommands)

To develop a new verb, for example `leaf package foo`
- In `src/leaf/cli/commands/package.py` add a new class `PackageFooCommand` which extends `LeafCommand`
- Add a name/description in constructor. Your command can also be configured to accept unknown arguments
- Setup the `ArgumentParser` in `def _configure_parser(self, parser)`
- Do not forget to add completions when possible (see `completion.py`)

```python
class MyCommand(LeafCommand):
    def __init__(self):
        LeafCommand.__init__(self, "sample", "do something special", allow_uargs=True)

    def _configure_parser(self, parser):
        super()._configure_parser(parser)
        parser.add_argument(
            "-p", "--package", 
            dest="package", 
            metavar="PKG_IDENTIFIER", 
            type=PackageIdentifier.parse, 
            help="packages"
        ).completer = complete_installed_packages

    def execute(self, args, uargs):
        return 1
```

## Tests

Tests are also sorted into 4 folders:
- `internal`: to tests internal utils methods and private APIs
- `api`: to test leaf *public* APIs
- `cli`: to test commands and arguments
- `plugins`: to test core plugins

> Every test extending `LeafTestCase` has its own leaf configuration and a dedicated volatile working folder


Some packages are available:

```sh
$ ls src/tests/resources/packages
compress-bz2_1.0  container-A_1.1              failure-postinstall-exec_1.0         scripts_1.0
compress-gz_1.0   container-A_2.0              failure-postinstall-exec-silent_1.0  settings_1.0
compress-tar_1.0  container-A_2.1              install_1.0                          subscripts_1.0
compress-xz_1.0   container-B_1.0              multitags_1.0                        subscripts_1.1
condition_1.0     container-C_1.0              pkg-with-deps-with-prereq_1.0        sync_1.0
condition-A_1.0   container-D_1.0              pkg-with-prereq_0.1                  testlatest_1.0
condition-A_2.0   container-E_1.0              pkg-with-prereq_1.0                  testlatest_2.0
condition-B_1.0   container-E_1.1              pkg-with-prereq_2.0                  testlatest_2.1
condition-C_1.0   env-A_1.0                    pluginA_1.0                          upgrade_1.0
condition-D_1.0   env-B_1.0                    pluginA_1.1                          upgrade_1.1
condition-E_1.0   failure-badhash_1.0          pluginB_1.0                          upgrade_1.2
condition-F_1.0   failure-depends-leaf_1.0     prereq-A_0.1-fail                    upgrade_2.0
condition-G_1.0   failure-large-ap_1.0         prereq-A_1.0                         version_1.0
condition-H_1.0   failure-large-extracted_1.0  prereq-B_1.0                         version_1.1
container-A_1.0   failure-minver_1.0           prereq-B_2.0                         version_2.0
```
> All these packages are built for every TestCase extending the class `LeafTestCaseWithRepo`

You can generate the unit test repository using:

```sh
$ python3 src/tests/testutils.py
$ leaf remote add test1 --insecure /tmp/leaf/repository/index1.json
$ leaf remote add test2 --insecure /tmp/leaf/repository/index2.json
```

# Design and Architecture

## System packages vs user packages

Leaf finds installed packages in:
- each folder declared in setting `leaf.system.roots` (env var `LEAF_SYSTEM_ROOTS`) which is a `:` separated list, like `$PATH`
- in folder declared in setting `leaf.user.root` (env var `LEAF_USER_ROOTS`)

So, there are two kinds of `InstalledPackage`, *system* packages which cannot be uninstalled nor installed and *user* packages.

Default value for `leaf.system.roots` is `/usr/share/leaf/packages:/usr/share/leaf/packages:~/.local/share/leaf/packages`

> Note: *system* packages can be shared by multiple users on the same machine like in folder: `/usr/share/leaf/packages`

## leaf vs. leaf-legato

Leaf core is Legato agnostic, there is no direct legato workflow related command nor configration file in leaf git repository.
The Legato leaf customization is done in the *leaf-legato* git repository.

The *leaf-legato* contains scripts to build de deb package for leaf.
To build the deb package, run `make` in *leaf-legato* repository (you may need to init a fake GPG key with `make gpg` first)

```sh
$ tree
.
├── Makefile
├── packaging
│   ├── gpg-script
│   ├── input
│   │   └── config
│   │       ├── config.json
│   │       └── themes.ini
│   ├── leaf
│   │   ├── inputs
│   │   │   └── swi-aptdeps-leaf-src
│   │   │       ├── copy.sh
│   │   │       └── packages.list
│   │   ├── Makefile
│   │   └── templates
│   │       ├── swi-aptdeps-leaf-src.json
│   │       ├── swi-leaf.json
│   │       └── swi-leaf-src.json
│   ├── leafsyspackages.txt
│   ├── mkdeb.sh
│   └── skel
│       └── debian
│           ├── compat
│           ├── control
│           ├── copyright
│           ├── docs
│           ├── install
│           ├── postinst
│           ├── postrm
│           ├── rules
│           └── source
│               └── format
└── README.md
```

This repository contains:
- the list of *system* packages to bundle in the deb package in the file `packaging/leafsyspackages.txt`
- the default configuration file with default repositories: `packaging/input/config/config.json`
- the default theme file: `packaging/input/config/themes.ini`
- the deb package configration files in `packaging/skel/debian`


## Develop plugins

Default system plugins:
- `shell`
- `completion`
- `update`
- `version`

Legato specific plugins are not part of leaf core, but added into the generated debian package:

```sh
syspacks:
	$(LEAF_CFG) && \
	$(LEAF) config --root $(LEAF_LEGATO_ROOT)/packaging/input/packages && \
	$(LEAF) remote add leaf-packages http://get.prod.legato/tools/leaf-packages/releases/latest/index.json --insecure && \
	yes | ($(LEAF) package install `cat $(LEAF_LEGATO_ROOT)/packaging/leafsyspackages.txt`)
```

```txt
swi-legato_1.3.190605
swi-linux_1.0.190419
swi-sdk-builder_4.0.190507
swi-getsrc_1.1.190524
swi-help_2.1-2
swi-leaf-legacy_1.0.190429
```

## Keep leaf core as small as possible

```sh
$ leaf setup -p sdk_1.0

$ leaf init 
$ leaf profile create SDK
$ leaf profile config -p sdk_1.0
$ leaf profile sync`
```

```sh
$ leaf version

$ leaf --version
```

```sh
$ leaf shell

$ eval "$(leaf env)"
```

## Main sequence

Entrypoints:
- file `__main__.py`
  - `setup.py` declares entrypoint to `def main():`
  - `if __name__ == "__main__":`
  - `def run_leaf(argv, catch_int_sig=True):`

Sequence:
- Check supported python version
- Replace CTRL+C exception
- Load the user configuration
  - initialize settings
  - load plugins from `leaf.user.root` and `leaf.system.roots`
- build the ArgumentParser
- parse command line
- execute the command handler
  - returns the handler returned value
  - in case of exception, the exception can define an error code


## Manifest validation

Json schema in file `src/leaf/model/manifest.schema.json`

Do not forget to update this file to document new fields.

```python
from jsonschema import validate
class Manifest(JsonObject):
    def validate_model(self):
        validate(self.json, jloads(resource_string(__name__, LeafFiles.SCHEMA).decode()))
```

> Note: only used in `leaf build pack`


## Handle model auto update

The configuration files:
- `ConfigFileWithLayer`
  - `_get_updaters` to manage auto mode upgrade
  - `_check_model` to ensure model consistency and upgrade
- `UserConfiguration` & `WorkspaceConfiguration`

```python
class UserConfiguration(ConfigFileWithLayer, IEnvProvider):

    def _get_updaters(self) -> dict:
        return super()._get_updaters() + ((Version("2.0"), update_root_folder),)

[...]

def update_root_folder(user_config):
    if LegacyJsonConstants.CONFIG_ROOTFOLDER in user_config.json:
        user_config._getenvmap()[LeafSettings.USER_PKG_FOLDER.key] = user_config.json[LegacyJsonConstants.CONFIG_ROOTFOLDER]
        del user_config.json[LegacyJsonConstants.CONFIG_ROOTFOLDER]
```

## Expected output

For tests, the *stdout* and/or the *stderr* can be checked

```python
class TestCliPackageManager(LeafTestCaseWithCli):
    def test_depends_available(self):
        with self.assertStdout("a.out"):
            self.leaf_exec(["package", "deps"], "--available", "condition_1.0")
```

```sh
$ cat src/tests/resources/expected_ouput/TestCliPackageManager/test_depends_available_a.out 
┌──────────────────────────────────────┐
│      9 packages - Filter: None       │
├─────────────────┬─────────────┬──────┤
│    Identifier   │ Description │ Tags │
╞═════════════════╪═════════════╪══════╡
│ condition-A_1.0 │             │      │
│ condition-B_1.0 │             │      │
│ condition-C_1.0 │             │      │
│ condition-D_1.0 │             │      │
│ condition-E_1.0 │             │      │
│ condition-F_1.0 │             │      │
│ condition-G_1.0 │             │      │
│ condition-H_1.0 │             │      │
│ condition_1.0   │             │      │
└─────────────────┴─────────────┴──────┘
```

Templates have builtin variables resolution:
```python
    def _get_default_variables(self):
        out = OrderedDict()
        out["{TESTS_LEAF_VERSION}"] = __version__
        out["{TESTS_PLATFORM_SYSTEM}"] = platform.system()
        out["{TESTS_PLATFORM_MACHINE}"] = platform.machine()
        out["{TESTS_PLATFORM_RELEASE}"] = platform.release()
        out["{LEAF_PROJECT_ROOT_FOLDER}"] = LEAF_PROJECT_ROOT_FOLDER
        out["{TESTS_FOLDER}"] = self.test_folder
        return out
```

You can also initialize the templates with:
```sh
$ LEAF_UT_CREATE_TEMPLATE=1 pytest src/tests/cli/test_cli_packagemanager.py
```
